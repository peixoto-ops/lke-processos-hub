# Fluxo Completo - Ciclo de Vida de Clientes LKE

**Data:** 2026-04-18
**Status:** Proposta em elaboração

---

## 1. ARQUITETURA GERAL

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SQLITE (HUB CENTRAL)                         │
│              secretario-agente-lke/10_REFERENCIAS/secretario.db     │
│                                                                      │
│  Tabela CLIENTES (19 campos):                                       │
│  id, nome, cpf, rg, nacionalidade, estado_civil, profissao,        │
│  data_nascimento, email, telefone, endereco, cidade, uf, cep,      │
│  observacoes, preferencia_contato, ativo, criado_em, atualizado_em  │
└─────────────────────────────────────────────────────────────────────┘
                              ↑↓
        ┌─────────────────────┼─────────────────────┐
        ↓                     ↓                     ↓
┌───────────────┐   ┌─────────────────┐   ┌───────────────────┐
│   Obsidian    │   │ Google Contacts │   │   Gmail + Labels  │
│    Fichas     │   │     (Cache)     │   │   (Trigger/Status)│
│  (Detalhado)  │   │   (Mobile)      │   │                   │
└───────────────┘   └─────────────────┘   └───────────────────┘
        ↑                     ↑                     ↑
        │                     │                     │
   Edição manual        Sync automático        Entrada + workflow
```

---

## 2. CICLO DE VIDA DO CLIENTE

### Estados Possíveis

| Estado | Label Gmail | Cor | Trigger | Ação Automática |
|--------|-------------|-----|---------|-----------------|
| **NOVO** | `cliente-novo` | 🔴 Vermelho | Formulário enviado | Extrair dados, criar ficha Obsidian |
| **VALIDADO** | `cliente-validado` | 🟢 Verde | Confirmação dados | Criar contato Google Contacts |
| **AGENDADO** | `cliente-agenda` | 🔵 Azul | Reunião pendente | Consultar Calendar, sugerir horários |
| **EM_REUNIAO** | `cliente-reuniao` | 🟡 Amarelo | Reunião marcada | Gerar áudio confirmação |
| **ATIVO** | `cliente-ativo` | ⚪ Cinza | Processo iniciado | Monitorar prazos, comunicações |
| **INATIVO** | `cliente-inativo` | ⚫ Preixo | 6m sem movimento | Arquivar ficha, notificar |
| **ATUALIZADO** | `cliente-atualizado` | 🟠 Laranja | Dados modificados | Atualizar SQLite + Contacts |
| **ARQUIVADO** | `cliente-arquivado` | 🔘 Cinza escuro | Caso encerrado | Mover para arquivo, manter histórico |

### Transições

```
NOVO ───────────────► VALIDADO ───────────────► AGENDADO
  │                      │                        │
  │                      │                        ▼
  │                      │                  EM_REUNIAO
  │                      │                        │
  │                      ▼                        ▼
  │                  ATIVO ◄──────────────────────┘
  │                      │
  │                      ▼
  └──────────────────► INATIVO ──────────────► ARQUIVADO
                       (6m timeout)            (manual)
```

---

## 3. FLUXOS DETALHADOS

### 3.1 Cadastro de Cliente Novo

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Google Form │───►│    Gmail    │───►│   Parser    │───►│   SQLite    │
│ (preenchido)│    │ label:novo  │    │  (extract)  │    │  (insert)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                     │
                                                                     ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Notificação│◄───│   Obsidian  │◄───│  Template   │◄───│  Trigger    │
│   (áudio)   │    │   (ficha)   │    │   (yaml)    │    │ AFTER INSERT│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

**Passos:**
1. Cliente preenche Google Form (campos: nome, cpf, email, telefone, etc.)
2. Form envia email para `luizpeixoto.adv@gmail.com` com label `cliente-novo`
3. Cronjob (5min) detecta emails com label `cliente-novo`
4. Parser extrai dados estruturados do corpo do email
5. Script insere no SQLite
6. Trigger AFTER INSERT gera ficha Obsidian
7. Notificação enviada (TTS/Telegram)

### 3.2 Atualização de Cliente Existente

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Google Form │───►│    Gmail    │───►│   Parser    │
│  (edição)   │    │label:atual  │    │  (extract)  │
└─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             ▼
                    ┌─────────────┐    ┌─────────────┐
                    │   SQLite    │───►│   Diff      │
                    │  (select)   │    │  (compare)  │
                    └─────────────┘    └─────────────┘
                                             │
                                             ▼
                    ┌─────────────┐    ┌─────────────┐
                    │  Contacts   │◄───│   Sync      │
                    │  (update)   │    │  (changes)  │
                    └─────────────┘    └─────────────┘
```

**Passos:**
1. Cliente preenche Form de atualização (com CPF para identificação)
2. Email recebe label `cliente-atualizado`
3. Parser extrai dados e CPF
4. Script busca cliente por CPF no SQLite
5. Compara campos antigos vs novos
6. Atualiza SQLite com campos modificados
7. Sincroniza mudanças com Google Contacts

