# Cenários Inteligentes para Cadastro de Clientes

## Arquitetura Confirmada

```
┌─────────────────────────────────────────────────────────────────┐
│                     SQLITE = HUB CENTRAL                        │
│  (secretario-agente-lke/10_REFERENCIAS/secretario.db)          │
│                                                                  │
│  Tabelas: clientes (8), processos (1), comunicacoes, skills    │
└─────────────────────────────────────────────────────────────────┘
                              ↑↓
    ┌─────────────────────────┼─────────────────────────┐
    ↓                         ↓                         ↓
┌───────────┐          ┌───────────┐          ┌───────────────────┐
│  Obsidian │          │   Google  │          │ Gmail + Auto-tag  │
│  Fichas   │          │  Contacts │          │   (Trigger)       │
└───────────┘          └───────────┘          └───────────────────┘
```

---

## Cenários Inteligentes

### CENÁRIO 1: Formulário → Gmail Tag → SQLite (RECOMENDADO)

**Fluxo:**
1. Cliente preenche Google Form
2. Form envia email para: `luizpeixoto.adv@gmail.com`
3. Email tem tag automática: `LABEL: cliente-novo`
4. Gmail API detecta emails com essa tag
5. Script extrai dados do email
6. Insere no SQLite
7. Gera ficha Obsidian
8. Remove tag (ou move para `cliente-processado`)

**Vantagens:**
- Zero dependência de Google Sheets
- Gmail como fila de entrada
- Tags funcionam como status
- Histórico completo no email
- Fácil auditoria

**Implementação:**
```python
# Gmail API filter
filter = "label:cliente-novo is:unread"

# Workflow:
# 1. Ler emails com label
# 2. Parsear dados estruturados
# 3. Inserir SQLite
# 4. Mover para label:cliente-processado
```

---

### CENÁRIO 2: WhatsApp → Print → Hermes → Gmail Tag

**Fluxo:**
1. Cliente manda print no WhatsApp
2. Você encaminha para seu email
3. Adiciona tag manual: `cliente-novo`
4. Hermes monitora tag
5. Extrai dados do print (OCR)
6. Preenche formulário automaticamente
7. Cliente recebe link para confirmar

**Vantagens:**
- Captura dados de qualquer fonte
- OCR como fallback
- Validação pelo cliente

---

### CENÁRIO 3: Gmail Labels como Máquina de Estados

**Labels propostas:**

| Label | Significado | Ação Automática |
|-------|-------------|-----------------|
| `cliente-novo` | Formulário recebido | Extrair dados, criar ficha |
| `cliente-validado` | Dados confirmados | Criar contato Google |
| `cliente-agenda` | Agendamento pendente | Consultar Calendar |
| `cliente-reuniao` | Reunião marcada | Gerar áudio confirmação |
| `cliente-ativo` | Cliente ativo | Monitorar processos |
| `cliente-inativo` | Sem movimento 6m+ | Arquivar ficha |

**Script de monitoramento:**
```python
# cronjob a cada 5 minutos
labels_monitor = [
    "cliente-novo",
    "cliente-validado",
    "cliente-agenda"
]

for label in labels_monitor:
    emails = gmail_client.list_messages(label)
    for email in emails:
        process(email, label)
```

---

### CENÁRIO 4: Google Contacts como Cache

**Filosofia:**
- SQLite = fonte de verdade
- Google Contacts = cache para mobile
- Obsidian = documentação detalhada

**Sync:**
```
SQLite (mestre)
    ↓
Google Contacts (espelho)
    ↓
Mobile (visualização)
```

**Campos sincronizados:**
- Nome
- Telefone (mobile)
- Email (work)
- Notas = link para ficha Obsidian

---

### CENÁRIO 5: Formulário Inteligente com Validação

**Campos do Google Form:**

