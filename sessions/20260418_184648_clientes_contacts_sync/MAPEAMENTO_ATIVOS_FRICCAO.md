# Mapeamento de Ativos - Sessão Clientes Contacts Sync

**Data:** 2026-04-19
**Tipo:** Análise de impacto e fricção

---

## 1. ATIVOS CRIADOS ESTA SESSÃO

### 1.1 Documentação (lke-processos-hub)

| Arquivo | Função | Impacto |
|---------|--------|---------|
| FLUXO_COMPLETO_CLIENTES.md | Arquitetura completa | Define como clientes fluem no sistema |
| MODELO_RELACIONAL_SQLITE.md | Schema + tabelas novas | Base para futuro desenvolvimento |
| ARQUITETURA_VAULTS_SATELITES.md | Hierarquia vaults | Afeta organização de casos |
| TEMPLATES_EMAIL_YAML.md | Parser e filtros | Integração com Gmail |
| MAPA_EVOLUCAO.md | Registro multi-repo | Template para futuras sessões |

### 1.2 Schema SQLite (secretario-agente-lke)

| Tabela | Criada | Status |
|--------|--------|--------|
| atendimentos | ✅ Esta sessão | Nova |
| andamentos_processuais | ✅ Esta sessão | Nova |
| vaults | ❌ Proposta | Pendente |
| vinculos_cliente_vault | ❌ Proposta | Pendente |

### 1.3 Sistema Multi-Repo

| Componente | Local | Status |
|------------|-------|--------|
| Memory PADRAO MULTI-REPO | Hermes | ✅ Ativo |
| Skill sessao-multi-repo-rastreamento | ~/.hermes/skills/workflow/ | ✅ Criada |
| Exemplo here.now | here-now-templates/examples/ | ✅ Publicado |

---

## 2. SKILLS RELACIONADAS EXISTENTES

### 2.1 Skills que Manipulam o Banco

| Skill | O que faz | Tabelas que usa | Ponto de fricção? |
|-------|-----------|-----------------|-------------------|
| secretario-agente-lke | Hub central | TODAS | **ALTO** - Schema mudou |
| sincronizacao-obsidian-google-contacts | Sync clientes | clientes, vinculos | **MÉDIO** - Precisa conhecer novas tabelas |
| secretario-validacao-cruzada-sync | Valida notas | clientes, processos | BAIXO - Não usa novas tabelas |
| sistema-gestao-sessoes-com-commits | Sessões | sessoes, atividade_logs | NENHUM |
| sistema-health-scoring-casos-juridicos | Score casos | clientes, processos | BAIXO |

### 2.2 Skills que Manipulam Repositórios

| Skill | O que faz | Repos envolvidos | Ponto de fricção? |
|-------|-----------|------------------|-------------------|
| sistema-criacao-repositorio-satelite-casos-juridicos | Cria satélites | Git local | NENHUM - não usa SQLite |
| sessao-multi-repo-rastreamento | Rastreia sessões | TODOS | NENHUM - nova skill |

---

## 3. PONTOS DE FRICÇÃO IDENTIFICADOS

### 3.1 ALTO: secretario-agente-lke

**Problema:** A skill descreve schema antigo (sem atendimentos, andamentos)

**Trechos desatualizados na skill:**
```
├── repositorios # Catálogo de repositórios monitorados
├── sessoes # Logs de sessões
├── artefatos # Documentos/código
├── atividade_logs # Commits, pushes, merges
├── credenciais # Vault para distribuição
├── tags # Categorização flexível
└── sessao_tags # Relacionamento N:N
```

**Falta:** Tabelas `atendimentos`, `andamentos_processuais`, `clientes`, `processos`, `vinculos_cliente_processo`

**Ação necessária:** Atualizar skill com schema atual

### 3.2 MÉDIO: sincronizacao-obsidian-google-contacts

**Problema:** Propõe ALTER TABLE que já não se aplica

**Trechos desatualizados:**
```sql
ALTER TABLE clientes ADD COLUMN google_contact_id TEXT;
ALTER TABLE clientes ADD COLUMN obsidian_path TEXT;
ALTER TABLE clientes ADD COLUMN sync_status TEXT DEFAULT 'pending';
ALTER TABLE clientes ADD COLUMN tipo TEXT DEFAULT 'cliente';
```

