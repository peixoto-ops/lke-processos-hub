# SOLUÇÃO: Detecção Automática de Handoff

**Data:** 2026-04-19 06:40

---

## RESPOSTA À PERGUNTA

**Pergunta:** O conteúdo do handoff pode ser detectado e carregado automaticamente para que a sessão já comece com ele como primeiro prompt?

**Resposta:** SIM, com ressalvas.

---

## SOLUÇÃO IMPLEMENTADA

### 1. Script de Detecção

Criado: `~/.hermes/scripts/detect_handoff.py`

```bash
# Detecta handoff mais recente
python3 ~/.hermes/scripts/detect_handoff.py
```

**Funcionalidade:**
- Procura ENUMERACAO_ATIVOS.md em todos os projetos
- Fallback para HANDOFF.md
- Retorna o mais recente com metadados
- Limita a 3000 chars para não sobrecarregar

### 2. Wrapper para Início Automático

Criado: `~/.hermes/scripts/hermes_handoff_wrapper.sh`

```bash
# Iniciar Hermes com handoff automático
~/.hermes/scripts/hermes_handoff_wrapper.sh
```

**Ou adicionar alias:**
```bash
# Adicionar ao ~/.bashrc
alias hermes='~/.hermes/scripts/hermes_handoff_wrapper.sh'
```

### 3. Via Config (prefill_messages_file)

```yaml
# Em ~/.hermes/config.yaml
prefill_messages_file: ~/.hermes/handoff_context.txt
```

**Nota:** O arquivo precisa ser atualizado antes de cada sessão.

---

## COMO FUNCIONA

### Fluxo sem Wrapper
```
Sessão fresh inicia
    ↓
Usuário diz "continuar" ou similar
    ↓
Hermes executa detect_handoff.py
    ↓
Handoff carregado como contexto
```

### Fluxo com Wrapper
```
Usuário executa hermes_handoff_wrapper.sh
    ↓
Script detecta handoff
    ↓
Hermes inicia com handoff como prefill
    ↓
Sessão começa com contexto completo
```

---

## LIMITAÇÕES

1. **prefill_messages_file é config-only** - Não é CLI flag
2. **Arquivo precisa ser atualizado** - Antes de cada sessão
3. **Wrapper é opcional** - Pode usar alias ou invocar manualmente

---

## RECOMENDAÇÃO

**Opção A (Automática):**
```bash
# Adicionar ao ~/.bashrc
alias hermes='~/.hermes/scripts/hermes_handoff_wrapper.sh'
```

**Opção B (Manual):**
```bash
# Na sessão, dizer: "continuar"
# Hermes executa detect_handoff.py via skill
```

---

## ARQUIVOS CRIADOS

```
~/.hermes/scripts/
├── detect_handoff.py         # Detecta handoff mais recente
├── hermes_handoff_wrapper.sh # Wrapper para início automático
└── session_init.py           # Já existia (NotebookLM)
```

---

## TESTE REALIZADO

```bash
$ python3 ~/.hermes/scripts/detect_handoff.py

📄 HANDOFF DETECTADO: .../ENUMERACAO_ATIVOS.md
📅 Última atualização: 2026-04-19 03:43 (2.8h atrás)
📁 Projeto: sessions
---

# ENUMERAÇÃO DE ATIVOS - Clientes Contacts Sync
...
```

✅ Funcionando!

---

## PRÓXIMA SESSÃO

Com o wrapper ou alias, o Hermes já iniciará com o handoff mais recente como contexto.

Sem wrapper, basta dizer "continuar" que a skill `hermes-session-initialization` executa a detecção.
