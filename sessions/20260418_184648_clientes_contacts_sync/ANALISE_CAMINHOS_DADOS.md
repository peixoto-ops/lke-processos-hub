# Análise de Caminhos: Fluxo de Dados de Clientes

## Contexto

- **Fonte de entrada:** Obsidian (fichas de clientes)
- **Possibilidade adicional:** Google Forms para cadastro de clientes
- **Múltiplos vaults:** Precisa centralização
- **SQLite existente:** `secretario.db` com estrutura de clientes/processos

---

## Três Caminhos Possíveis

### CAMINHO A: Obsidian como Fonte de Verdade

```
Obsidian (múltiplos vaults)
    ↓
SQLite (centralização/relacionamentos)
    ↓
Google Contacts (comunicação)
```

**Características:**
- Fichas Obsidian são o "source of truth"
- SQLite serve como índice centralizado
- Google Contacts é espelho para comunicação

**Vantagens:**
- Mantém workflow atual (você edita fichas no Obsidian)
- SQLite permite consultas relacionais
- Centraliza dados de múltiplos vaults
- Simples de entender

**Desvantagens:**
- Se editar em dois vaults, qual prevalece?
- Precisa resolver conflitos manuais
- Google Forms não se encaixa naturalmente

**Quando faz sentido:**
- Você prefere editar no Obsidian
- Não planeja usar Google Forms para cadastro
- Dados vêm principalmente das fichas

---

### CAMINHO B: SQLite como Fonte de Verdade

```
Obsidian (visualização) ← SQLite (verdade) → Google Contacts
        ↑                         ↑
    Google Forms → → → → → → → → →
```

**Características:**
- SQLite é o "source of truth"
- Obsidian é apenas visualização/exportação
- Google Forms popula diretamente o SQLite
- Google Contacts espelha SQLite

**Vantagens:**
- Centralização clara em um ponto
- Google Forms pode alimentar diretamente
- Consultas relacionais nativas
- Backup simples (um arquivo)
- Fácil de sync com Google Contacts

**Desadvantages:**
- Perde a experiência de edição no Obsidian
- Precisa recriar notas do Obsidian a partir do SQLite
- Workflow diferente do atual

**Quando faz sentido:**
- Planeja usar Google Forms extensivamente
- Clientes fazem auto-cadastro
- Quer centralização absoluta
- Múltiplos vaults precisam sincronizar

---

### CAMINHO C: Sync Bidirecional (Merkle Tree)

```
Obsidian ← → SQLite ← → Google Contacts
    ↑           ↑           ↑
    └───────────┴───────────┘
            Sync Engine
              (Merkle)
```

**Características:**
- Todos os sistemas podem ser editados
- Merkle Tree detecta mudanças
- Engine de sync resolve conflitos

**O que é Merkle Tree:**
- Estrutura de dados que cria "hash" de cada versão
- Permite detectar exatamente o que mudou
- Usado em Git, bancos distribuídos

**Vantagens:**
- Edição em qualquer lugar sincroniza
- Detecção automática de conflitos
- Sistema distribuído robusto
- Google Forms pode alimentar qualquer ponto

**Desvantagens:**
- **Complexidade ALTA**
- Over-engineering para volume atual
- Muitas coisas podem dar errado
- Manutenção pesada
- Requer timestamps consistentes

**Quando faz sentido:**
- Volume alto de dados
- Múltiplos usuários editando
- Conflitos frequentes
- Sistema já maduro

---

## Análise de Implicações

### Volume Atual

| Métrica | Valor |
|---------|-------|
| Fichas Obsidian | 33 |
| Clientes SQLite | 8 |
| Contatos Google | 118 |
| Vaults com clientes | 2 |

**Conclusão:** Volume BAIXO não justifica complexidade de sync bidirecional.

### Google Forms

**Se usar Google Forms:**

| Caminho | Integração |
|---------|-----------|
| A | Forms → Obsidian (manual) |
| B | Forms → SQLite (automático) |
| C | Forms → qualquer ponto (automático) |

**Conclusão:** Caminho B é mais natural para Forms.

### Múltiplos Vaults

**Problema:** Se o mesmo cliente está em dois vaults:

| Caminho | Solução |
|---------|---------|
| A | Vault "principal" vence |
| B | SQLite centraliza (vaults são views) |
| C | Merkle detecta e pede resolução |

**Conclusão:** Caminho B resolve naturalmente.

---

## Recomendação por Fase

### Fase 1: MVP (Simples)
```
Obsidian → SQLite → Google Contacts
```
- Obsidian é fonte de entrada
- SQLite centraliza e relaciona
- Contacts espelha para comunicação
- **SEM** sync bidirecional
- **SEM** Merkle Tree

### Fase 2: Google Forms
```
Google Forms → SQLite → Obsidian (cria nota)
                  ↓
           Google Contacts
```
- Forms popula SQLite
- Script cria nota no Obsidian a partir do SQLite
- Contacts espelha SQLite

### Fase 3: Bidirecional (SÓ SE NECESSÁRIO)
```
Implementar Merkle/conflitos
```
- **Apenas** se volume justificar
- **Apenas** se houver múltiplos editores
- **Apenas** após sistema maduro

---

## Perguntas para Decisão

1. **Você edita fichas diretamente no Obsidian?**
   - Sim → Caminho A ou C
   - Não/Pouco → Caminho B

2. **Planeja usar Google Forms para cadastro?**
   - Sim → Caminho B é mais natural
   - Não → Qualquer caminho serve

3. **Há risco de edição simultânea em vaults diferentes?**
   - Sim → Precisa de resolução de conflitos (C)
   - Não → Caminho A ou B

4. **Qual sua tolerância a complexidade técnica?**
   - Baixa → Caminho A (mais simples)
   - Média → Caminho B
   - Alta → Caminho C (overkill?)

---

## Minha Recomendação

**Para FASE 1 (agora):** Caminho A
- Mantém seu workflow atual
- Simples de implementar
- SQLite centraliza sem ser "dono"

**Para FASE 2 (quando usar Forms):** Evoluir para B
- SQLite passa a ser fonte de verdade
- Obsidian passa a ser visualização
- Forms alimenta SQLite

**NÃO RECOMENDO** começar com C (Merkle)
- Volume não justifica
- Complexidade prematura
- Over-engineering
