# Análise: Supabase Self-Hosted vs SQLite Local

## Fonte: Documentação Oficial Supabase

### Descobertas sobre Self-Hosted

#### Requisitos Mínimos
- Docker e Docker Compose
- Git
- ~30 minutos para setup inicial
- Conhecimento de administração Linux, Docker, redes

#### O que Inclui
- PostgreSQL (banco relacional)
- Auth Server (autenticação)
- Storage Server (arquivos)
- Realtime Server (websocket)
- Functions Server (serverless)
- Analytics Server

#### Complexidade
- **Alta** para iniciantes
- Múltiplos containers Docker
- Configuração de portas, DNS, firewalls
- Manutenção de atualizações

### Comparação: Supabase Self-Hosted vs SQLite Local

| Critério | Supabase Self-Hosted | SQLite Local |
|----------|---------------------|--------------|
| Setup | ~30 min, múltiplos containers | Segundos, 1 arquivo |
| Recursos | PostgreSQL + Auth + Storage + Realtime | Apenas banco SQL |
| Manutenção | Alta (Docker, atualizações) | Baixa (backup do arquivo) |
| Escalabilidade | Alta (pode crescer) | Baixa (apenas local) |
| Complexidade | Alta | Baixa |
| Custo hardware | Maior (múltiplos containers) | Mínimo |
| Curva aprendizado | Íngreme | Plana |

## Análise para o Caso LKE

### Situação Atual
- SQLite já existe em `secretario.db`
- Tabelas de clientes, processos, vínculos já criadas
- 8 clientes cadastrados (vs 33 fichas Obsidian)
- Apenas 1 processo cadastrado

### Recomendação: KISS Approach

**Fase 1 (MVP):** Manter SQLite local
- Já está funcionando
- Schema adequado
- Baixa complexidade
- Foco em popular dados

**Fase 2 (Se necessário):** Migrar para Supabase
- Quando volume de dados justificar
- Quando precisar de multi-usuário
- Quando precisar de API pública
- Quando precisar de autenticação

### Por que NÃO migrar agora

1. **YAGNI:** Não precisamos de Auth, Storage, Realtime ainda
2. **Complexidade:** Adiciona maintenance overhead
3. **Foco:** Energia deve ir em sincronizar dados, não em infra
4. **Risco:** Migração pode introduzir bugs
5. **Prioridade:** Dados desconectados são problema maior

## Decisão Arquitetural Proposta

```
FONTES DE DADOS:
1. Obsidian (fichas) ← fonte primária de dados de clientes
2. Google Contacts ← usado para comunicação
3. SQLite (secretario.db) ← banco relacional local

SINCRONIZAÇÃO MVP:
Obsidian → SQLite (importar fichas)
SQLite ↔ Google Contacts (sync contatos)

NÃO FAZER AGORA:
- Migração para Supabase
- Sistema complexo bidirecional
- API pública
```

## Próximo Passo Recomendado

**Simples primeiro:** Criar script que importa fichas do Obsidian para SQLite, depois sync SQLite → Google Contacts.

**Evitar:** Começar com Supabase self-hosted, over-engineering, sistema bidirecional completo desde o início.
