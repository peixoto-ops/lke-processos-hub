# Formulário Inteligente Multi-Propósito - Especificação

**Data:** 2026-04-19
**Status:** Proposta de evolução

---

## 1. PROBLEMA ATUAL

O formulário criado é simples e funcional, mas:

1. **Não diferencia** primeiro atendimento de atualização
2. **Sem validação** de campos (CPF, telefone)
3. **Sem condicionais** - todos respondem tudo
4. **Sem resposta automática** personalizada
5. **Sem triagem inteligente** por tipo

---

## 2. FORMULÁRIO INTELIGENTE PROPOSTO

### 2.1 Lógica Condicional

```
INÍCIO
    │
    ▼
[Pergunta 1: Tipo de Solicitação]
    │
    ├── Novo Cadastro ──────────────────► Página A (Dados Completos)
    │                                        │
    │                                        ▼
    │                                   [Qualificação Completa]
    │                                        │
    │                                        ▼
    │                                   [Tipo de Atendimento]
    │                                        │
    │                                        ▼
    │                                   [Resposta Personalizada]
    │
    ├── Atualização de Dados ───────────► Página B (Identificação)
    │                                        │
    │                                        ▼
    │                                   [CPF para buscar cadastro]
    │                                        │
    │                                        ▼
    │                                   [O que deseja atualizar?]
    │                                        │
    │                                        ├── Endereço
    │                                        ├── Telefone
    │                                        ├── E-mail
    │                                        └── Outro
    │
    └── Consulta/Informação ────────────► Página C (Rápido)
                                             │
                                             ▼
                                        [CPF ou Nome]
                                             │
                                             ▼
                                        [Assunto]
```

### 2.2 Páginas do Formulário

#### Página A: Novo Cadastro (Dados Completos)

| Campo | Tipo | Validação | Obrigatório |
|-------|------|-----------|-------------|
| Nome completo | Texto | Mín 3 caracteres | ✅ |
| CPF | Texto | Regex: `\d{3}\.\d{3}\.\d{3}-\d{2}` | ✅ |
| RG | Texto | - | ❌ |
| Data de nascimento | Data | - | ✅ |
| Nacionalidade | Texto | Default: "Brasileira" | ❌ |
| Estado civil | Dropdown | Solteiro/Casado/Divorciado/Viúvo | ❌ |
| Profissão | Texto | - | ❌ |
| E-mail | E-mail | Validação Google | ✅ |
| Telefone/WhatsApp | Texto | Regex: `\(\d{2}\)\s*\d{4,5}-\d{4}` | ✅ |
| CEP | Texto | 8 dígitos | ✅ |
| Logradouro | Texto | - | ✅ |
| Número | Texto | - | ✅ |
| Complemento | Texto | - | ❌ |
| Cidade | Texto | - | ✅ |
| UF | Dropdown | 27 estados | ✅ |

**Próxima página:** Tipo de Atendimento

#### Página B: Atualização de Dados

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| CPF | Texto | ✅ (para buscar cadastro) |
| O que deseja atualizar? | Checkbox | ✅ |
| - Endereço | | |
| - Telefone | | |
| - E-mail | | |
| - Estado civil | | |
| - Profissão | | |
| - Outro | | |

**Condicional:** Se "Endereço" marcado → mostrar campos de endereço

#### Página C: Consulta Rápida

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| CPF ou Nome | Texto | ✅ |
| Assunto | Dropdown | ✅ |
| - Verificar prazo | | |
| - Agendar reunião | | |
| - Enviar documentação | | |
| - Outro | | |
| Descrição | Parágrafo | ✅ |

---

## 3. TIPOS DE ATENDIMENTO E RESPOSTAS

### 3.1 Consulta Inicial

**Resposta automática:**

> Olá, {nome}!
> 
> Recebemos sua solicitação de consulta inicial sobre {tema}.
> 
> **Próximos passos:**
> 1. Nossa equipe entrará em contato em até 24h
> 2. Você receberá um link para agendar horário
> 3. Preparamos um resumo do seu caso para análise
> 
> **Urgência identificada:** {urgencia}
> 
> Atenciosamente,
> Peixoto Advocacia - OAB/RJ 94719

**Trigger para skill:**

```yaml
tipo: consulta_inicial
acoes:
  - criar_cliente_sqlite
  - gerar_ficha_obsidian
  - verificar_agenda
  - sugerir_horarios
  - enviar_audio_boas_vindas
  - notificar_advogado
```

### 3.2 Segunda Opinião

**Resposta automática:**

> Olá, {nome}!
> 
> Recebemos sua solicitação de segunda opinião.
> 
> **Para agilizar, pedimos:**
> 1. Envie a cópia do processo (PDF) para analise@peixoto.adv.br
> 2. Informe qual é a dúvida principal
> 3. Anexe a decisão/parecer que deseja revisar
> 
> **Prazo de análise:** 3-5 dias úteis
> 
> Atenciosamente,
> Peixoto Advocacia

