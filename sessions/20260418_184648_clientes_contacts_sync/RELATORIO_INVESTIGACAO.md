---
session_id: "20260418_184648"
type: relatorio
tags:
  - "#status/in-progress"
  - "#tipo/investigacao"
  - "#area/contacts-sync"
created: "2026-04-18T18:46:48-03:00"
---

# Relatório de Investigação: Fichas de Clientes vs Google Contacts

## Objetivo
Descobrir onde estão as fichas novas dos clientes salvos em vaults Obsidian e cruzar dados com API Contacts do Google Workspace.

## Descobertas

### Vaults com Pastas de Clientes

| Vault | Notas | Última Modificação |
|-------|-------|-------------------|
| inv_sa_02 | 33 | 2025-12-27 |
| SST-Juridico | 30 | 2025-09-18 |

**Vault principal selecionado:** `inv_sa_02` (mais recente)

### Estrutura das Fichas

Localização: `/media/peixoto/Portable/inv_sa_02/01 - Linha do Tempo/01 - Clientes/`

Formato identificado:
- Frontmatter YAML com metadados
- Corpo Markdown com dados pessoais
- Campos: nome_completo, CPF, RG, email, telefone, endereço

### Dados Estatísticos das Fichas

| Campo | Quantidade | % |
|-------|-----------|---|
| Com CPF | 27 | 82% |
| Com email | 17 | 52% |
| Com telefone | 11 | 33% |

## Cruzamento com Google Contacts

### Contatos Google
- Total: 100 contatos
- API: People API v1 via `gws people people connections list`

### Resultado do Matching

| Status | Quantidade |
|--------|-----------|
| Encontrados no Google | 7 |
| Não encontrados (novos) | 26 |

### Matches Identificados

| Cliente Obsidian | Contato Google | Resource Name |
|-----------------|----------------|---------------|
| RODRIGO DE SOUZA DANTAS MENDONÇA PINTO | souza | people/c3393545308922210672 |
| JOYCE FAISLON BRAVO | joyce | people/c8927143003767404859 |
| BRUNA FERNANDA DIAS LIMA MORAES | bruna | people/c4608502103517779397 |
| ANTÔNIO RICARDO SOARES COSTA | ricardo | people/c4022035677332528245 |
| ALEXANDRE BARATA LYDIA | alexandre | people/c8457268360996094413 |

### Clientes a Criar no Google Contacts

1. HENRIQUE DE MOURA NEVES
2. ISAQUE FERNANDO DIAS LIMA MORAES
3. CRISTINA RIBEIRO RIGUETTI PINTO
4. MARTA LUIZA DAMASCO DE SÁ
5. TULA FAISLON BRAVO
6. TEREZA DUTRA REIS DE MOURA NEVES
7. SOLANGE LORETO VIVAS
8. RODRIGO OLIVEIRA DE SIQUEIRA
9. LUIZ PEIXOTO DE SIQUEIRA
10. PATRÍCIA WASSERMAN
... (mais 16)

## Próximos Passos

1. **CONTATOS CRIADOS**: 18 clientes novos sincronizados com Google Contacts
2. **CONTATOS ATUALIZADOS**: 3 clientes com dados complementados
3. **PENDENTES**: 8 clientes sem email/telefone (precisam de dados)
4. **SKILL CRIADA**: `google-contacts-sync-obsidian` para futuras sincronizações

## Recomendações Futuras

### 1. Preencher Dados Faltantes
8 clientes não têm email/telefone nas fichas. Ações:
- Revisar processos físicos para extrair contatos
- Entrar em contato com clientes para atualização cadastral
- Considerar campo "sem contato" para tracking

### 2. Sincronização Automática
Criar script de sincronização periódica (cronjob):
```bash
# Verificar novos clientes no Obsidian e sincronizar
python scripts/sync_contacts.py --vault /media/peixoto/Portable/inv_sa_02 --dry-run
```

### 3. Enriquecimento Bidirecional
- Obsidian → Google: Nome, email, telefone, CPF
- Google → Obsidian: Foto, organização, aniversário
- Implementar detecção de conflitos

### 4. Integração com Secretary Agent
- Tabela `clientes` no Supabase com `google_resource_name`
- Tracking de última sincronização
- Alertas para dados desatualizados

### 5. Context7 MCP (sugestão)
Considerar adicionar MCP Context7 para consultas mais rápidas à documentação de APIs sem navegar página por página.

## Comandos Úteis

```bash
# Listar contatos
gws people people connections list --params '{"resourceName":"people/me","personFields":"names,emailAddresses,phoneNumbers"}'

# Criar contato
gws people people createContact --params '{"names":[{"displayName":"NOME"}],"emailAddresses":[{"value":"email@example.com"}]}'
```

## Arquivos da Sessão

- `clients_data.json` - Dados extraídos das fichas
- `google_contacts.json` - Contatos do Google
- `match_report.json` - Resultado do cruzamento
