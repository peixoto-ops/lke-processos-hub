# Modelo Relacional - Secretario Agente LKE

**Data:** 2026-04-18
**Status:** Análise do schema existente + propostas

---

## 1. SCHEMA EXISTENTE

### 1.1 Tabelas Principais

```
┌─────────────────┐     ┌─────────────────────┐     ┌─────────────────┐
│    clientes     │     │ vinculos_cliente_   │     │    processos    │
│                 │─────│ processo            │─────│                 │
│ id, nome, cpf   │     │                     │     │ id, numero,     │
│ rg, email,      │     │ cliente_id (FK)     │     │ tribunal, classe│
│ telefone, ...   │     │ processo_id (FK)    │     │ fase, situacao  │
└─────────────────┘     │ qualidade, posicao  │     └─────────────────┘
                        └─────────────────────┘              │
                                                             │
                        ┌─────────────────────┐              │
                        │      prazos         │◄─────────────┘
                        │                     │
                        │ processo_id (FK)    │
                        │ data_limite, tipo   │
                        │ status, prioridade  │
                        └─────────────────────┘
```

### 1.2 Tabela de Comunicações

```
┌─────────────────────┐
│   comunicacoes      │
│                     │
│ cliente_id (FK) ────┼──► clientes
│ processo_id (FK) ───┼──► processos
│ tipo (telefone/     │
│      email/whatsapp/│
│      reuniao/outro) │
│ data, resumo        │
│ responsavel         │
│ seguimento          │
└─────────────────────┘
```

### 1.3 Tabelas de Sistema

| Tabela | Função |
|--------|--------|
| `repositorios` | Git repos monitorados |
| `sessoes` | Sessões de trabalho |
| `artefatos` | Arquivos gerados |
| `atividade_logs` | Histórico de commits |
| `credenciais` | Referências a secrets |
| `skills` | Skills registradas |
| `tags` / `sessao_tags` | Tagging de sessões |

---

## 2. ANÁLISE DO RELACIONAMENTO CLIENTES ↔ PROCESSOS

### 2.1 Tabela de Vínculos Existente

```sql
CREATE TABLE vinculos_cliente_processo (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  cliente_id INTEGER NOT NULL,
  processo_id INTEGER NOT NULL,
  qualidade TEXT NOT NULL,   -- autor, reu, interessado, assistente
  posicao TEXT,              -- qualificacao processual
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  FOREIGN KEY (processo_id) REFERENCES processos(id),
  UNIQUE(cliente_id, processo_id)
);
```

**Isso já resolve:**
- ✅ Cliente sem processo (não há registro na tabela)
- ✅ Processo com vários clientes (múltiplos vínculos)
- ✅ Cliente com vários processos (múltiplos vínculos)
- ✅ Qualidade processual (autor, réu, assistente)

### 2.2 Qualidades Processuais Comuns

| Qualidade | Descrição |
|-----------|-----------|
| `autor` | Parte que propõe a ação |
| `reu` | Parte que responde à ação |
| `interessado` | Terceiro interessado |
| `assistente` | Assistente litisconsorcial |
| `litisconsorte` | Litisconsorte passivo/ativo |
| `herdeiro` | Herdeiro em inventário |
| `credor` | Credor em execução |
| `devedor` | Devedor em execução |

---

## 3. TABELAS FALTANTES (PROPOSTAS)

### 3.1 Tabela: atendimentos

**Justificativa:** A tabela `comunicacoes` registra contatos pontuais. `atendimentos` seria para reuniões formais com pauta, duração, encaminhamentos.

```sql
CREATE TABLE atendimentos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  
  -- Relacionamentos
  cliente_id INTEGER,           -- pode ser NULL se atendimento interno
  processo_id INTEGER,          -- pode ser NULL se consulta inicial
  
  -- Dados do atendimento
  data_hora_inicio TIMESTAMP NOT NULL,
  data_hora_fim TIMESTAMP,
  duracao_minutos INTEGER,
  
  -- Classificação
  tipo TEXT CHECK(tipo IN (
    'consulta_inicial',
    'reuniao_processual',
    'reuniao_status',
    'atendimento_urgente',
    'videochamada',
    'presencial',
    'telefone'
  )),
  
  -- Conteúdo
  pauta TEXT,                   -- itens discutidos
  resumo TEXT,                  -- ata resumida
  encaminhamentos TEXT,         -- JSON ou texto estruturado
  proximos_passos TEXT,
  
  -- Metadados
  local TEXT,                   -- endereco ou "online"
  participantes TEXT,           -- JSON com lista de presentes
  gravado BOOLEAN DEFAULT 0,
  transcricao TEXT,             -- link ou texto
  
  -- Controle
  status TEXT CHECK(status IN (
    'agendado',
    'confirmado',
    'realizado',
    'cancelado',
    'remarcado'
  )) DEFAULT 'agendado',
  
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  atualizado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  FOREIGN KEY (processo_id) REFERENCES processos(id)
);

CREATE INDEX idx_atendimentos_cliente ON atendimentos(cliente_id);
CREATE INDEX idx_atendimentos_processo ON atendimentos(processo_id);
CREATE INDEX idx_atendimentos_data ON atendimentos(data_hora_inicio);
```

