# Mapa de Evolução - Sessão Clientes Contacts Sync

**Data:** 2026-04-19
**Escopo:** Desde sessão inicial até estado atual

---

## 1. PROBLEMA IDENTIFICADO

### Fragmentação de Informação

```
┌─────────────────────────────────────────────────────────────────┐
│                    ECOSISTEMA LKE ESPALHADO                     │
│                                                                 │
│  lke-processos-hub          secretario-agente-lke               │
│  ├── sessions/              ├── 10_REFERENCIAS/                 │
│  │   └── 184648_...         │   └── secretario.db (SQLite)     │
│  │       ├── HANDOFF.md     ├── 00_INBOX/                      │
│  │       ├── FLUXO_*.md     │   └── (entradas pendentes)        │
│  │       └── ...            └── 30_IMPLEMENTACAO/               │
│  │                               └── sync_supabase_agent.py     │
│  │                                                              │
│  hermes_lab                  caso-wasserman-*                   │
│  ├── .obsidian/              ├── 10-89_PROJECT/                 │
│  └── sessions/               │   └── 11_ficha_cliente.md        │
│                              └── ...                             │
│                                                                 │
│  inv_sa_02 (master)                                             │
│  └── (vault principal, templates)                               │
└─────────────────────────────────────────────────────────────────┘
```

**Questão:** Como registrar a evolução de uma ideia que passa por múltiplos repositórios?

---

## 2. SUA INTUIÇÃO ESTÁ CORRETA

### Por Que Este Mapa é Importante

1. **Rastreabilidade de Decisões**
   - Uma decisão tomada em um repositório afeta outros
   - Ex: "SQLite como hub central" (decidido em sessão, implementado em banco)

2. **Banco de Dados como ORQUESTRAADOR**
   - A tabela `repositorios` já existe no SQLite
   - Pode registrar: nome, path, owner, prioridade, status_git
   - **Novo:** campo `ultima_sessao_relacionada`

3. **Contexto Cruzado**
   - Clientes em `clientes` ↔ vaults em `vaults` ↔ repos em `repositorios`
   - O SQLite vira o "cemitério central" de metadados

---

## 3. O QUE MAIS PODERIA SER CONSIDERADO

### 3.1 Tabela de Sessões Cruzadas

```sql
-- Já existe: sessoes
-- Proposta: vincular com repositorios

ALTER TABLE sessoes ADD COLUMN repositorio_principal INTEGER 
  REFERENCES repositorios(id);

-- Nova tabela: sessoes_repositorios_secundarios
CREATE TABLE sessoes_repositorios_secundarios (
  sessao_id INTEGER NOT NULL,
  repositorio_id INTEGER NOT NULL,
  tipo_relacao TEXT CHECK(tipo_relacao IN (
    'fonte_dados',
    'destino_dados',
    'referencia',
    'vault_relacionado'
  )),
  PRIMARY KEY (sessao_id, repositorio_id)
);
```

### 3.2 Tabela de Decisões Arquiteturais

```sql
CREATE TABLE decisoes_arquiteturais (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  sessao_id INTEGER,
  titulo TEXT NOT NULL,
  decisao TEXT NOT NULL,
  impacto TEXT,              -- quais repositorios/sistemas afeta
  status TEXT CHECK(status IN (
    'proposta',
    'aprovada',
    'implementada',
    'depreciada'
  )),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (sessao_id) REFERENCES sessoes(id)
);
```

### 3.3 Link Obsidian ↔ SQLite

Cada nota de sessão deveria ter frontmatter apontando para o SQLite:

```yaml
---
sqlite_table: sessoes
sqlite_id: 42
repositorios_envolvidos:
  - lke-processos-hub
  - secretario-agente-lke
  - hermes_lab
decisoes:
  - SQLite como hub central
  - Validar conceito primeiro
---
```

---

## 4. CRONOLOGIA DA EVOLUÇÃO

### Sessão Inicial: 2026-04-18 18:23

**Contexto:** Investigação de fichas de clientes em vaults Obsidian

**Local:** `lke-processos-hub/sessions/20260418_182320_e27d06`

**Decisões iniciais:**
- Usar `google-workspace` skill para sync
- Localizar clientes em `inv_sa_02`
- Extrair dados de `.md` files

### Descobertas:

1. **33 arquivos de clientes** no vault principal
2. **18 contatos** já sincronizados com Google
3. **8 clientes** pendentes (sem dados completos)
4. **Skill `google-contacts-sync-obsidian`** já existe

### Evolução:

```
18:23 ───► Investigação inicial
           │
           ▼
       Descoberta: 8 clientes no SQLite
           │
           ▼
       Pergunta: SQLite ou Supabase?
           │
           ▼
       Decisão: SQLite como hub central
           │
           ▼
       Proposta: 6 cenários de fluxo
           │
           ▼
       Escolha: Gmail + Labels
           │
           ▼
       Detalhamento: Modelo relacional
           │
           ▼
       Insight: Vaults satélites
           │
           ▼
       Implementação: Tabelas criadas
           │
           ▼
       Registro: Commits + Push
           │
           ▼
       Agora: Mapear evolução
```

---

## 5. PROPSTA: NOTA DE EVOLUÇÃO

