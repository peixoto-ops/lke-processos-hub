# Plano de Análise: Integração Obsidian ↔ Google Contacts ↔ Supabase

> **Para implementação:** Este é um plano de ANÁLISE, não de execução. Foco em entender o problema antes de propor soluções.

**Objetivo:** Avaliar o problema de sincronização entre três fontes de dados de clientes antes de propor qualquer implementação.

**Princípio:** KISS - Keep It Simple, Stupid. Evitar over-engineering.

---

## Questões Prejudiciais (devem ser resolvidas ANTES de qualquer implementação)

### QP-1: Centralização de Dados
**Pergunta:** Onde deve ficar a "verdade" dos dados de clientes?
- Opção A: Obsidian (atual fonte primária)
- Opção B: Supabase (banco relacional)
- Opção C: Google Contacts (já usado para comunicação)

**Dependência:** Esta decisão afeta toda a arquitetura.

### QP-2: Supabase Local vs Cloud
**Pergunta:** Como funciona o Supabase local (Docker)?
**Ação:** Consultar documentação via Context7 sobre self-hosted Supabase.

### QP-3: Necessidade Real
**Pergunta:** Qual problema estamos resolvendo?
- Contatos desatualizados no Google?
- Fichas desconectadas do banco?
- Processos sem vinculação com clientes?

### QP-4: Escopo do MVP
**Pergunta:** Qual o mínimo viável para entregar valor?
- Sincronização unidirecional (Obsidian → Google)?
- Sincronização bidirecional completa?
- Apenas consulta/auditoria?

---

## Fontes de Dados Atuais

### Fonte 1: Obsidian (`inv_sa_02`)
- **Local:** `/media/peixoto/Portable/inv_sa_02/01 - Linha do Tempo/01 - Clientes/`
- **Quantidade:** 33 fichas markdown
- **Dados:** nome, CPF, email, telefone, processos vinculados
- **Status:** Fonte primária atual

### Fonte 2: SQLite (`secretario.db`)
- **Local:** `/media/peixoto/Portable/secretario-agente-lke/10_REFERENCIAS/secretario.db`
- **Tabelas:** clientes (8), processos (1), vinculos (1)
- **Status:** Parcialmente populado, desconectado

### Fonte 3: Google Contacts
- **Quantidade:** 118 contatos
- **API:** People API via `gws` CLI
- **Status:** Sincronizado parcialmente (18 novos criados hoje)

---

## Análise de Riscos

### Risco 1: Over-Engineering
**Sintoma:** Criar sistema complexo antes de entender necessidade real
**Mitigação:** Começar com sync unidirecional simples

### Risco 2: Fontes Divergentes
**Sintoma:** Três fontes com dados diferentes
**Mitigação:** Definir UMA fonte como verdade

### Risco 3: Premature Optimization
**Sintoma:** Criar sync bidirecional antes de validar valor
**Mitigação:** MVP = Obsidian → Google Contacts apenas

---

## Próximos Passos (Análise, não implementação)

### Passo 1: Consultar Context7 sobre Supabase
- Como funciona self-hosted?
- Qual a complexidade de setup?
- Vale a pena vs manter SQLite?

### Passo 2: Definir Fonte de Verdade
- Consultar usuário sobre preferência
- Documentar decisão

### Passo 3: Avaliar MVP
- Qual o menor passo que entrega valor?
- Sincronização simples vs sistema completo?

---

## Localização dos Arquivos de Análise

**Atenção:** Estes arquivos são de IDEAÇÃO, não de projeto consolidado.

```
/media/peixoto/Portable/lke-processos-hub/sessions/20260418_184648_clientes_contacts_sync/
├── RELATORIO_INVESTIGACAO.md     # O que encontramos
├── DIAGNOSTICO_QUESTOES_PREJUDICIAIS.md  # O que precisamos resolver
└── PLANO_ANALISE.md              # Este arquivo (como pensar sobre o problema)
```

**NÃO criar:**
- Scripts de implementação
- Estrutura de projeto
- Commits de código

**SIM criar:**
- Documentação de análise
- Perguntas a responder
- Decisões a tomar