### 3.2 Tabela: andamentos_processuais

**Justificativa:** Registro cronológico de movimentações processuais (publicações, decisões, despachos).

```sql
CREATE TABLE andamentos_processuais (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  
  -- Relacionamento
  processo_id INTEGER NOT NULL,
  
  -- Dados do andamento
  data_publicacao DATE NOT NULL,
  data_integracao TIMESTAMP,    -- quando chegou ao sistema
  numero_seq INTEGER,           -- sequencial no processo
  
  -- Classificação
  tipo TEXT CHECK(tipo IN (
    'despacho',
    'decisao',
    'sentenca',
    'acordao',
    'certidao',
    'publicacao',
    'intimacao',
    'citação',
    'juntada',
    'movimentacao'
  )),
  
  -- Conteúdo
  conteudo TEXT,                -- texto do andamento
  resumo TEXT,                  -- resumo gerado por IA
  link_fonte TEXT,              -- URL do tribunal
  
  -- Metadados
  orgao_origem TEXT,            -- vara, turma, camara
  responsavel TEXT,             -- juiz, relator
  
  -- Status
  lido BOOLEAN DEFAULT 0,
  processado_ia BOOLEAN DEFAULT 0,  -- analisado pelo secretario-agente
  
  -- Prioridade (preenchido por IA)
  urgente BOOLEAN DEFAULT 0,
  requer_prazo BOOLEAN DEFAULT 0,
  prazo_dias INTEGER,
  
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (processo_id) REFERENCES processos(id)
);

CREATE INDEX idx_andamentos_processo ON andamentos_processuais(processo_id);
CREATE INDEX idx_andamentos_data ON andamentos_processuais(data_publicacao);
CREATE INDEX idx_andamentos_tipo ON andamentos_processuais(tipo);
CREATE INDEX idx_andamentos_nao_lidos ON andamentos_processuais(lido) WHERE lido = 0;
```

---

## 4. MODELO RELACIONAL COMPLETO

```
                            ┌─────────────────────┐
                            │     clientes        │
                            │                     │
                            │ id, nome, cpf, ...  │
                            └─────────────────────┘
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         │                           │                           │
         ▼                           ▼                           ▼
┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐
│   atendimentos  │        │  comunicacoes   │        │ vinculos_cliente│
│                 │        │                 │        │    _processo    │
│ cliente_id (FK) │        │ cliente_id (FK) │        │                 │
│ processo_id(FK) │        │ processo_id(FK) │        │ cliente_id (FK) │──┐
│ data_hora, tipo │        │ tipo, data,     │        │ processo_id(FK) │  │
│ resumo, pauta   │        │ resumo          │        │ qualidade       │  │
└─────────────────┘        └─────────────────┘        └─────────────────┘  │
         │                                                   │           │
         │                                                   ▼           │
         │                                           ┌─────────────────┐  │
         │                                           │    processos    │  │
         │                                           │                 │  │
         │                                           │ id, numero,     │  │
         │                                           │ tribunal, fase  │  │
         │                                           └─────────────────┘  │
         │                                                   │           │
         │                           ┌───────────────────────┼───────────┘
         │                           │                       │
         ▼                           ▼                       ▼
┌──────────────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│andamentos_processuais│    │     prazos      │    │   vinculados_       │
│                      │    │                 │    │   repositorios      │
│ processo_id (FK)     │    │ processo_id(FK) │    │                     │
│ data_publicacao      │    │ data_limite     │    │ (ver schema)        │
│ tipo, conteudo       │    │ status          │    └─────────────────────┘
│ resumo (IA)          │    └─────────────────┘
└──────────────────────┘
```

---

## 5. IMPACTO NAS LABELS GMAIL

### 5.1 Relação Atendimento → Label

Com a tabela `atendimentos`, o fluxo de labels muda:

```
cliente-novo ──► cliente-validado ──► cliente-agenda
                                            │
                                            ▼
                                    [FORM: Agendamento]
                                            │
                                            ▼
                                     cliente-reuniao
                                            │
                                            ▼
                                      ATENDIMENTO
                                    (tabela SQLite)
                                            │
                                            ▼
                                     cliente-ativo
```

### 5.2 Labels Propostas (Revisão)

