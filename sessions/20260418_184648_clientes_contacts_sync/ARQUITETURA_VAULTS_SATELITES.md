# Arquitetura de Vaults - Descentralização por Caso/Análise

**Data:** 2026-04-18
**Status:** Especificação complementar

---

## 1. PROBLEMA: CONTAMINAÇÃO DE CONTEXTO

### 1.1 Cenário Problemático

Quando múltiplos casos/processos compartilham o mesmo vault Obsidian:

```
┌─────────────────────────────────────────────────┐
│              VAULT ÚNICO (PROBLEMA)             │
│                                                 │
│  Caso A ────┐                                  │
│             ├──► Mesmo contexto                 │
│  Caso B ────┤    ├─ Conflito de informações    │
│             │    ├─ Poluição de busca          │
│  Caso C ────┘    └─ Análises contaminadas      │
│                                                 │
│  Problemas:                                     │
│  - Buscas retornam resultados irrelevantes      │
│  - IA confunde fatos entre casos               │
│  - Templates/herdados misturam contextos       │
│  - Dificuldade de auditoria por caso           │
└─────────────────────────────────────────────────┘
```

### 1.2 Solução LKE: Vaults Satélites

```
┌─────────────────────────────────────────────────────────────────┐
│                    COFRE MASTER (inv_sa_02)                     │
│                                                                 │
│  - Templates globais                                           │
│  - Skills e metodologias                                        │
│  - Base de conhecimento reutilizável                            │
│  - Fichas de clientes (referência)                              │
│  - Links para satélites                                         │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  SATÉLITE A      │ │  SATÉLITE B      │ │  SATÉLITE C      │
│  Caso Tepedino   │ │  Caso Wasserman  │ │  Recurso X       │
│                  │ │                  │ │                  │
│  - Petições      │ │  - Anulação ADM  │ │  - Análise       │
│  - Andamentos    │ │  - Documentos    │ │    específica    │
│  - Notas         │ │  - Notas         │ │  - Isolamento    │
│  - Contexto      │ │  - Contexto      │ │    total         │
│    isolado       │ │    isolado       │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

---

## 2. MODELO: CLIENTE ↔ VAULTS

### 2.1 Relacionamento N:N

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────┐
│  clientes   │────────│ vinculos_cliente_ │────────│   vaults    │
│             │        │     vault        │         │             │
│ id, nome    │        │                  │         │ id, path    │
│ cpf, ...    │        │ cliente_id (FK)  │         │ nome        │
└─────────────┘        │ vault_id (FK)    │         │ tipo        │
                       │ papel            │         │ ativo       │
                       └──────────────────┘         │ satelite_de │
                                                    └─────────────┘
```

### 2.2 Tipos de Vault

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| `master` | Cofre principal, templates | `inv_sa_02` |
| `satelite_caso` | Criado para caso específico | `caso-wasserman-anulatoria-adm-satelite` |
| `satelite_analise` | Criado para análise específica | `recurso-apelacao-123-analise` |
| `satelite_peticao` | Criado para petição específica | `contestacao-456-draft` |
| `arquivado` | Satélite finalizado | `caso-xyz-arquivado` |

### 2.3 Papéis do Cliente no Vault

| Papel | Descrição |
|-------|-----------|
| `principal` | Cliente principal do caso |
| `interessado` | Interessado no processo |
| `contra` | Parte contrária (para referência) |
| `testemunha` | Testemunha no processo |
| `referencia` | Apenas referência cruzada |

---

## 3. SCHEMA SQL: TABELA VAULTS

### 3.1 Nova Tabela

```sql
CREATE TABLE vaults (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  
  -- Identificação
  nome TEXT NOT NULL UNIQUE,
  path TEXT NOT NULL,              -- caminho absoluto
  
  -- Classificação
  tipo TEXT CHECK(tipo IN (
    'master',
    'satelite_caso',
    'satelite_analise',
    'satelite_peticao',
    'arquivado'
  )) NOT NULL,
  
  -- Hierarquia
  satelite_de INTEGER,             -- id do vault pai (se satélite)
  caso_referencia TEXT,            -- número do processo se aplicável
  
  -- Metadados
  descricao TEXT,
  proposito TEXT,                  -- objetivo do vault
  
  -- Status
  ativo BOOLEAN DEFAULT 1,
  
  -- Integração
  obsidian_config BOOLEAN DEFAULT 0,  -- tem pasta .obsidian
  sync_status TEXT CHECK(sync_status IN (
    'nao_configurado',
    'ativo',
    'pausado',
    'erro'
  )) DEFAULT 'nao_configurado',
  
  -- Auditoria
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  atualizado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  arquivado_em TIMESTAMP,
  
  FOREIGN KEY (satelite_de) REFERENCES vaults(id)
);

CREATE INDEX idx_vaults_tipo ON vaults(tipo);
CREATE INDEX idx_vaults_satelite_de ON vaults(satelite_de);
CREATE INDEX idx_vaults_path ON vaults(path);
```