---

## 4. MODELOS DE FORMULÁRIOS

### 4.1 Formulário de Cadastro - Novo Cliente

**Google Forms - Título:** "Cadastro de Cliente - Peixoto Advocacia"

| Seção | Campo | Tipo | Obrigatório | Validação |
|-------|-------|------|-------------|-----------|
| **Dados Pessoais** | | | | |
| | Nome completo | Texto curto | ✅ | Mín 3 caracteres |
| | CPF | Texto curto | ✅ | Regex: `\d{3}\.\d{3}\.\d{3}-\d{2}` |
| | RG | Texto curto | ❌ | - |
| | Data de nascimento | Data | ✅ | - |
| | Nacionalidade | Texto curto | ❌ | Default: "Brasileira" |
| **Contato** | | | | |
| | E-mail | E-mail | ✅ | Validação Google |
| | Telefone/WhatsApp | Texto curto | ✅ | Regex: `\(\d{2}\)\s*\d{4,5}-\d{4}` |
| | Preferência de contato | Múltipla escolha | ❌ | WhatsApp / Telefone / E-mail |
| **Endereço** | | | | |
| | CEP | Texto curto | ✅ | 8 dígitos |
| | Logradouro | Texto curto | ✅ | - |
| | Número | Texto curto | ✅ | - |
| | Complemento | Texto curto | ❌ | - |
| | Cidade | Texto curto | ✅ | - |
| | UF | Dropdown | ✅ | Lista de 27 estados |
| **Profissionais** | | | | |
| | Profissão | Texto curto | ❌ | - |
| | Estado civil | Dropdown | ❌ | Solteiro(a) / Casado(a) / Divorciado(a) / Viúvo(a) |
| **Atendimento** | | | | |
| | Tipo de atendimento | Dropdown | ✅ | Consulta inicial / Segunda opinião / Análise de contrato / Acompanhamento processual |
| | Descrição do caso | Parágrafo | ✅ | 20-2000 caracteres |
| | Urgência | Escala linear | ❌ | 1 (baixa) a 5 (urgente) |
| | Como conheceu | Dropdown | ❌ | Indicação / Google / Redes sociais / Outro |
| | Se indicação, quem indicou? | Texto curto | ❌ | Condicional |
| **Disponibilidade** | | | | |
| | Horários disponíveis | Caixas de seleção | ❌ | Manhã (8-12) / Tarde (14-18) / Noite (19-21) |
| | Observações adicionais | Parágrafo | ❌ | - |

**Configurações do Form:**
- Coletar e-mails: ✅
- Limitar a 1 resposta: ✅
- Permitir edição: ✅
- Notificação por e-mail: `luizpeixoto.adv@gmail.com`
- Tag automática: `cliente-novo`

---

### 4.2 Formulário de Atualização - Cliente Existente

**Google Forms - Título:** "Atualização de Cadastro - Peixoto Advocacia"

| Campo | Tipo | Obrigatório | Observação |
|-------|------|-------------|------------|
| CPF para identificação | Texto curto | ✅ | usado para buscar cadastro |
| Nome completo | Texto curto | ❌ | se vazio, mantém atual |
| E-mail | E-mail | ❌ | se vazio, mantém atual |
| Telefone/WhatsApp | Texto curto | ❌ | se vazio, mantém atual |
| Endereço completo | Parágrafo | ❌ | se vazio, mantém atual |
| Profissão | Texto curto | ❌ | se vazio, mantém atual |
| Estado civil | Dropdown | ❌ | se vazio, mantém atual |
| O que mudou? | Parágrafo | ✅ | descrição livre da mudança |

**Configurações:**
- Tag automática: `cliente-atualizado`
- Nota: "Preencha apenas os campos que deseja atualizar"

---

### 4.3 Formulário de Agendamento

**Google Forms - Título:** "Agendamento de Reunião - Peixoto Advocacia"

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| CPF | Texto curto | ✅ |
| Datas preferenciais (3 opções) | Data | ✅ (3x) |
| Horários preferenciais | Hora | ✅ |
| Forma de reunião | Dropdown | Presencial / Online / Telefone |
| Observações | Parágrafo | ❌ |

**Configurações:**
- Tag automática: `cliente-agenda`
- Integração: Google Calendar API

---

## 5. EMAIL TEMPLATE (gerado pelo Form)

### 5.1 Formato do Email de Cadastro

```
Subject: [NOVO CLIENTE] {nome} - {tipo_atendimento}

---YAML_START---
tipo_formulario: cadastro_novo
data_envio: {timestamp}

# Dados Pessoais
nome: {nome_completo}
cpf: {cpf}
rg: {rg}
data_nascimento: {data_nascimento}
nacionalidade: {nacionalidade}

# Contato
email: {email}
telefone: {telefone}
preferencia_contato: {preferencia}

# Endereço
cep: {cep}
logradouro: {logradouro}
numero: {numero}
complemento: {complemento}
cidade: {cidade}
uf: {uf}

# Profissionais
profissao: {profissao}
estado_civil: {estado_civil}

# Atendimento
tipo_atendimento: {tipo}
descricao_caso: |
  {descricao}
urgencia: {urgencia}
como_conheceu: {origem}
indicacao: {quem_indicou}

# Disponibilidade
horarios_disponiveis: {lista_horarios}
observacoes: {obs}
---YAML_END---
```