**Trigger para skill:**

```yaml
tipo: segunda_opiniao
acoes:
  - criar_cliente_sqlite
  - vincular_processo_existente
  - criar_pasta_analise
  - solicitar_documentos
  - agendar_reuniao_analise
```

### 3.3 Análise de Contrato

**Resposta automática:**

> Olá, {nome}!
> 
> Recebemos sua solicitação de análise contratual.
> 
> **Envie o contrato para:**
> - E-mail: analise@peixoto.adv.br
> - WhatsApp: (21) 96919-1621
> 
> **Informações importantes:**
> - Prazo médio: 5-7 dias úteis
> - Valor: a combinar após análise inicial
> - Relatório completo incluído
> 
> Atenciosamente,
> Peixoto Advocacia

### 3.4 Acompanhamento Processual

**Resposta automática:**

> Olá, {nome}!
> 
> Confirmamos seu cadastro para acompanhamento processual.
> 
> **Processo vinculado:** {numero_processo}
> **Próxima movimentação:** {proxima_data}
> 
> Você receberá atualizações automáticas por WhatsApp.
> 
> Atenciosamente,
> Peixoto Advocacia

---

## 4. VALIDAÇÕES PROPOSTAS

### 4.1 CPF

```javascript
// Validação Google Forms (via Apps Script)
function validarCPF(cpf) {
    cpf = cpf.replace(/[^\d]/g, '');
    
    if (cpf.length !== 11) return false;
    
    // Verificar dígitos iguais
    if (/^(\d)\1+$/.test(cpf)) return false;
    
    // Calcular dígitos verificadores
    let soma = 0;
    for (let i = 0; i < 9; i++) {
        soma += parseInt(cpf[i]) * (10 - i);
    }
    let resto = (soma * 10) % 11;
    if (resto === 10) resto = 0;
    if (resto !== parseInt(cpf[9])) return false;
    
    soma = 0;
    for (let i = 0; i < 10; i++) {
        soma += parseInt(cpf[i]) * (11 - i);
    }
    resto = (soma * 10) % 11;
    if (resto === 10) resto = 0;
    
    return resto === parseInt(cpf[10]);
}
```

### 4.2 Telefone

```javascript
function validarTelefone(telefone) {
    // Aceita (21) 99999-9999 ou 21999999999
    const regex = /^\(?\d{2}\)?\s*\d{4,5}-?\d{4}$/;
    return regex.test(telefone);
}
```

### 4.3 CEP

```javascript
function validarCEP(cep) {
    // Busca automática de endereço via API
    const cepLimpo = cep.replace(/\D/g, '');
    if (cepLimpo.length !== 8) return false;
    
    // API ViaCEP
    const response = UrlFetchApp.fetch(`https://viacep.com.br/ws/${cepLimpo}/json/`);
    const data = JSON.parse(response.getContentText());
    
    if (data.erro) return false;
    
    // Auto-preencher cidade e UF
    return {
        valido: true,
        cidade: data.localidade,
        uf: data.uf,
        logradouro: data.logradouro
    };
}
```

---

## 5. INTEGRAÇÃO COM SKILL

### 5.1 Trigger Automático

Quando uma resposta é recebida:

```yaml
trigger: form_response_received
tipo: ${tipo_solicitacao}

fluxo:
  - ler_resposta_sheets
  - classificar_tipo
  - criar_registro_sqlite
  - gerar_ficha_obsidian
  - atualizar_google_contacts
  - verificar_agenda
  - gerar_resposta_personalizada
  - enviar_audio_cliente
  - notificar_advogado
```

### 5.2 Resposta por WhatsApp

```python
# Pseudocódigo
async def processar_form_response(response):
    tipo = response.get('tipo_solicitacao')
    cliente = response.get('nome')
    
    if tipo == 'novo_cadastro':
        # Criar cliente no SQLite
        cliente_id = sqlite.insert(response)
        
        # Gerar ficha Obsidian
        obsidian.create_ficha(cliente_id)
        
        # Verificar agenda
        horarios = calendar.get_available_slots()
        
        # Gerar áudio
        audio = tts.generate(f"Olá {cliente}, recebemos seu cadastro...")
        
        # Enviar para advogado
        telegram.send_to_advogado(audio, response)
        
    elif tipo == 'atualizacao':
        # Buscar cliente
        cliente = sqlite.find_by_cpf(response['cpf'])
        
        # Diff campos
        changes = diff(cliente, response)
        
        # Atualizar SQLite
        sqlite.update(cliente.id, changes)
        
        # Sync Google Contacts
        contacts.sync(cliente.id)
