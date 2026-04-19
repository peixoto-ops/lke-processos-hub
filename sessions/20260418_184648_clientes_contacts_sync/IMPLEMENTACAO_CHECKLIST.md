# Checklist de Implementação - Fluxo de Clientes

## Fase 1: Validação (IMEDIATA)

### 1.1 Schema SQLite
- [ ] Adicionar colunas necessárias:
```sql
ALTER TABLE clientes ADD COLUMN google_contact_id TEXT;
ALTER TABLE clientes ADD COLUMN obsidian_path TEXT;
ALTER TABLE clientes ADD COLUMN sync_status TEXT DEFAULT 'pending';
ALTER TABLE clientes ADD COLUMN tipo TEXT DEFAULT 'cliente';
```

### 1.2 Template Obsidian
- [x] Template criado
- [ ] Mover para `inv_sa_02/01 - Linha do Tempo/01 - Clientes/_templates/`

### 1.3 Teste Manual
- [ ] Importar 1 ficha para SQLite manualmente
- [ ] Verificar relacionamentos N:N funcionando

## Fase 2: Google Forms (QUANDO PRONTO)

### 2.1 Criar Formulário
- Campos: nome, cpf, email, telefone, tipo_atendimento
- Conectar ao Google Sheets

### 2.2 Script de Importação
- [ ] Criar script que lê Sheets → SQLite
- [ ] Criar ficha Obsidian a partir do SQLite

## Fase 3: Sincronização (FUTURO)

### 3.1 Scripts Definitivos
- [ ] sync_obsidian_to_sqlite.py
- [ ] sync_sqlite_to_contacts.py
- [ ] sync_all.py (orquestrador)

---

## Status Atual: IDEAÇÃO COMPLETA

**Não implementar até:**
1. Validar template com usuário
2. Testar importação manual
3. Confirmar fluxo Google Forms

---

## Arquivos Criados Esta Sessão

```
sessions/20260418_184648_clientes_contacts_sync/
├── RELATORIO_INVESTIGACAO.md          # O que encontramos
├── DIAGNOSTICO_QUESTOES_PREJUDICIAIS.md  # Problemas identificados
├── PLANO_ANALISE.md                   # Como pensar o problema
├── ANALISE_SUPABASE_VS_SQLITE.md      # Decisão técnica
├── ANALISE_CAMINHOS_DADOS.md          # Opções de fluxo
├── DECISAO_ARQUITETURAL.md            # Decisão final
├── TEMPLATE_FICHA_CLIENTE.md          # Template padronizado
└── IMPLEMENTACAO_CHECKLIST.md         # Este arquivo
```

## Próxima Sessão

1. Validar template com fichas reais
2. Executar ALTER TABLE no SQLite
3. Testar importação de 1 cliente
4. Criar Google Form piloto