### 5.2 Formato do Email de Atualização

```
Subject: [ATUALIZAÇÃO CLIENTE] CPF: {cpf}

---YAML_START---
tipo_formulario: atualizacao_cadastro
data_envio: {timestamp}
cpf_identificacao: {cpf}

# Dados novos (preenchidos = alterar, vazios = manter)
nome: {nome}
email: {email}
telefone: {telefone}
endereco: {endereco_completo}
profissao: {profissao}
estado_civil: {estado_civil}

# Metadados
motivo_atualizacao: |
  {o_que_mudou}
---YAML_END---
```

---

## 6. TEMPLATE FICHA OBSIDIAN

### 6.1 Estrutura YAML Frontmatter

```yaml
---
tipo: cliente
id: {sqlite_id}
cpf: "{cpf}"
status: novo
ativo: true

# Dados pessoais
nome: "{nome}"
rg: "{rg}"
data_nascimento: {data_nascimento}
nacionalidade: "{nacionalidade}"
estado_civil: "{estado_civil}"
profissao: "{profissao}"

# Contato
email: "{email}"
telefone: "{telefone}"
preferencia_contato: "{preferencia}"

# Endereço
endereco:
  logradouro: "{logradouro}"
  numero: "{numero}"
  complemento: "{complemento}"
  cidade: "{cidade}"
  uf: "{uf}"
  cep: "{cep}"

# Metadados
criado_em: {timestamp}
atualizado_em: {timestamp}
primeiro_atendimento: {data}
tipo_atendimento: "{tipo}"
como_conheceu: "{origem}"
indicado_por: "{indicacao}"
urgencia: {nivel}

# Sistema
tags: [cliente, {status}]
sync_google_contacts: false
sync_status: pendente
---
```

### 6.2 Corpo da Nota

```markdown
# {nome}

> [!info] Dados do Cliente
> - **CPF:** {cpf}
> - **Telefone:** {telefone}
> - **E-mail:** {email}
> - **Status:** #cliente/{status}

## Histórico de Atendimento

### Primeira Consulta ({data})

**Tipo:** {tipo_atendimento}
**Urgência:** {urgencia}/5

**Descrição do caso:**
{descricao_caso}

**Disponibilidade:** {horarios}

---

## Processos Relacionados

> [!note] Processos
> - [[]]  <!-- link para processos -->

---

## Comunicações

> [!tip] Últimas comunicações
> - <!-- registrações de contato -->

---

## Observações

<!-- notas internas -->

---

## Histórico de Atualizações

| Data | Campo | Valor Anterior | Valor Novo |
|------|-------|----------------|------------|
```

---

## 7. SCRIPTS NECESSÁRIOS

### 7.1 Lista de Scripts

| Script | Função | Frequência |
|--------|--------|------------|
| `gmail_monitor.py` | Monitora labels e processa emails | 5 minutos (cron) |
| `email_parser.py` | Extrai YAML do corpo do email | Sob demanda |
| `sqlite_insert.py` | Insere/atualiza cliente no banco | Sob demanda |
| `obsidian_generator.py` | Gera ficha a partir do SQLite | Trigger AFTER INSERT |
| `contacts_sync.py` | Sincroniza SQLite → Google Contacts | 1 hora (cron) |
| `status_monitor.py` | Verifica clientes inativos | Diário (cron) |

### 7.2 Dependências

```
google-api-python-client
google-auth-oauthlib
sqlite3 (built-in)
pyyaml
jinja2 (templates)
python-dateutil
```

---

## 8. PRÓXIMOS PASSOS

### Fase 1: Infraestrutura (Hoje)
- [ ] Criar Google Form de cadastro
- [ ] Configurar labels no Gmail
- [ ] Testar fluxo manual

### Fase 2: Automação (Esta semana)
- [ ] Implementar `gmail_monitor.py`
- [ ] Implementar `email_parser.py`
- [ ] Criar trigger AFTER INSERT

### Fase 3: Integração (Próxima semana)
- [ ] Sync com Google Contacts
- [ ] Cronjobs de monitoramento
- [ ] Dashboard de status

### Fase 4: Refinamento
- [ ] Formulário de atualização
- [ ] Formulário de agendamento
- [ ] Templates de notificação

---

## 9. DECISÕES PENDENTES

1. **Labels:** Usar cores padrão ou customizar?
2. **Formulário:** Delegar criação ao OpenCode ou fazer manual?
3. **Prioridade:** Começar por cadastro novo ou sync Contacts?
4. **Notificações:** TTS, Telegram ou ambos?