### 3.2 Tabela de Vínculos

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
  
  -- Metadados específicos do vínculo
  processo_relacionado TEXT,      -- número do processo se aplicável
  notas TEXT,
  
  ativo BOOLEAN DEFAULT 1,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  FOREIGN KEY (vault_id) REFERENCES vaults(id),
  UNIQUE(cliente_id, vault_id)
);

CREATE INDEX idx_vinculos_cliente_vault_cliente ON vinculos_cliente_vault(cliente_id);
CREATE INDEX idx_vinculos_cliente_vault_vault ON vinculos_cliente_vault(vault_id);
```

---

## 4. DIAGRAMA RELACIONAL ATUALIZADO

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
┌─────────────────┐        ┌─────────────────┐        ┌─────────────────────┐
│   atendimentos  │        │  comunicacoes   │        │ vinculos_cliente_   │
│                 │        │                 │        │      vault          │
│ cliente_id (FK) │        │ cliente_id (FK) │        │                     │
│ vault_id (FK)   │◄───────│                 │        │ cliente_id (FK)     │
│ processo_id(FK) │        └─────────────────┘        │ vault_id (FK)       │
│ data_hora, tipo │                                   │ papel               │
└─────────────────┘                                   └─────────────────────┘
         │                                                      │
         │                                                      ▼
         │                                           ┌─────────────────────┐
         │                                           │       vaults        │
         │                                           │                     │
         │                                           │ id, nome, path      │
         │                                           │ tipo, satelite_de    │
         │                                           └─────────────────────┘
         │                                                      │
         │                           ┌──────────────────────────┘
         │                           │
         ▼                           ▼
┌──────────────────────┐    ┌─────────────────────┐
│andamentos_processuais│    │ vinculos_cliente_   │
│                      │    │     processo        │
│ processo_id (FK)     │    │                     │
│ vault_id (FK) ◄──────┼────│ (já existe)         │
│ data_publicacao      │    └─────────────────────┘
└──────────────────────┘                 │
                                         ▼
                                ┌─────────────────┐
                                │    processos    │
                                │                 │
                                │ id, numero, ... │
                                └─────────────────┘
```

---

## 5. FLUXO DE CRIAÇÃO DE SATÉLITE

### 5.1 Gatilhos para Criar Satélite

| Gatilho | Tipo de Satélite | Motivo |
|---------|------------------|--------|
| Novo caso complexo | `satelite_caso` | Isolar documentação |
| Análise de recurso | `satelite_analise` | Evitar contaminação |
| Petição importante | `satelite_peticao` | Rascunhos isolados |
| Consulta inicial | Usar master | Ainda não há caso |
| Caso simples | Usar master | Não justifica satélite |

### 5.2 Fluxo Automatizado

```
[DETECÇÃO DE NECESSIDADE]
         │
         ▼
[AVALIAÇÃO: caso merece satélite?]
         │
    ┌────┴────┐
    ▼         ▼
  [SIM]     [NÃO]
    │         │
    ▼         ▼
[criar_vault_satelite]  [usar_master]
    │
    ▼
[configurar .obsidian]
    │
    ▼
[registrar em vaults (SQLite)]
    │
    ▼
[criar vínculo cliente_vault]
    │
    ▼
[popular templates do master]
    │
    ▼
[notificar criação]
```

---

## 6. TABELA DE VAULTS EXISTENTES

### 6.1 Vaults Detectados (da sessão anterior)

| Nome | Path | Tipo | Status |
|------|------|------|--------|
| hermes_lab | `/media/peixoto/Portable/workspace/hermes_lab` | master? | Ativo (119 .md) |
| secretario-agente-lke | `/media/peixoto/Portable/secretario-agente-lke` | satelite? | Ativo (117 .md) |
| caso-wasserman-anulatoria-adm-satelite | `/media/peixoto/Portable/caso-wasserman-anulatoria-adm-satelite` | satelite_caso | Ativo |

### 6.2 Script para Popular Tabela

