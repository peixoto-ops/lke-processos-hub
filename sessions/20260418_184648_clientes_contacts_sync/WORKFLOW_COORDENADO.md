# Workflow Coordenado: Cadastro de Cliente via WhatsApp

## Análise de Skills Necessárias

### Opção A: Uma Skill Única (Não Recomendado)

**Problema:** Criaria uma skill monolítica que faz tudo - viola princípio SRP (Single Responsibility Principle).

### Opção B: Workflow Coordenado (Recomendado)

Distribuir em etapas, cada uma com responsabilidade clara.

---

## Mapeamento de Skills por Etapa

### Etapa 1: Recebimento e Extração (Hermes)
**Skill:** `sincronizacao-obsidian-google-contacts`

**Função:**
- Receber print/texto do Telegram
- Extrair dados básicos (nome, telefone, contexto)
- Validar informações mínimas

**Ação:**
```
Você envia print → Hermes extrai:
- Nome: "Maria Silva"
- Telefone: "2199999-9999"
- Contexto: "Indicação do Dr. João"
```

---

### Etapa 2: Criação de Formulário (Delegar ao OpenCode)
**Skill:** `opencode` + `here-now`

**Tarefa para OpenCode:**
```
opencode run 'Criar Google Form para cadastro de cliente jurídico com campos:
- nome_completo (obrigatório)
- cpf (obrigatório, validação)
- email (obrigatório)
- telefone (obrigatório)
- tipo_atendimento (dropdown: consulta inicial, segunda opinião...)
- tema_discussao (textarea, 20-500 chars)
- urgencia (dropdown)
- disponibilidade (checkbox: manhã/tarde/noite)

Após criar, gerar página here.now com link do formulário.
Página deve ter instruções claras e expiração em 7 dias.
'
```

**Por que delegar ao OpenCode?**
- Tarefa de implementação (criar form, configurar)
- Requer múltiplos passos com decisões técnicas
- Não precisa de interação contínua com você

---

### Etapa 3: Monitoramento (Hermes + Cronjob)
**Skill:** `cronjob` + `sincronizacao-obsidian-google-contacts`

**Função:**
- Script roda a cada 5 minutos
- Verifica novas respostas no Google Sheets
- Quando detecta, dispara notificação

**Cronjob:**
```yaml
schedule: "*/5 * * * *"
prompt: |
  Verificar Google Sheets por novas respostas do formulário de clientes.
  Se houver nova resposta:
  1. Extrair dados
  2. Inserir no SQLite
  3. Gerar ficha Obsidian
  4. Notificar via Telegram
```

---

### Etapa 4: Verificação de Agenda (Hermes)
**Skill:** `google-workspace` (Calendar API)

**Função:**
- Consultar Calendar API
- Encontrar horários disponíveis
- Propor opções para você

**Comando:**
```bash
gws calendar events list --format json
# Filtrar próximos 7 dias
# Identificar slots livres
```

---

### Etapa 5: Pesquisa Preliminar (Hermes + MCP)
**Skill:** `context7-mcp` + `central-pesquisas-peixoto-ops`

**Função:**
- Usar tema da discussão do formulário
- Consultar documentação via Context7
- Gerar resumo para você preparar reunião

**Exemplo:**
```
tema_discussao = "Disputa de herança com irmãos"

mcp_context7_resolve-library-id: libraryName="direito-sucessorio"
mcp_context7_query-docs: query="herança irmãos partilha controversia"
```

---

### Etapa 6: Comunicação com Cliente (Hermes)
**Skill:** `sistema-comunicacao-voz-multiplos-perfis`

**Função:**
- Gerar áudio confirmando recebimento
- Enviar por WhatsApp
- Agendar reunião

---

## Fluxo Visual Completo

```
┌─────────────────────────────────────────────────────────────────┐
│ CLIENTE chama no WhatsApp                                       │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 1: Hermes (sincronizacao-obsidian-google-contacts)       │
│ - Recebe print do Telegram                                      │
│ - Extrai: nome, telefone, contexto                              │
│ - Valida informações mínimas                                    │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 2: OpenCode (opencode + here-now)                        │
│ - Cria Google Form com campos definidos                         │
│ - Gera página here.now com link                                 │
│ - Configura expiração                                           │
│ - Retorna link para você enviar ao cliente                      │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ CLIENTE preenche formulário                                     │
│ (nome, cpf, email, telefone, tipo, tema, urgência)              │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 3: Hermes Cronjob (monitoramento)                        │
│ - Verifica Sheets a cada 5 min                                  │
│ - Detecta nova resposta                                         │
│ - Insere no SQLite                                              │
│ - Gera ficha Obsidian                                           │
│ - Notifica Telegram                                             │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 4: Hermes (google-workspace)                             │
│ - Consulta Calendar API                                         │
│ - Identifica horários disponíveis                               │
│ - Propõe 3 opções para você                                     │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 5: Hermes (context7-mcp + central-pesquisas)            │
│ - Usa tema da discussão                                         │
│ - Consulta documentação jurídica                                │
│ - Gera resumo para preparação                                   │
└─────────────────────────┬───────────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ ETAPA 6: Hermes (sistema-comunicacao-voz)                      │
│ - Gera áudio de confirmação                                     │
│ - Envia por WhatsApp ao cliente                                 │
│ - Agenda reunião no Calendar                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## O Que Delegar ao OpenCode?

### DELEGAR (tarefas de implementação):
- Criar Google Form com campos específicos
- Configurar validações (CPF, campos obrigatórios)
- Gerar página here.now
- Criar script de monitoramento
- Implementar lógica de sync

### MANTER NO HERMES (tarefas de orquestração):
- Receber input do usuário
- Coordenar fluxo entre etapas
- Consultar APIs (Calendar, Contacts)
- Gerar comunicação com cliente
- Tomar decisões (horários, prioridades)

---

## Implementação em Fases

### Fase 1: Manual (Sem OpenCode)
- Hermes recebe print
- Você cria Google Form manualmente
- Hermes importa dados quando recebe

### Fase 2: Semi-Automatizada (Com OpenCode)
- OpenCode cria formulário
- Hermes monitora e importa
- Verificação de agenda manual

### Fase 3: Totalmente Automatizada
- Cronjob monitora Forms
- SQLite + Obsidian atualizados automaticamente
- Pesquisa preliminar gerada
- Horários propostos automaticamente

---

## Próximos Passos

1. **Definir prioridade:** Começar com Fase 1 (manual)?
2. **Criar Google Form piloto:** Delegar ao OpenCode?
3. **Implementar monitoramento:** Cronjob básico?
4. **Testar fluxo completo:** Com um cliente real?

---

## Resumo de Skills Envolvidas

| Etapa | Skill | Agente |
|-------|-------|--------|
| Extração | sincronizacao-obsidian-google-contacts | Hermes |
| Criação Form | opencode + here-now | OpenCode |
| Monitoramento | cronjob | Hermes |
| Agenda | google-workspace | Hermes |
| Pesquisa | context7-mcp | Hermes |
| Comunicação | sistema-comunicacao-voz | Hermes |

**Total:** 6 skills coordenadas
**Delegado ao OpenCode:** 1 etapa (criação do formulário)
**Gerenciado pelo Hermes:** 5 etapas (orquestração)