| Campo | Tipo | Validação |
|-------|------|-----------|
| Nome completo | Texto | Obrigatório |
| CPF | Texto | Regex: `\d{3}\.\d{3}\.\d{3}-\d{2}` |
| Email | Email | Validação Google |
| Telefone | Texto | Regex: `\(\d{2}\) \d{5}-\d{4}` |
| Tipo atendimento | Dropdown | - Consulta inicial<br>- Segunda opinião<br>- Análise de contrato<br>- Acompanhamento |
| Tema discussão | Paragraph | 20-500 caracteres |
| Urgência | Escala | 1-5 |
| Como conheceu | Dropdown | - Indicação<br>- Google<br>- Redes sociais<br>- Outro |
| Disponibilidade | Checkbox | - Manhã<br>- Tarde<br>- Noite |

**Email gerado contém:**
```
Subject: [NOVO CLIENTE] {nome} - {tipo_atendimento}

Body estruturado com YAML:
---
nome: Maria Silva
cpf: 123.456.789-00
email: maria@email.com
telefone: (21) 99999-9999
tipo: Consulta inicial
tema: Disputa de herança com irmãos
urgencia: 4
disponibilidade: [tarde, noite]
indicacao: Dr. João
---
```

---

### CENÁRIO 6: Delegação ao OpenCode

**Tarefas delegáveis:**

1. **Criar Google Form:**
   ```
   opencode run 'Criar Google Form para cadastro de clientes jurídicos
   com campos definidos em CENARIO_5. Configurar resposta por email
   para luizpeixoto.adv@gmail.com com tag cliente-novo.'
   ```

2. **Criar script de monitoramento:**
   ```
   opencode run 'Criar script Python que monitora Gmail API
   por labels cliente-* e processa emails.'
   ```

3. **Implementar sync SQLite↔Contacts:**
   ```
   opencode run 'Implementar sync bidirecional entre
   tabela clientes do SQLite e Google Contacts API.'
   ```

**O que NÃO delegar:**
- Decisões de negócio
- Validação de dados sensíveis
- Comunicação com clientes
- Orquestração do fluxo

---

## Estrutura de Arquivos da Sessão

```
/media/peixoto/Portable/lke-processos-hub/
└── sessions/
    └── 20260418_184648_clientes_contacts_sync/
        ├── WORKFLOW_COORDENADO.md      # Este arquivo
        ├── CENARIOS_INTELIGENTES.md    # Cenários
        ├── MODELOS_FORMULARIOS.md      # Templates
        ├── scripts/
        │   ├── gmail_monitor.py        # Monitor Gmail
        │   ├── sync_contacts.py        # Sync Contacts
        │   └── parse_email.py          # Parser YAML
        └── diarios/
            └── 20260418_NOTAS.md       # Notas do dia
```

---

## Próximos Passos

### Imediato (Hoje)
1. [ ] Criar pasta sessão
2. [ ] Definir qual cenário implementar primeiro
3. [ ] Delegar criação do Google Form ao OpenCode?

### Curto Prazo (Esta semana)
1. [ ] Implementar Gmail monitor com labels
2. [ ] Criar parser de emails estruturados
3. [ ] Testar fluxo completo

### Médio Prazo (Próximas 2 semanas)
1. [ ] Sync SQLite ↔ Google Contacts
2. [ ] Cronjob de monitoramento
3. [ ] Dashboard de status

---

## Comparação: Gmail vs Sheets

| Aspecto | Gmail + Tags | Google Sheets |
|---------|--------------|---------------|
| Latência | Instantâneo | ~5 min (polling) |
| Auditoria | Completa (email permanece) | Parcial |
| Status | Labels naturais | Campo booleano |
| Mobile | App Gmail nativo | Sheets app |
| Busca | Full-text + labels | Limitada |
| Triagem | Manual ou automática | Automática |
| Histórico | Preservado | Sobrescrito |

**Recomendação:** Gmail + Tags para entrada, SQLite para persistência.

---

## Validação Necessária

Antes de prosseguir, preciso que você confirme:

1. **Cenário preferido:** Gmail com tags (Cenário 1)?
2. **Delegação:** Usar OpenCode para criar formulário?
3. **Prioridade:** Começar com fluxo manual ou automatizado?
4. **Google Form:** Criar novo ou usar template existente?