### 5.1 Onde Criar?

**Opção A:** No próprio SQLite (tabela `sessoes`)
- Pr: Centralizado, queryável
- Con: Não é Markdown (difícil edição)

**Opção B:** No vault principal (`inv_sa_02`)
- Pr: Editável em Obsidian
- Con: Vault pode mudar

**Opção C:** No repositório de processos (`lke-processos-hub`)
- Pr: Já é o "hub de sessões"
- Con: Fragmenta do vault

**Recomendação:** **Opção B + Link SQLite**
- Criar nota em `inv_sa_02/00-09_SYSTEM/90-99_META/`
- Registrar referência no SQLite

### 5.2 Estrutura da Nota

```markdown
# Mapa de Evolução - Clientes Contacts Sync

## Linha do Tempo

| Data/Hora | Repositório | Evento | Decisão |
|-----------|-------------|--------|---------|
| 2026-04-18 18:23 | lke-processos-hub | Sessão inicial | Investigar fichas |
| 2026-04-18 19:00 | secretario-agente-lke | Descoberta SQLite | 8 clientes encontrados |
| 2026-04-18 20:00 | lke-processos-hub | Análise cenários | 6 cenários propostos |
| 2026-04-18 22:00 | lke-processos-hub | Documentação | 15KB de especificações |
| 2026-04-19 00:06 | secretario-agente-lke | Implementação | Tabelas criadas |
| 2026-04-19 00:15 | lke-processos-hub | Commits | 19 arquivos commitados |

## Decisões Arquiteturais

1. **SQLite como Hub Central** (aprovada)
   - Impacto: secretario-agente-lke, todos os vaults
   - Racional: Dados sensíveis não vão para cloud

2. **Gmail + Labels como Trigger** (aprovada)
   - Impacto: Fluxo de entrada de clientes
   - Racional: Auditável, sem dependência de Sheets

3. **Vaults Satélites para Casos Complexos** (proposta)
   - Impacto: Arquitetura de vaults
   - Racional: Isolar contexto

4. **Validar Conceito Primeiro** (aprovada)
   - Impacto: Cronograma de implementação
   - Racional: "Não construir castelos sobre as nuvens"

## Repositórios Envolvidos

| Repo | Papel | URL |
|------|-------|-----|
| lke-processos-hub | Documentação de sessões | github.com/peixoto-ops/lke-processos-hub |
| secretario-agente-lke | SQLite hub, referências | github.com/peixoto-ops/secretario-agente-lke |
| hermes_lab | Ambiente de desenvolvimento | github.com/peixoto-ops/hermes_lab |
| inv_sa_02 | Vault master Obsidian | (local) |

## Arquivos Gerados

[lista completa com paths]

## Próximos Passos

[link para HANDOFF atualizado]
```

---

## 6. BENEFÍCIOS DO MAPEAMENTO

1. **Continuidade**
   - Nova sessão pode começar de onde parou
   - Não precisa reler 100KB de documentação

2. **Auditoria**
   - Rastreabilidade de decisões
   - Por que escolhemos X em vez de Y?

3. **Impacto**
   - Mudança em um repo → ver afetados
   - SQLite como índice central

4. **Aprendizado**
   - Padrões de evolução de ideias
   - Métodos que funcionaram

---

## 7. IMPLEMENTAÇÃO SUGERIDA

### Passo 1: Criar nota no vault master

```bash
mkdir -p /media/peixoto/Portable/inv_sa_02/00-09_SYSTEM/90-99_META/
touch /media/peixoto/Portable/inv_sa_02/00-09_SYSTEM/90-99_META/MAPA_EVOLUCAO_CLIENTES_CONTACTS_SYNC.md
```

### Passo 2: Popular tabela de decisões no SQLite

```sql
INSERT INTO decisoes_arquiteturais (sessao_id, titulo, decisao, impacto, status)
VALUES 
(1, 'SQLite como Hub Central', 'Usar SQLite local em vez de Supabase', 'secretario-agente-lke, todos os vaults', 'aprovada'),
(1, 'Gmail + Labels como Trigger', 'Usar labels Gmail como máquina de estados', 'Fluxo de clientes', 'aprovada'),
(1, 'Validar Conceito Primeiro', 'Implementar manualmente antes de automatizar', 'Cronograma', 'aprovada');
```

### Passo 3: Link cruzado

- Nota Obsidian → aponta para sessao_id no SQLite
- SQLite → aponta para path da nota
- Git commit → referencia ambos

---

## 8. QUESTÕES ABERTAS

1. **Onde criar a tabela `decisoes_arquiteturais`?**
   - No SQLite (recomendado)
   - Ou apenas como Markdown?

2. **Nome da nota de evolução?**
   - `MAPA_EVOLUCAO_*.md`
   - Ou `SESSAO_RESUMO_*.md`?

3. **Manter em inv_sa_02 ou em lke-processos-hub?**

4. **Popular `repositorios` no SQLite com os 4 envolvidos?**

---

## 9. PRÓXIMA AÇÃO

Aguardo sua confirmação para:
1. Criar a nota de evolução no local escolhido
2. Popular tabelas de decisão no SQLite
3. Estabelecer o padrão para futuras sessões
