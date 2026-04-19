# ANÁLISE: Precisamos de Wrapper para Hermes?

**Data:** 2026-04-19 04:05
**Questão:** O Hermes precisa de wrapper como o OpenCode?

---

## DESCOBERTAS

### 1. Hermes JÁ TEM Sistema de Memória

```
Memory status
────────────────────────────────────────
 Built-in: always active
 Provider: mem0 ← active
 Plugin: installed ✓
 Status: available ✓
```

**Conclusão:** O Hermes NÃO é "fresh start" como o OpenCode.

### 2. Hermes JÁ TEM Session Resume

```bash
hermes --continue          # Retoma última sessão
hermes --resume SESSION_ID # Retoma sessão específica
hermes sessions browse     # Navegador interativo
```

### 3. Hermes JÁ TEM Session Search

O `session_search` tool busca em TODAS as sessões anteriores - não precisa de wrapper.

### 4. Hermes JÁ TEM Skills

Skills são carregadas automaticamente quando relevantes (incluindo `hermes-session-initialization`).

---

## COMPARAÇÃO

| Recurso | OpenCode | Hermes |
|---------|----------|--------|
| Memória entre sessões | ❌ PERDE | ✅ TEM (mem0 + built-in) |
| Resume de sessão | ❌ Não tem nativo | ✅ `--continue` |
| Busca em sessões | ❌ Não tem | ✅ `session_search` |
| Skills persistentes | ❌ Não tem | ✅ Sistema completo |
| Wrapper necessário | ✅ SIM | ❌ **NÃO** |

---

## CISNE NEGRO (Taleb)

**O que poderia dar errado se NÃO usarmos wrapper?**

1. **Cenário A:** Usuário esquece de dizer "continuar"
   - Risco: Baixo - memória já está lá
   - Solução: `session_search` recupera contexto

2. **Cenário B:** Contexto degradado após compaction
   - Risco: Médio - já acontece
   - Solução: ENUMERACAO_ATIVOS.md no repositório

3. **Cenário C:** Múltiplos projetos simultâneos
   - Risco: Baixo - memória sabe dos projetos
   - Solução: Dizer "continuar projeto X"

**Conclusão:** Não há cisne negro crítico. O pior caso é dizer "continuar" e eu buscar o contexto.

---

## O QUE O OPENCODE FAZ QUE HERMES NÃO FAZ

1. **Detecção automática de projeto** - OpenCode detecta qual projeto você está
2. **Sugestões proativas** - Mostra opções sem você pedir
3. **Auto-handoff no exit** - Cria handoff ao sair

**Mas isso é necessário?**

O usuário perguntou se precisamos disso. A resposta é: **depende do fluxo de trabalho**.

---

## RECOMENDAÇÃO

### Opção A: Apenas "Continuar" (SIMPLES)

- Usuário diz "continuar" ou "retomar"
- Eu busco ENUMERACAO_ATIVOS.md mais recente
- Contexto recuperado instantaneamente

**Vantagens:**
- Simples
- Sem overhead
- Funciona com o que já temos

### Opção B: Wrapper para Detecção Automática (COMPLEXO)

- Criar hook de inicialização
- Detectar projeto automaticamente
- Mostrar sugestões sem pedir

**Vantagens:**
- Mais automatizado
- Similar ao OpenCode
- Menos digitação

**Desvantagens:**
- Mais complexidade
- Pode detectar errado
- Overhead de manutenção

---

## VEREDICTO

**Para o Hermes, Opção A ("Continuar") é suficiente.**

O Hermes já tem:
- Memória persistente (mem0)
- Session resume (`--continue`)
- Session search
- Skills

O que falta (detecção automática) pode ser adicionado via:
1. Skill `hermes-session-initialization` (JÁ EXISTE)
2. ENUMERACAO_ATIVOS.md no repositório (JÁ CRIAMOS)
3. Protocolo na memória (JÁ SALVAMOS)

---

## PRÓXIMA SESSÃO

Basta dizer: **"continuar"** ou **"retomar clientes_contacts_sync"**

Eu vou automaticamente:
1. Buscar ENUMERACAO_ATIVOS.md
2. Carregar contexto
3. Executar comandos de reinício rápido se necessário

**Não precisamos de wrapper externo.**

---

## EXCEÇÃO

Se no futuro quisermos detecção automática sem dizer nada, podemos criar um hook de inicialização do Hermes (similar ao gateway hook). Mas isso é opcional, não necessário.