**Verificar:** Estas colunas já existem no schema atual?

### 3.3 BAIXO: Fluxo de inbox do secretario

**Problema:** Novas tabelas não são mencionadas no protocolo de notícia

**Protocolo atual:** Foca em skills, não em mudanças de schema

**Ação:** Adicionar seção sobre "Notícia de Mudança de Schema"

---

## 4. VERIFICAÇÃO DE COLUNAS EXISTENTES

Vou verificar se as colunas propostas já existem:

```bash
# Verificar schema atual da tabela clientes
sqlite3 secretario.db "PRAGMA table_info(clientes);"
```

---

## 5. AÇÕES NECESSÁRIAS

### 5.1 Imediato

| # | Ação | Prioridade |
|---|------|------------|
| 1 | Atualizar skill secretario-agente-lke com schema atual | ALTA |
| 2 | Verificar colunas propostas em sincronizacao-obsidian-google-contacts | ALTA |
| 3 | Atualizar skill sincronizacao-obsidian-google-contacts | MÉDIA |
| 4 | Adicionar protocolo de "Notícia de Mudança de Schema" | BAIXA |

### 5.2 Futuro

| # | Ação | Quando |
|---|------|--------|
| 1 | Criar tabela vaults e vinculos_cliente_vault | Após validação |
| 2 | Atualizar secretario_cli.py para novas tabelas | Quando implementar |
| 3 | Criar script de migração versionada | Antes de Supabase |

---

## 6. SKILLS ANTIGAS VS NOVA CONFIGURAÇÃO

### 6.1 Compatibilidade

| Skill | Compatível? | Observação |
|-------|-------------|------------|
| secretario-agente-lke | ⚠️ PARCIAL | Precita atualização de schema |
| sincronizacao-obsidian-google-contacts | ⚠️ PARCIAL | Colunas podem já existir |
| secretario-validacao-cruzada-sync | ✅ SIM | Não usa novas tabelas |
| sistema-gestao-sessoes-com-commits | ✅ SIM | Independente |
| sessao-multi-repo-rastreamento | ✅ SIM | Nova, já compatível |

### 6.2 Não Há Perda de Funcionalidade

As skills antigas continuam funcionando porque:

1. Novas tabelas são ADIÇÕES, não alterações
2. Colunas existentes não foram modificadas
3. FKs existentes continuam válidas
4. CLI secretario_cli.py continua funcionando

---

## 7. RECOMENDAÇÕES

### 7.1 Atualizar Skill Principal

Atualizar `secretario-agente-lke/SKILL.md` com:

```markdown
## Schema Atual (2026-04-19)

### Tabelas de Entidades Jurídicas
- clientes (8 registros)
- processos (1 registro)
- vinculos_cliente_processo (N:N com qualidade)
- comunicacoes (histórico de contatos)
- prazos (ligados a processos)

### Tabelas de Atendimento (NOVAS)
- atendimentos - Reuniões formais com pauta
- andamentos_processuais - Movimentações processuais

### Tabelas de Sistema
- repositorios, sessoes, artefatos, atividade_logs
- credenciais, tags, sessao_tags, skills
- schema_versao (controle de versões)
```

### 7.2 Atualizar Skill de Sincronização

Verificar se colunas propostas já existem antes de recomendar ALTER TABLE.

### 7.3 Documentar Mudanças

Criar protocolo para:
- Notificar mudanças de schema
- Versionar migrações
- Atualizar skills dependentes

---

## 8. PRÓXIMA SESSÃO

Antes de continuar, recomenda-se:

1. Atualizar skill secretario-agente-lke
2. Verificar schema real vs proposto
3. Decidir sobre tabelas vaults/vinculos_cliente_vault
4. Validar conceito com formulário manual

---

## 9. RESUMO

**Status:** Nenhuma skill foi QUEBRADA, mas algumas estão DESATUALIZADAS.

**Ação优先:** Atualizar documentação das skills existentes.

**Risco:** BAIXO - Novas tabelas não afetam funcionamento existente.