```sql
-- Inserir vaults existentes
INSERT INTO vaults (nome, path, tipo, satelite_de, obsidian_config, sync_status) VALUES
('inv_sa_02', '/media/peixoto/Portable/inv_sa_02', 'master', NULL, 1, 'ativo'),
('hermes_lab', '/media/peixoto/Portable/workspace/hermes_lab', 'satelite_analise', 
 (SELECT id FROM vaults WHERE nome = 'inv_sa_02'), 1, 'ativo'),
('secretario-agente-lke', '/media/peixoto/Portable/secretario-agente-lke', 'satelite_caso',
 (SELECT id FROM vaults WHERE nome = 'inv_sa_02'), 1, 'ativo'),
('caso-wasserman-anulatoria-adm-satelite', '/media/peixoto/Portable/caso-wasserman-anulatoria-adm-satelite', 'satelite_caso',
 (SELECT id FROM vaults WHERE nome = 'inv_sa_02'), 1, 'ativo');
```

---

## 7. IMPACTO NAS LABELS GMAIL

### 7.1 Labels Atualizadas com Conceito de Vaults

| Label | Ação | Relação com Vault |
|-------|------|-------------------|
| `cliente-novo` | Criar cliente | - |
| `cliente-validado` | Criar contato | - |
| `cliente-caso-novo` | Criar satélite? | Avaliar necessidade |
| `cliente-agenda` | Criar atendimento | Vincular ao vault do caso |
| `cliente-reuniao` | Confirmar | Atualizar vault |
| `cliente-ativo` | Processar | Vincular processo ↔ vault |
| `cliente-satelite-criado` | Notificar | Registro de novo vault |

### 7.2 Automação de Criação de Satélite

```python
def should_create_satellite(cliente_id: int, processo_id: int, caso_info: dict) -> bool:
    """Determina se caso merece vault satélite."""
    
    # Critérios para criar satélite
    criterios = {
        'complexidade': caso_info.get('urgencia', 1) >= 4,
        'documentos_esperados': caso_info.get('documentos_estimados', 0) > 20,
        'multiplas_partes': len(caso_info.get('partes', [])) > 3,
        'recurso': 'recurso' in caso_info.get('tipo_atendimento', '').lower(),
        'valor_causa': caso_info.get('valor_causa', 0) > 100000,
    }
    
    # Se 2+ critérios atendidos, criar satélite
    return sum(criterios.values()) >= 2

def create_satellite_vault(cliente_id: int, processo_numero: str, tipo: str) -> int:
    """Cria vault satélite e registra no SQLite."""
    
    # Gerar nome
    nome = f"caso-{processo_numero.replace('.', '-')}-satelite"
    path = f"/media/peixoto/Portable/{nome}"
    
    # Criar estrutura
    os.makedirs(path)
    os.makedirs(f"{path}/.obsidian")
    
    # Registrar no SQLite
    vault_id = db.execute(
        "INSERT INTO vaults (nome, path, tipo, satelite_de) VALUES (?, ?, ?, ?)",
        (nome, path, tipo, MASTER_VAULT_ID)
    )
    
    # Criar vínculo cliente-vault
    db.execute(
        "INSERT INTO vinculos_cliente_vault (cliente_id, vault_id, papel) VALUES (?, ?, 'principal')",
        (cliente_id, vault_id)
    )
    
    return vault_id
```

---

## 8. RELAÇÃO COM FORMULÁRIOS

### 8.1 Campo Adicional no Form de Cadastro

```
| Campo | Tipo | Obrigatório | Observação |
|-------|------|-------------|------------|
| Número do processo | Texto | ❌ | Se já tiver processo |
| Complexidade do caso | Escala | ❌ | 1-5 para avaliar necessidade de satélite |
| Documentos disponíveis | Número | ❌ | Estimativa de volume |
```

### 8.2 Fluxo Pós-Formulário

```yaml
# Se processo informado E complexidade >= 4:
- criar_satelite_vault:
    nome: "caso-{processo}-satelite"
    tipo: "satelite_caso"
    
- registrar_vinculos:
    cliente_vault: principal
    processo_vault: referência
    
- notificar_criacao:
    template: "satelite_criado"
    destinatario: "luizpeixoto.adv@gmail.com"
```

---

## 9. PRÓXIMOS PASSOS

1. **Criar tabelas `vaults` e `vinculos_cliente_vault`?**
2. **Popular com vaults existentes?**
3. **Implementar lógica de decisão para criar satélites?**
4. **Atualizar formulário com campos de avaliação?**

---

## 10. NOTAS

- A criação de satélites pode ser manual ou automática
- O threshold de complexidade é configurável
- Satélites arquivados permanecem vinculados ao master
- A auditoria de contexto isolado é facilitada