```

---

## 6. CENÁRIO: CLIENTE LIGA E VOCÊ NÃO PODE ATENDER

### Fluxo Atual

```
Cliente liga → Você não pode atender → Perde a ligação → Cliente frustrado
```

### Fluxo Proposto

```
Cliente liga
    │
    ▼
Você não pode atender
    │
    ▼
Ativa skill: "Triagem Cliente"
    │
    ▼
Skill verifica:
    - Quem é o cliente? (CPF na resposta do form)
    - Qual o tipo de atendimento?
    - Qual o processo relacionado?
    │
    ▼
Skill gera:
    - Áudio personalizado: "Olá {nome}, o Dr. Luiz está em audiência..."
    - Resumo do processo
    - Próximos passos
    - Link para agendar horário
    │
    ▼
Você encaminha áudio do WhatsApp para Telegram
    │
    ▼
Cliente ouve e é atendido automaticamente
```

### Skill de Triagem

```yaml
name: triagem-cliente-audiencia
trigger:
  - cliente ligou
  - advogado em audiência/ocupado
  - form response recebida

acoes:
  1. Identificar cliente via CPF/nome
  2. Buscar processo relacionado
  3. Gerar relatório rápido:
     - Status do processo
     - Última movimentação
     - Próximo prazo
     - Pendências
  4. Gerar áudio personalizado
  5. Enviar para advogado (Telegram)
  6. Advogado encaminha para cliente (WhatsApp)

audio_template: |
  Olá {nome}, aqui é a assistente do Dr. Luiz Peixoto.
  
  O Dr. está em uma audiência e não pode atender agora.
  
  Sobre seu processo {numero_processo}:
  - Situação atual: {status}
  - Última movimentação: {ultima_mov}
  - Próximo prazo: {proximo_prazo}
  
  O Dr. retornará sua ligação em aproximadamente {tempo_estimado}.
  
  Se preferir, você pode:
  1. Agendar um horário: {link_agenda}
  2. Enviar mensagem por aqui mesmo
  
  Obrigado pela compreensão!
```

---

## 7. INTERVALO DE TEMPO E ANÁLISES

### 7.1 Métricas a Coletar

| Métrica | Uso |
|---------|-----|
| Hora da ligação | Identificar padrões (manhã/tarde/noite) |
| Tempo até resposta | Medir SLA |
| Tipo de atendimento | Distribuir demanda |
| Urgência | Priorizar casos |
| Origem do cliente | ROI por canal |

### 7.2 Dashboard de Análises

```sql
-- Query para análise de tempo
SELECT 
    cliente_id,
    tipo_atendimento,
    hora_contato,
    hora_resposta,
    tempo_resposta_minutos,
    urgencia
FROM atendimentos
WHERE data_hora_inicio >= date('now', '-7 days')
ORDER BY tempo_resposta_minutos DESC;
```

### 7.3 Relatório Antecipado

Antes de retornar ligação:

```python
def gerar_relatorio_pre_retorno(cliente_id):
    cliente = sqlite.get_cliente(cliente_id)
    processos = sqlite.get_processos_cliente(cliente_id)
    comunicacoes = sqlite.get_comunicacoes(cliente_id, limit=5)
    prazos = sqlite.get_prazos_proximos(cliente_id)
    
    return {
        'cliente': cliente,
        'processos': processos,
        'ultimas_comunicacoes': comunicacoes,
        'prazos_proximos': prazos,
        'sugestoes': gerar_sugestoes(cliente, processos)
    }
```

---

## 8. IMPLEMENTAÇÃO

### 8.1 Fase 1: Formulário Condicional (HOJE)

- [ ] Adicionar pergunta inicial: "Tipo de Solicitação"
- [ ] Criar seções condicionais
- [ ] Adicionar validações (Apps Script)
- [ ] Testar fluxo completo

### 8.2 Fase 2: Respostas Automáticas

- [ ] Configurar Apps Script noForm
- [ ] Criar templates de resposta por tipo
- [ ] Integrar com SQLite
- [ ] Testar notificações

### 8.3 Fase 3: Skill de Triagem

- [ ] Criar skill `triagem-cliente-audiencia`
- [ ] Integrar com TTS
- [ ] Integrar com Telegram/WhatsApp
- [ ] Testar cenário real

### 8.4 Fase 4: Dashboard de Análises

- [ ] Criar queries de análise
- [ ] Dashboard Streamlit
- [ ] Relatórios automáticos

---

## 9. PRÓXIMOS PASSOS IMEDIATOS

1. **Adicionar pergunta condicional inicial** ao form existente
2. **Criar seção de atualização** com campos condicionais
3. **Adicionar validação** de CPF via Apps Script
4. **Configurar resposta automática** por tipo

Deseja que eu implemente a próxima fase do formulário agora?
