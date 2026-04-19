---
session_id: "20260418_184648"
type: plano-integracao
tags:
  - "#status/planejamento"
  - "#tipo/integracao"
  - "#area/supabase-contacts"
created: "2026-04-18T22:50:00-03:00"
---

# Plano: Integração Supabase + Obsidian + Google Contacts

## Objetivo
Criar sincronização bidirecional entre fichas de clientes do Obsidian, Google Contacts e tabela Supabase.

## Arquitetura Proposta

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Obsidian      │────▶│    Supabase     │◀────│ Google Contacts │
│   (inv_sa_02)   │     │   (tabela)      │     │   (People API)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        └───────────────────────┴───────────────────────┘
                       sync_contacts.py
```

## Tabela Supabase: clientes

```sql
CREATE TABLE clientes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  nome_completo TEXT NOT NULL,
  email TEXT,
  telefone TEXT,
  cpf TEXT,
  rg TEXT,
  endereco TEXT,
  profissao TEXT,
  estado_civil TEXT,
  
  -- Obsidian
  obsidian_path TEXT,
  obsidian_filename TEXT,
  
  -- Google Contacts
  google_resource_name TEXT,
  google_synced_at TIMESTAMPTZ,
  
  -- Controle
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  sync_status TEXT DEFAULT 'pending',
  
  UNIQUE(cpf)
);

CREATE INDEX idx_clientes_email ON clientes(email);
CREATE INDEX idx_clientes_google_resource ON clientes(google_resource_name);
CREATE INDEX idx_clientes_sync_status ON clientes(sync_status);
```

## Fluxo de Sincronização

### 1. Obsidian → Supabase (entrada)
```python
# Para cada ficha .md na pasta de clientes:
# 1. Extrair dados do frontmatter + corpo
# 2. Verificar se existe na tabela (por CPF)
# 3. Inserir ou atualizar
```

### 2. Supabase → Google Contacts (saída)
```python
# Para cada registro com sync_status='pending':
# 1. Se sem google_resource_name: criar contato
# 2. Se com google_resource_name: atualizar contato
# 3. Marcar sync_status='synced'
```

### 3. Google Contacts → Supabase (enriquecimento)
```python
# Para cada contato com dados novos:
# 1. Verificar organização, foto, aniversário
# 2. Atualizar tabela se houver mudanças
```

## Próximos Passos

1. **Usar Context7 MCP** para consultar documentação Supabase Python SDK
2. Criar tabela no Supabase
3. Implementar script `sync_to_supabase.py`
4. Implementar script `sync_from_google.py`
5. Criar cronjob para sincronização periódica

## Comandos Context7 (após restart)

```
mcp_context7_resolve-library-id:
  libraryName: "supabase-python"

mcp_context7_query-docs:
  libraryId: "/supabase/supabase-py"
  query: "insert upsert python client"
```

## Dependências

- supabase-py: `pip install supabase`
- Variáveis de ambiente:
  - SUPABASE_URL
  - SUPABASE_ANON_KEY
  - SUPABASE_SERVICE_KEY (para operações admin)

## Cronjob Sugerido

```bash
# Sincronizar a cada hora
0 * * * * ~/.hermes/skills/juridico/google-contacts-sync-obsidian/scripts/sync_contacts.py --vault /media/peixoto/Portable/inv_sa_02 --create-new
```
