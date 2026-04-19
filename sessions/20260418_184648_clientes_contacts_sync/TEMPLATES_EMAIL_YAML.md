# Templates de Email YAML - Especificação Técnica

**Data:** 2026-04-18
**Versão:** 1.0

---

## 1. ESTRUTURA DO EMAIL

### Header Fields

| Campo | Formato | Exemplo |
|-------|---------|---------|
| Subject | `[TIPO] {nome} - {assunto}` | `[NOVO CLIENTE] Maria Silva - Consulta inicial` |
| From | `noreply@peixotoadvocacia.com` | Google Forms |
| To | `luizpeixoto.adv@gmail.com` | Caixa principal |
| Labels | Automático via filtro | `cliente-novo` |

---

## 2. TEMPLATE: CADASTRO NOVO CLIENTE

### 2.1 Especificação Completa

```yaml
---YAML_START---
# METADADOS
tipo_formulario: cadastro_novo_cliente
versao_template: "1.0"
data_envio: "2026-04-18T22:00:00-03:00"
form_id: "1ABC123def456"

# IDENTIFICAÇÃO
nome: "Maria da Silva Santos"
cpf: "123.456.789-00"
rg: "12.345.678-9"
data_nascimento: "1985-03-15"
nacionalidade: "Brasileira"

# CONTATO
email: "maria.silva@email.com"
telefone: "(21) 99999-8888"
telefone_alternativo: null
preferencia_contato: "WhatsApp"

# ENDEREÇO
endereco:
  cep: "22041-080"
  logradouro: "Rua do Rosário"
  numero: "123"
  complemento: "Apto 45"
  bairro: "Centro"
  cidade: "Rio de Janeiro"
  uf: "RJ"

# DADOS PROFISSIONAIS
profissao: "Contadora"
estado_civil: "Casada"

# ATENDIMENTO
tipo_atendimento: "Consulta inicial"
descricao_caso: |
  Estou em disputa com meus irmãos sobre a herança do meu pai que faleceu 
  em dezembro de 2025. O testamento não está claro sobre a divisão de um 
  imóvel em Copacabana. Gostaria de entender meus direitos.
urgencia: 4
valor_causa_estimado: null

# ORIGEM
como_conheceu: "Indicação"
indicado_por: "Dr. João Ferreira - OAB/RJ 12345"

# DISPONIBILIDADE
horarios_disponiveis:
  - "Tarde (14-18h)"
  - "Noite (19-21h)"
preferencia_reuniao: "Online"

# OBSERVAÇÕES
observacoes: "Tenho documentação completa do inventário."
---YAML_END---
```

### 2.2 Mapeamento para SQLite

| YAML Field | SQLite Column | Transformação |
|------------|---------------|---------------|
| `nome` | `nome` | Direto |
| `cpf` | `cpf` | Limpa caracteres não numéricos |
| `rg` | `rg` | Direto |
| `data_nascimento` | `data_nascimento` | ISO format |
| `nacionalidade` | `nacionalidade` | Default "Brasileira" |
| `email` | `email` | Lowercase |
| `telefone` | `telefone` | Formato (XX) XXXXX-XXXX |
| `endereco.*` | `endereco`, `cidade`, `uf`, `cep` | Concatenado + campos separados |
| `profissao` | `profissao` | Direto |
| `estado_civil` | `estado_civil` | Direto |
| `observacoes` | `observacoes` | Direto |

---

## 3. TEMPLATE: ATUALIZAÇÃO DE CADASTRO

### 3.1 Especificação

```yaml
---YAML_START---
# METADADOS
tipo_formulario: atualizacao_cadastro
versao_template: "1.0"
data_envio: "2026-04-18T22:30:00-03:00"

# IDENTIFICAÇÃO (obrigatório para busca)
cpf_identificacao: "123.456.789-00"

# DADOS A ATUALIZAR
# (campos vazios = manter atual)
nome: null  # não alterar
email: "novo.email@email.com"
telefone: "(21) 98888-7777"
endereco: null
profissao: "Contadora Chefe"
estado_civil: null

# MOTIVO
motivo_atualizacao: "Mudei de telefone e e-mail recentemente."

# ASSINATURA DIGITAL
confirmacao_veracidade: true
---YAML_END---
```

