---
session_id: "20260418_184648"
type: roadmap
tags:
  - "#status/in-progress"
  - "#tipo/integracao"
  - "#area/supabase-contacts"
created: "2026-04-18T19:00:00-03:00"
---

# Roadmap: Integração Obsidian ↔ Google Contacts ↔ Supabase

## Fase 1: CONCLUÍDA ✓

### 1.1 Investigação de Fontes
- [x] Identificar vaults Obsidian com fichas de clientes
- [x] Encontrado: `inv_sa_02` (33 fichas)
- [x] Mapear estrutura das fichas (frontmatter + corpo)

### 1.2 Sincronização Inicial
- [x] Listar contatos Google existentes (100)
- [x] Criar 18 contatos novos
- [x] Atualizar 3 contatos existentes
- [x] Identificar 8 clientes pendentes (sem dados)

### 1.3 Infraestrutura
- [x] Configurar Context7 MCP no Hermes
- [x] Criar skill `google-contacts-sync-obsidian`
- [x] Criar script `sync_contacts.py`
- [x] Renovar token OAuth2

## Fase 2: PLANEJAMENTO

### 2.1 Supabase - Tabela Clientes
```sql
CREATE TABLE clientes (
  id SERIAL PRIMARY KEY,
  nome_completo TEXT NOT NULL,
  email TEXT,
  telefone TEXT,
  cpf TEXT UNIQUE,
  google_resource_name TEXT,
  obsidian_path TEXT,
  ultimo_sync TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 2.2 Consultar Documentação (Context7)
```
1. mcp_context7_resolve-library-id: libraryName="supabase"
2. mcp_context7_query-docs: query="insert upsert python client"
```

### 2.3 Scripts Necessários
- [ ] `sync_to_supabase.py` - Envia dados para Supabase
- [ ] `sync_from_supabase.py` - Atualiza Obsidian com dados do Supabase
- [ ] `full_sync.py` - Sincronização completa bidirecional

## Fase 3: IMPLEMENTAÇÃO

### 3.1 Backend Supabase
1. Criar tabela `clientes`
2. Configurar RLS (Row Level Security)
3. Criar função `upsert_cliente()`
4. Adicionar triggers para tracking

### 3.2 Scripts de Sincronização
1. Implementar cliente Supabase Python
2. Detectar conflitos (Obsidian vs Google vs Supabase)
3. Resolver com merge automático ou alerta

### 3.3 Automação
1. Cronjob semanal de sincronização
2. Alertas para dados desatualizados
3. Dashboard de status

## Pendências Imediatas

### 8 Clientes sem Contato
| Nome | Ação |
|------|------|
| ISAQUE FERNANDO DIAS LIMA MORAES | Revisar processo físico |
| TEREZA DUTRA REIS DE MOURA NEVES | Revisar processo físico |
| LUIZ PEIXOTO DE SIQUEIRA | Verificar se é cliente |
| MATEUS FERNANDO DIAS LIMA MORAES | Revisar processo físico |
| ... | ... |

## Comandos Úteis

```bash
# Sincronizar clientes
python ~/.hermes/skills/juridico/google-contacts-sync-obsidian/scripts/sync_contacts.py \
  --vault /media/peixoto/Portable/inv_sa_02 \
  --list-missing

# Consultar doc Supabase (Context7)
# Usar ferramenta: mcp_context7_resolve-library-id
# Depois: mcp_context7_query-docs
```

## Métricas

| Métrica | Valor |
|---------|-------|
| Clientes no Obsidian | 33 |
| Contatos Google | 118 (100 + 18 novos) |
| Sincronizados | 21 |
| Pendentes | 8 |
| Vault principal | inv_sa_02 |
