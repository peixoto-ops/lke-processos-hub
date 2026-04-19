# DIAGNÓSTICO - Desvio de Foco da Sessão

**Data:** 2026-04-19 09:15
**Duração da sessão:** ~8 horas

---

## 1. CONTEXTO ORIGINAL

**Repositório:** `/media/peixoto/Portable/lke-processos-hub`
**Objetivo:** Sistema de gestão de processos jurídicos com API CNJ, Fabric patterns e Obsidian

**Sessão principal:** `clientes_contacts_sync`
**Tarefa original:**
- Validar cenário Gmail+Tags para novos clientes
- Criar formulário de cadastro de clientes
- Sincronizar fichas Obsidian com Google Contacts

---

## 2. ONDE COMEÇAMOS A NOS PERDER

### Ponto de Partida (válidos):
```
0cd9f6d init(lke-processos-hub): repositorio inicial
56280d1 docs(diagnostico): relatorio completo de recursos
efff1d6 docs: diagnostico completo, instrucoes
51f8e71 docs(sessao): arquitetura completa clientes-contacts-sync
3e6075a docs(registro): registro central
cae9158 docs(handoff): referencia ao mapa
a20edb8 docs(mapeamento): analise de ativos
c324eb9 docs(formulario): proposta de formulário inteligente  ← ÚLTIMO COMMIT VÁLIDO
```

### Ponto de Divergência (desvio):
```
0ba2c2e docs(retrospectiva): sessão com antecedentes  ← COMEÇA DESVIO
695ec86 docs(enumeracao): ativos e comandos de reinicio
f971df4 docs(analise): wrapper não necessário
8f3c141 docs(handoff): solução de detecção automática
```

**Marcador:** O commit `0ba2c2e` marca onde começamos a discutir handoff automático ao invés do formulário.

---

## 3. ARQUIVOS CRIADOS DURANTE O DESVIO

### No repositório (válidos):
```
sessions/20260418_184648_clientes_contacts_sync/
├── HANDOFF.md                          ✓ (sessão anterior)
├── WORKFLOW_COORDENADO.md              ✓ (sessão anterior)
├── CENARIOS_INTELIGENTES.md            ✓ (sessão anterior)
├── TEMPLATE_FICHA_CLIENTE.md           ✓ (sessão anterior)
├── FORMULARIO_INTELIGENTE_PROPOSTA.md  ✓ (útil, manter)
├── RETROSPECTIVA_20260419.md           ? (documenta desvio)
├── ENUMERACAO_ATIVOS.md                ? (útil, mas contexto de desvio)
├── ANALISE_WRAPPER_HERMES.md           ✗ (desvio total)
└── SOLUCAO_DETECCAO_HANDOFF.md         ✗ (desvio total)
```

### Fora do repositório (problemáticos):
```
~/.hermes/scripts/
├── detect_handoff.py                   ✗ (criado sem validação)
└── hermes_handoff_wrapper.sh           ✗ (criado sem validação)

~/.hermes/skills/productivity/hermes-session-initialization/
└── SKILL.md                            ✗ (modificado sem validação)
```

---

## 4. ALTERAÇÕES REALIZADAS SEM VALIDAÇÃO

### Scripts criados:
- `detect_handoff.py` - 3789 bytes
- `hermes_handoff_wrapper.sh` - 813 bytes

### Skills modificadas:
- `hermes-session-initialization/SKILL.md` - seção adicionada

### Memória alterada:
- Entry substituída sobre protocolo de inicialização

---

## 5. ANÁLISE DE IMPACTO

### Commits relacionados ao desvio:
| Commit | Arquivos | Impacto |
|--------|----------|---------|
| 0ba2c2e | RETROSPECTIVA_20260419.md | Documentação |
| 695ec86 | ENUMERACAO_ATIVOS.md | Documentação útil |
| f971df4 | ANALISE_WRAPPER_HERMES.md | Desvio documentado |
| 8f3c141 | SOLUCAO_DETECCAO_HANDOFF.md | Desvio documentado |

### Scripts externos:
| Arquivo | Status | Risco |
|---------|--------|-------|
| detect_handoff.py | Ativo (chmod +x) | Baixo - não integrado |
| hermes_handoff_wrapper.sh | Ativo (chmod +x) | Baixo - não integrado |

### Skills:
| Skill | Modificação | Risco |
|-------|-------------|-------|
| hermes-session-initialization | Seção adicionada | Médio - pode causar confusão |