### 3.2 Lógica de Atualização

```python
# Pseudocódigo
cpf = yaml_data['cpf_identificacao']
cliente = db.query("SELECT * FROM clientes WHERE cpf = ?", cpf)

for campo, valor in yaml_data.items():
    if campo in ['cpf_identificacao', 'motivo_atualizacao']:
        continue
    if valor is not None and valor != cliente[campo]:
        db.execute(f"UPDATE clientes SET {campo} = ?, atualizado_em = NOW() WHERE cpf = ?", valor, cpf)
        log_change(cliente['id'], campo, cliente[campo], valor)
```

---

## 4. TEMPLATE: AGENDAMENTO DE REUNIÃO

```yaml
---YAML_START---
# METADADOS
tipo_formulario: agendamento_reuniao
data_envio: "2026-04-18T23:00:00-03:00"

# IDENTIFICAÇÃO
cpf_identificacao: "123.456.789-00"

# AGENDAMENTO
datas_preferenciais:
  - "2026-04-21"
  - "2026-04-22"
  - "2026-04-23"
horario_preferencial: "16:00"
forma_reuniao: "Online"

# OBSERVAÇÕES
observacoes: "Preciso que seja após às 15h."
---YAML_END---
```

---

## 5. TEMPLATE: CONFIRMAÇÃO DE DADOS

```yaml
---YAML_START---
# METADADOS
tipo_formulario: confirmacao_dados
data_envio: "2026-04-19T10:00:00-03:00"

# IDENTIFICAÇÃO
cpf_identificacao: "123.456.789-00"

# CONFIRMAÇÃO
dados_estao_corretos: true
campos_incorretos: []

# OU SE HOUVER ERRO:
# dados_estao_corretos: false
# campos_incorretos:
#   - campo: "telefone"
#     valor_atual: "(21) 99999-8888"
#     valor_correto: "(21) 99999-9999"
---YAML_END---
```

---

## 6. PARSER PYTHON

### 6.1 Código do Parser

```python
import re
import yaml
from typing import Dict, Optional, Any

class EmailYAMLParser:
    """Extrai dados estruturados YAML de emails."""
    
    PATTERN = r'---YAML_START---\s*(.*?)\s*---YAML_END---'
    
    def __init__(self, email_body: str):
        self.raw_body = email_body
        self.parsed_data: Optional[Dict[str, Any]] = None
    
    def extract_yaml(self) -> Optional[str]:
        """Extrai bloco YAML do corpo do email."""
        match = re.search(self.PATTERN, self.raw_body, re.DOTALL)
        if match:
            return match.group(1)
        return None
    
    def parse(self) -> Dict[str, Any]:
        """Parseia YAML para dicionário Python."""
        yaml_str = self.extract_yaml()
        if not yaml_str:
            raise ValueError("Nenhum bloco YAML encontrado")
        
        try:
            self.parsed_data = yaml.safe_load(yaml_str)
            return self.parsed_data
        except yaml.YAMLError as e:
            raise ValueError(f"Erro ao parsear YAML: {e}")
    
    def get_tipo_formulario(self) -> str:
        """Retorna o tipo do formulário."""
        if not self.parsed_data:
            self.parse()
        return self.parsed_data.get('tipo_formulario', 'desconhecido')
    
    def validate_required_fields(self, required: list) -> list:
        """Valida campos obrigatórios. Retorna lista de faltantes."""
        if not self.parsed_data:
            self.parse()
        missing = []
        for field in required:
            if field not in self.parsed_data or self.parsed_data[field] is None:
                missing.append(field)
        return missing


# USO:
# parser = EmailYAMLParser(email_body)
# data = parser.parse()
# tipo = parser.get_tipo_formulario()
```

### 6.2 Validações por Tipo