| Label | Quando Usar | Ação Automática |
|-------|-------------|-----------------|
| `cliente-novo` | Form cadastro | Extrair dados, criar cliente |
| `cliente-validado` | Dados confirmados | Criar contato Google |
| `cliente-agenda` | Solicitar agendamento | Criar registro atendimento (status=agendado) |
| `cliente-reuniao` | Reunião confirmada | Atualizar atendimento (status=confirmado) |
| `cliente-ativo` | Reunião realizada | Atualizar atendimento (status=realizado) |
| `cliente-atualizado` | Form atualização | Diff campos, atualizar SQLite |
| `cliente-inativo` | 180d sem atendimento | Arquivar, notificar |
| `cliente-arquivado` | Caso encerrado | Manter histórico |

### 5.3 Automatismos por Label

```python
LABEL_ACTIONS = {
    'cliente-novo': [
        'parse_email_to_yaml',
        'insert_cliente_sqlite',
        'create_obsidian_ficha',
        'move_label: cliente-validado'
    ],
    'cliente-validado': [
        'sync_google_contacts',
        'await_manual_action'  # aguardar agendamento
    ],
    'cliente-agenda': [
        'parse_agendamento_email',
        'create_atendimento_record',
        'check_calendar_availability',
        'move_label: cliente-reuniao'
    ],
    'cliente-reuniao': [
        'update_atendimento_status(confirmed)',
        'generate_audio_confirmation',
        'await_realizacao'
    ],
    'cliente-ativo': [
        'update_atendimento_status(realizado)',
        'generate_relatorio_resumo',
        'link_processos_if_any'
    ],
    'cliente-atualizado': [
        'parse_atualizacao_email',
        'diff_cliente_fields',
        'update_sqlite_changed_fields',
        'sync_google_contacts'
    ]
}
```

---

## 6. PRÓXIMOS PASSOS

### 6.1 Decisões Pendentes

1. **Criar tabelas `atendimentos` e `andamentos_processuais`?**
   - Sim, são necessárias para completude do modelo

2. **Relacionar `atendimentos` com `comunicacoes`?**
   - Sugestão: `comunicacoes` = contatos informais
   - `atendimentos` = reuniões formais com pauta

3. **Popular `vinculos_cliente_processo` para clientes existentes?**
   - 8 clientes, 1 processo → verificar se há relação

### 6.2 Migração Necessária

```sql
-- Aplicar após aprovação

-- 1. Criar tabela atendimentos
-- (ver schema acima)

-- 2. Criar tabela andamentos_processuais
-- (ver schema acima)

-- 3. Verificar vinculos existentes
SELECT c.nome, p.numero, v.qualidade, v.posicao
FROM clientes c
LEFT JOIN vinculos_cliente_processo v ON c.id = v.cliente_id
LEFT JOIN processos p ON v.processo_id = p.id;

-- 4. Registrar versão do schema
INSERT INTO schema_versao (versao, descricao) 
VALUES (2, 'Adiciona tabelas atendimentos e andamentos_processuais');
```

---

## 7. REFERÊNCIA CRUZADA COM OBSIDIAN

### 7.1 Estrutura de Pastas Proposta

```
secretario-agente-lke/
├── 10_REFERENCIAS/
│   └── secretario.db          ← SQLite hub
├── 20_CLIENTES/
│   ├── 21_Ativos/
│   │   └── maria-silva/
│   │       ├── ficha_cliente.md
│   │       ├── atendimentos/
│   │       │   ├── 2026-04-18-consulta-inicial.md
│   │       │   └── 2026-04-25-reuniao-processo.md
│   │       └── comunicacoes/
│   │           └── 2026-04-19-whatsapp-duvida.md
│   └── 22_Arquivados/
├── 30_PROCESSOS/
│   └── 0001234-56.2026.8.19.0000/
│       ├── processo.md
│       ├── andamentos/
│       │   ├── 2026-04-01-distribuicao.md
│       │   └── 2026-04-15-despacho-inicial.md
│       └── prazos/
│           └── 2026-04-30-contestacao.md
└── 40_AGENDA/
    └── reunioes/
        └── 2026-04-25-maria-silva-inventario.md
```

### 7.2 Link Obsidian ↔ SQLite

Cada nota Obsidian teria YAML frontmatter com referência ao SQLite:

```yaml
---
sqlite_table: atendimentos
sqlite_id: 42
cliente_id: 5
processo_id: 1
data: 2026-04-25
tipo: reuniao_processual
status: realizado
---
```

---

## 8. NOTAS TÉCNICAS (GEMINI)

Baseado nas informações do Gemini sobre Forms API:

### 8.1 Arquitetura Recomendada

```
Google Form ──► Google Sheet ──► Apps Script Trigger ──► Hermes
                (destino)        (onFormSubmit)           (webhook)
```