---

## 6. ALTERNATIVAS DE RECUPERAÇÃO

### Opção A: Branch de Auditoria + Revert Parcial (RECOMENDADA)

**Vantagens:**
- Preserva histórico completo
- Permite auditoria futura
- Revert limpo dos commits de desvio
- Scripts desativados sem deletar

**Passos:**
1. Criar branch `audit/desvio-handoff-automatico`
2. Revert commits `0ba2c2e` até `8f3c141` (4 commits)
3. Desativar scripts: `chmod -x` + comentários
4. Reverter modificação da skill
5. Manter documentação útil (FORMULARIO, ENUMERACAO)

### Opção B: Reset Hard + Branch Órfão

**Vantagens:**
- Mais simples
- Histórico limpo

**Desvantagens:**
- Perde rastro dos commits
- Difícil recuperar contexto futuro
- Não permite auditoria

### Opção C: Cherry-pick Commits Válidos

**Vantagens:**
- Seleção granular

**Desvantagens:**
- Complexo
- Pode causar conflitos
- Não resolve scripts externos

---

## 7. RECOMENDAÇÃO FUNDAMENTADA

**Escolha: Opção A**

**Justificativa:**
1. Preserva experiência valiosa (8h de trabalho)
2. Permite auditoria posterior
3. Scripts podem ser reativados se necessário
4. Documentação útil não é perdida
5. Segue princípio de não-destruição

**Riscos da Opção A:**
- Branch órfão pode ser esquecido
- Mitigação: Adicionar ao README do branch

---

## 8. PLANO DE EXECUÇÃO (Opção A)

### Fase 1: Preparação
```bash
# Criar branch de auditoria
git checkout -b audit/desvio-handoff-automatico

# Marcar como não-PR
git push origin audit/desvio-handoff-automatico --set-upstream
# Adicionar nota no README do branch
```

### Fase 2: Revert
```bash
# Voltar para main
git checkout main

# Revert commits de desvio (ordem reversa)
git revert --no-commit 8f3c141  # SOLUCAO_DETECCAO_HANDOFF.md
git revert --no-commit f971df4  # ANALISE_WRAPPER_HERMES.md
git revert --no-commit 695ec86  # ENUMERACAO_ATIVOS.md
git revert --no-commit 0ba2c2e  # RETROSPECTIVA_20260419.md

# Commit do revert
git commit -m "revert: desvio de handoff automático - voltar ao foco original"
```

### Fase 3: Desativar Scripts
```bash
# Desativar scripts externos
chmod -x ~/.hermes/scripts/detect_handoff.py
chmod -x ~/.hermes/scripts/hermes_handoff_wrapper.sh

# Adicionar comentários explicativos
# (inserir cabeçalho explicativo nos scripts)
```

### Fase 4: Reverter Skill
```bash
# Reverter modificação da skill
cd ~/.hermes/skills/productivity/hermes-session-initialization/
git diff SKILL.md > /tmp/skill_changes.patch
git checkout SKILL.md
```

### Fase 5: Commit Final
```bash
git add -A
git commit -m "fix: desativar scripts de handoff automático - aguardando validação"
git push origin main
```

---

## 9. PRÓXIMOS PASSOS APÓS RECUPERAÇÃO

1. Voltar ao foco original: formulário de clientes
2. Validar proposta de formulário inteligente
3. Decidir: criar manualmente ou delegar
4. Implementar APÓS validação

---

## 10. LIÇÕES APRENDIDAS

1. **Sempre validar antes de implementar** - Especialmente scripts externos
2. **Manter foco no objetivo** - Desvios devem ser anotados, não implementados
3. **KISS** - Simplificar antes de complexificar
4. **Documentar decisões** - Mesmo desvios geram aprendizado
5. **Commits semânticos ajudam** - Permitiram identificar ponto de divergência

---

## ANEXO: Lista de Arquivos para Auditoria

**Manter:**
- FORMULARIO_INTELIGENTE_PROPOSTA.md (útil para projeto)
- ENUMERACAO_ATIVOS.md (útil para continuidade)

**Mover para branch de auditoria:**
- RETROSPECTIVA_20260419.md
- ANALISE_WRAPPER_HERMES.md
- SOLUCAO_DETECCAO_HANDOFF.md
- detect_handoff.py
- hermes_handoff_wrapper.sh
