# HANDOFF - Sessão Clientes Contacts Sync

**Data:** 2026-04-18 22:45
**Status:** Pausado para nova sessão

---

## ONDE PARAMOS

### Tarefa Principal
Investigar fichas de clientes em vaults Obsidian e cruzar dados com Google Contacts via SQLite.

### O Que Foi Descoberto

1. **Arquitetura Confirmada:**
   - SQLite como hub central (NÃO Supabase)
   - Local: `/media/peixoto/Portable/secretario-agente-lke/10_REFERENCIAS/secretario.db`
   - Tabela `clientes` tem 8 registros

2. **Fichas de Clientes Encontradas:**
   - `caso-wasserman-anulatoria-adm-satelite/10-89_PROJECT/10-19_CONTEXT/11_ficha_cliente.md`
   - Template com YAML frontmatter
   - Campos: nome, cpf, rg, email, telefone, endereço

3. **Sincronização Já Implementada:**
   - 18 clientes criados no Google Contacts
   - 3 clientes atualizados
   - Skill `google-contacts-sync-obsidian` existe

4. **Cenários Propostos:**
   - **CENÁRIO 1 (recomendado):** Gmail + Labels como trigger
   - Formulário → Email com tag `cliente-novo` → Parser → SQLite
   - Labels como máquina de estados

---

## ARQUIVOS CRIADOS ESTA SESSÃO

```
/media/peixoto/Portable/lke-processos-hub/sessions/20260418_184648_clientes_contacts_sync/
├── WORKFLOW_COORDENADO.md      # Análise de skills e fluxo
└── CENARIOS_INTELIGENTES.md    # 6 cenários propostos
```

---

## DECISÕES TOMADAS (2026-04-18 23:45)

1. ✅ Criar tabelas `atendimentos` e `andamentos_processuais` - **EXECUTADO**
2. ✅ Popular dados existentes - **EXECUTADO** (8 clientes, 1 processo)
3. ✅ Validar conceito primeiro (fluxo manual antes de automatizar) - **CONFIRMADO**
4. ✅ Schema versão 3 registrado

## PRÓXIMO PASSO IMEDIATO

- Criar Google Form de cadastro manualmente
- Testar com 1 cliente real
- Validar parser YAML

---

## MAPA DE EVOLUÇÃO CRIADO

**Local:** `/media/peixoto/Portable/workspace/hermes_lab/2026-04-18--clientes-contacts-sync-mapa/`

**Conteúdo:**
- `MAPA_EVOLUCAO.md` - Registro centralizado multi-repositório
- `README.md` - Resumo da pasta

**Commit:** `e2dac5c` em hermes_lab

---

## PRÓXIMOS PASSOS SUGERIDOS

1. Confirmar cenário escolhido
2. Criar Google Form com campos estruturados
3. Implementar monitor Gmail via API
4. Criar parser de emails YAML
5. Configurar cronjob de sincronização

---

## CONTINUAÇÃO (2026-04-19)

Ver retrospectiva completa em: `RETROSPECTIVA_20260419.md`

**Resumo:** Proposto formulário inteligente com 3 páginas condicionais, validações e skill de triagem. Formulário anterior deletado, necessário criar manualmente.

---

## VAULTS COM .obsidian ENCONTRADOS

```
/media/peixoto/Portable/workspace/hermes_lab/         (119 .md)
/media/peixoto/Portable/secretario-agente-lke/        (117 .md)
/media/peixoto/Portable/caso-wasserman-anulatoria-adm-satelite/
```

---

## COMANDOS ÚTEIS PARA RETOMAR

```bash
# Ver clientes no SQLite
sqlite3 /media/peixoto/Portable/secretario-agente-lke/10_REFERENCIAS/secretario.db \
  "SELECT id, nome, telefone FROM clientes;"

# Ver estrutura da tabela
sqlite3 /media/peixoto/Portable/secretario-agente-lke/10_REFERENCIAS/secretario.db \
  ".schema clientes"

# Encontrar fichas de clientes
find /media/peixoto/Portable -name "*ficha_cliente*.md" 2>/dev/null
```

---

## SKILLS RELEVANTES

- `sincronizacao-obsidian-google-contacts` - já existe
- `google-workspace` - Gmail/Calendar/Contacts API
- `opencode` - delegar criação de formulários
- `here-now` - publicar páginas temporárias

---

## CONTEXTO DO USUÁRIO

- Luiz Peixoto - Advogado OAB/RJ 94719
- Prefere temas ESCUROS (sensibilidade à luz)
- Sistema LKE v5.0 / Cognição Desacoplada
- Metodologia MRSA
- Abordagem simples, passo a passo
- Verifica o que já existe antes de criar