### 8.2 Não Usar

- ❌ Apps Script API (para criar código remotamente)
- ❌ Forms API sozinha (não faz link com tabela)

### 8.3 Usar

- ✅ Forms API (criar formulário programaticamente)
- ✅ Sheets API (ler/escrever na planilha destino)
- ✅ Apps Script vinculado (gatilho onFormSubmit)

### 8.4 Fluxo Proposto

1. Forms API: Criar formulário com `destinationId` (Sheet)
2. Apps Script: Vincular à planilha, criar `onFormSubmit`
3. Trigger: Ao receber resposta, gerar JSON + tag
4. Webhook: POST para endpoint do Hermes
5. Hermes: Processar, inserir SQLite, atualizar labels

---

## 9. TABELAS DE VAULTS (SATÉLITES)

### 9.1 Justificativa

Clientes podem ter múltiplos vaults, e vaults podem ser satélites criados para análise isolada de petição ou recurso, evitando contaminação de contexto.

### 9.2 Tabela: vaults

```sql
CREATE TABLE vaults (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT NOT NULL UNIQUE,
  path TEXT NOT NULL,
  tipo TEXT CHECK(tipo IN (
    'master',
    'satelite_caso',
    'satelite_analise',
    'satelite_peticao',
    'arquivado'
  )) NOT NULL,
  satelite_de INTEGER,
  caso_referencia TEXT,
  descricao TEXT,
  proposito TEXT,
  ativo BOOLEAN DEFAULT 1,
  obsidian_config BOOLEAN DEFAULT 0,
  sync_status TEXT DEFAULT 'nao_configurado',
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  atualizado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  arquivado_em TIMESTAMP,
  FOREIGN KEY (satelite_de) REFERENCES vaults(id)
);
```

### 9.3 Tabela: vinculos_cliente_vault

```sql
CREATE TABLE vinculos_cliente_vault (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  cliente_id INTEGER NOT NULL,
  vault_id INTEGER NOT NULL,
  papel TEXT CHECK(papel IN (
    'principal',
    'interessado',
    'contra',
    'testemunha',
    'referencia'
  )) NOT NULL,
  processo_relacionado TEXT,
  notas TEXT,
  ativo BOOLEAN DEFAULT 1,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  FOREIGN KEY (vault_id) REFERENCES vaults(id),
  UNIQUE(cliente_id, vault_id)
);
```

### 9.4 Vaults Existentes (Detectados)

| Nome | Path | Tipo Sugerido |
|------|------|---------------|
| inv_sa_02 | /media/peixoto/Portable/inv_sa_02 | master |
| hermes_lab | /media/peixoto/Portable/workspace/hermes_lab | satelite_analise |
| secretario-agente-lke | /media/peixoto/Portable/secretario-agente-lke | satelite_caso |
| caso-wasserman-anulatoria-adm-satelite | /media/peixoto/Portable/caso-wasserman-anulatoria-adm-satelite | satelite_caso |

---

## 10. MODELO RELACIONAL COMPLETO (VISUAL)

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐     ┌─────────────┐
│  clientes   │────│vinculos_cliente_ │────│   vaults    │────│ satelite_de │
│             │    │     vault        │    │             │    │   (self)    │
│ id, nome    │    │                  │    │ id, nome    │    └─────────────┘
│ cpf, ...    │    │ papel            │    │ tipo, path  │
└─────────────┘    └──────────────────┘    └─────────────┘
       │                                          │
       │    ┌──────────────────┐                  │
       └───►│vinculos_cliente_ │                  │
            │    processo      │                  │
            │                  │                  │
            └──────────────────┘                  │
                     │                            │
                     ▼                            ▼
            ┌─────────────┐              ┌──────────────────┐
            │  processos  │              │   atendimentos   │
            │             │              │                  │
            │ id, numero  │              │ vault_id (FK)    │
            └─────────────┘              │ cliente_id (FK)  │
                     │                   └──────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
┌─────────────────┐    ┌──────────────────────┐
│     prazos      │    │andamentos_processuais│
│                 │    │                      │
│ processo_id(FK) │    │ processo_id (FK)     │
└─────────────────┘    │ vault_id (FK)        │
                       └──────────────────────┘
```

---

## 11. QUESTÕES ABERTAS

1. **Implementar todas as tabelas novas?**
   - `atendimentos` ✓
   - `andamentos_processuais` ✓
   - `vaults` ✓
   - `vinculos_cliente_vault` ✓

2. **Popular tabelas com dados existentes?**
   - 4 vaults detectados
   - 8 clientes
   - 1 processo com vínculo

3. **Implementar lógica de decisão para criar satélites?**

4. **Seguir com abordagem Forms API + Apps Script ou fluxo manual primeiro?**
