---
session_id: "20260418_184648"
type: diagnostico
tags:
  - "#status/in-progress"
  - "#tipo/investigacao"
  - "#area/supabase-contacts"
created: "2026-04-18T19:30:00-03:00"
---

# Diagnóstico: Questões Prejudiciais - Supabase/SQLite

## Descobertas

### 1. Estrutura EXISTENTE no Secretário

O banco `secretario.db` JÁ POSSUI tabelas relacionadas:

| Tabela | Registros | Status |
|--------|-----------|--------|
| clientes | 8 | Parcialmente populado |
| processos | 1 | Quase vazio |
| vinculos_cliente_processo | 1 | Precisa sincronizar |
| comunicacoes | 0 | Vazio |
| prazos | 0 | Vazio |

### 2. Schema das Tabelas

```sql
-- Tabela CLIENTES (já existe)
CREATE TABLE clientes (
  id INTEGER PRIMARY KEY,
  nome TEXT NOT NULL,
  cpf TEXT UNIQUE,
  rg TEXT,
  email TEXT,
  telefone TEXT,
  endereco TEXT,
  ...
  ativo BOOLEAN DEFAULT 1
);

-- Tabela PROCESSOS (já existe)
CREATE TABLE processos (
  id INTEGER PRIMARY KEY,
  numero TEXT NOT NULL UNIQUE,
  tribunal TEXT NOT NULL,
  classe TEXT,
  ...
);

-- Tabela VINCULOS (já existe)
CREATE TABLE vinculos_cliente_processo (
  cliente_id INTEGER NOT NULL,
  processo_id INTEGER NOT NULL,
  qualidade TEXT NOT NULL,  -- autor, reu, advogado
  ...
);
```

### 3. Gaps Identificados

#### Gap 1: Dados Desconectados
- **Obsidian**: 33 fichas de clientes
- **SQLite**: 8 clientes cadastrados
- **Google Contacts**: 118 contatos (100 + 18 novos)
- **Problema**: Não há sincronização entre as 3 fontes

#### Gap 2: Vinculação Cliente-Processo
- Campo `google_resource_name` NÃO existe na tabela clientes
- Campo `obsidian_path` NÃO existe na tabela clientes
- Campo `ultimo_sync` NÃO existe na tabela clientes

#### Gap 3: Processos sem Dados
- Apenas 1 processo cadastrado
- Fichas do Obsidian referenciam múltiplos processos
- Necessário importar números de processo das fichas

### 4. Clientes no Banco vs Fichas Obsidian

| Cliente no Banco | Status no Obsidian |
|------------------|-------------------|
| Solange Loreto Vivas | ✅ Presente |
| Renato Loreto Vivas | ✅ Presente |
| André Luiz Loreto Vivas | ✅ Presente |
| João Silva | ❓ Não encontrado |
| Leonardo Tepedino | ❓ Possível caso separado |
| Ana Paula Tepedino | ❓ Possível caso separado |
| Renato Tepedino | ❓ Possível caso separado |
| Ana Maria Esteves | ❓ Não encontrada |

## Questões Prejudiciais (a resolver ANTES de criar tabelas)

### QP-1: Sincronização Obsidian → SQLite
**Pergunta:** Como popular a tabela `clientes` com os 33 clientes das fichas Obsidian?
**Ação:** Script de importação necessário

### QP-2: Migração Supabase
**Pergunta:** O SQLite será migrado para Supabase ou mantido como backup local?
**Contexto:** Projeto Supabase original foi pausado por inatividade
**Ação:** Decidir arquitetura final

### QP-3: Vinculação com Google Contacts
**Pergunta:** Como relacionar `clientes.id` com `google_resource_name`?
**Ação:** Adicionar colunas ao schema existente

### QP-4: Processos Órfãos
**Pergunta:** Os processos referenciados nas fichas precisam ser importados?
**Ação:** Extrair números de processo das fichas

### QP-5: Duplicatas e Conflitos
**Pergunta:** Clientes como "João Silva" no banco não correspondem às fichas
**Ação:** Auditoria de dados existentes

## Próximos Passos Revisados

### Fase 2.1: Diagnóstico e Planejamento (ATUAL)

1. [ ] **Auditoria de dados existentes**
   - Comparar 8 clientes do banco com 33 fichas
   - Identificar matches por nome/CPF
   - Mapear processos referenciados

2. [ ] **Consultar documentação via Context7**
   - `mcp_context7_resolve-library-id`: libraryName="supabase"
   - `mcp_context7_query-docs`: query="python client upsert insert"

3. [ ] **Decisão arquitetural**
   - SQLite local + Supabase cloud?
   - Migração completa para Supabase?
   - Híbrido com sync?

### Fase 2.2: Schema Evolution

1. [ ] **Adicionar colunas à tabela clientes**
   ```sql
   ALTER TABLE clientes ADD COLUMN google_resource_name TEXT;
   ALTER TABLE clientes ADD COLUMN obsidian_path TEXT;
   ALTER TABLE clientes ADD COLUMN ultimo_sync TIMESTAMP;
   ```

2. [ ] **Popular tabela com dados existentes**
   - Importar das fichas Obsidian
   - Vincular com Google Contacts
   - Validar duplicatas

### Fase 2.3: Sincronização

1. [ ] Script de importação Obsidian → SQLite
2. [ ] Script de sincronização SQLite ↔ Google Contacts
3. [ ] Cron job de manutenção

## Comandos Úteis

```bash
# Ver schema completo
sqlite3 10_REFERENCIAS/secretario.db ".schema clientes"

# Ver clientes existentes
sqlite3 10_REFERENCIAS/secretario.db "SELECT id, nome, email FROM clientes;"

# Contar vínculos
sqlite3 10_REFERENCIAS/secretario.db "SELECT COUNT(*) FROM vinculos_cliente_processo;"
```

## Documentação Relacionada

- `~/Downloads/Configuracao_Context7_OpenCode.md` - Configuração do Context7
- `/media/peixoto/Portable/secretario-agente-lke/30_IMPLEMENTACAO/secretario_cli.py` - CLI do Secretário
- `/media/peixoto/Portable/secretario-agente-lke/00_INBOX/FLUXO_INBOX.md` - Fluxo de processamento