```python
REQUIRED_FIELDS = {
    'cadastro_novo_cliente': ['nome', 'cpf', 'email', 'telefone', 'tipo_atendimento'],
    'atualizacao_cadastro': ['cpf_identificacao', 'motivo_atualizacao'],
    'agendamento_reuniao': ['cpf_identificacao', 'datas_preferenciais'],
    'confirmacao_dados': ['cpf_identificacao', 'dados_estao_corretos'],
}

def validate_email_data(data: dict) -> tuple[bool, list]:
    """Valida dados do email."""
    tipo = data.get('tipo_formulario')
    if tipo not in REQUIRED_FIELDS:
        return False, [f"Tipo desconhecido: {tipo}"]
    
    required = REQUIRED_FIELDS[tipo]
    parser = EmailYAMLParser("")
    parser.parsed_data = data
    missing = parser.validate_required_fields(required)
    
    if missing:
        return False, [f"Campos faltantes: {', '.join(missing)}"]
    
    return True, []
```

---

## 7. FILTROS GMAIL

### 7.1 Configuração de Filtros

```json
[
  {
    "name": "Cliente Novo",
    "criteria": {
      "from": "noreply@google.com",
      "subject": "[NOVO CLIENTE]",
      "has_attachment": false
    },
    "actions": {
      "add_label": "cliente-novo",
      "mark_unread": true,
      "archive": true
    }
  },
  {
    "name": "Cliente Atualizado",
    "criteria": {
      "from": "noreply@google.com",
      "subject": "[ATUALIZAÇÃO CLIENTE]"
    },
    "actions": {
      "add_label": "cliente-atualizado",
      "mark_unread": true
    }
  },
  {
    "name": "Cliente Agenda",
    "criteria": {
      "from": "noreply@google.com",
      "subject": "[AGENDAMENTO]"
    },
    "actions": {
      "add_label": "cliente-agenda",
      "mark_unread": true
    }
  }
]
```

### 7.2 Via API Gmail

```python
from googleapiclient.discovery import build

def create_filter(service, user_id='me'):
    """Cria filtro para labels automáticas."""
    
    filter_obj = {
        'criteria': {
            'from': 'noreply@google.com',
            'subject': '[NOVO CLIENTE]'
        },
        'action': {
            'addLabelIds': ['Label_1234567890'],  # ID da label cliente-novo
            'removeLabelIds': ['INBOX']  # Arquivar
        }
    }
    
    result = service.users().settings().filters().create(
        userId=user_id, 
        body=filter_obj
    ).execute()
    
    return result
```

---

## 8. TRANSIÇÃO DE LABELS

```python
LABEL_TRANSITIONS = {
    'cliente-novo': {
        'on_success': 'cliente-validado',
        'on_error': 'cliente-erro-parse'
    },
    'cliente-validado': {
        'on_success': None,  # Aguarda ação manual
        'on_calendar': 'cliente-agenda'
    },
    'cliente-atualizado': {
        'on_success': 'cliente-sync-ok',
        'on_error': 'cliente-sync-erro'
    },
    'cliente-agenda': {
        'on_scheduled': 'cliente-reuniao',
        'on_cancelled': 'cliente-ativo'
    }
}

def transition_label(email_id: str, current_label: str, outcome: str):
    """Move email para próxima label."""
    if current_label in LABEL_TRANSITIONS:
        next_label = LABEL_TRANSITIONS[current_label].get(f'on_{outcome}')
        if next_label:
            gmail_api.modify_message(
                email_id,
                remove_labels=[current_label],
                add_labels=[next_label]
            )
```

---

## 9. CRONJOB SPEC

```yaml
# /etc/cron.d/lke-clientes-monitor

# Monitorar emails a cada 5 minutos
*/5 * * * * peixoto /usr/bin/python3 /path/to/gmail_monitor.py --label cliente-novo,cliente-atualizado

# Sincronizar Contacts a cada hora
0 * * * * peixoto /usr/bin/python3 /path/to/contacts_sync.py

# Verificar inativos diariamente
0 9 * * * peixoto /usr/bin/python3 /path/to/status_monitor.py --timeout 180
```

---

## 10. PRÓXIMA ETAPA

Com templates definidos, próximo passo é:
1. Criar Google Form com campos correspondentes
2. Configurar saída do Form para gerar YAML
3. Testar parser com emails reais

Deseja que eu avance para criar o formulário?
