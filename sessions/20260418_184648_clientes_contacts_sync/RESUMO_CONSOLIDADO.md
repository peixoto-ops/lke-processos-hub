# RESUMO CONSOLIDADO - Sessão Clientes Contacts Sync

**Data:** 2026-04-18 23:30
**Status:** Documentação completa, aguardando decisões

---

## DOCUMENTOS CRIADOS

| # | Arquivo | Tamanho | Conteúdo |
|---|---------|---------|----------|
| 1 | HANDOFF.md | 3KB | Ponto de partida da sessão |
| 2 | FLUXO_COMPLETO_CLIENTES.md | 15KB | Arquitetura, estados, formulários |
| 3 | TEMPLATES_EMAIL_YAML.md | 10KB | Templates YAML, parser, filtros |
| 4 | MODELO_RELACIONAL_SQLITE.md | 18KB | Schema completo, novas tabelas |
| 5 | ARQUITETURA_VAULTS_SATELITES.md | 17KB | Vaults, satélites, contaminação |
| 6 | ESTE RESUMO | - | Síntese para tomada de decisão |

**Total:** ~63KB de documentação

---

## RESUMO DO MODELO RELACIONAL

### Tabelas Existentes

| Tabela | Registros | Chave | Relacionamentos |
|--------|-----------|-------|-----------------|
| clientes | 8 | id, cpf (unique) | N:N processos, N:N vaults |
| processos | 1 | id, numero (unique) | N:N clientes, 1:N prazos |
| vinculos_cliente_processo | 1 | cliente_id + processo_id | qualidade, posicao |
| comunicacoes | ? | cliente_id, processo_id | histórico de contatos |
| prazos | ? | processo_id | datas limite |

### Tabelas Propostas

| Tabela | Finalidade | Chave |
|--------|------------|-------|
| atendimentos | Reuniões formais | cliente_id, vault_id |
| andamentos_processuais | Movimentações | processo_id, vault_id |
| vaults | Satélites Obsidian | id, nome (unique) |
| vinculos_cliente_vault | Cliente ↔ Vault | cliente_id + vault_id |

---

## CENÁRIO DE LABELS GMAIL

```
NOVO ───► VALIDADO ───► AGENDA ───► REUNIÃO ───► ATIVO
   │                        │                        │
   │                        │                        ▼
   │                        │                   [criar satélite?]
   │                        │                        │
   ▼                        ▼                        ▼
[atualiza SQLite]    [cria atendimento]    [vincula vault]
```

### Labels Definidas

| Label | Cor | Gatilho | Ação SQLite |
|-------|-----|---------|-------------|
| cliente-novo | 🔴 | Form cadastro | INSERT clientes |
| cliente-validado | 🟢 | Confirmação | UPDATE ativo=1 |
| cliente-agenda | 🔵 | Form agendamento | INSERT atendimentos |
| cliente-reuniao | 🟡 | Confirmação | UPDATE atendimentos.status |
| cliente-ativo | ⚪ | Reunião realizada | UPDATE atendimentos.status |
| cliente-atualizado | 🟠 | Form atualização | UPDATE clientes (diff) |
| cliente-inativo | ⚫ | 180d timeout | UPDATE ativo=0 |
| cliente-satelite | 🟣 | Criar vault | INSERT vaults + vinculos |

---

## FLUXO DE FORMULÁRIOS

### 3 Formulários Google Forms

1. **Cadastro Novo Cliente** (17 campos)
   - Dados pessoais, contato, endereço
   - Tipo atendimento, urgência
   - Disponibilidade, origem

2. **Atualização de Cadastro**
   - CPF para identificação
   - Campos opcionais (só os que mudaram)

3. **Agendamento de Reunião**
   - CPF + 3 datas preferenciais
   - Forma de reunião

### Saída: Email YAML

```yaml
---YAML_START---
tipo_formulario: cadastro_novo_cliente
nome: "Maria Silva"
cpf: "123.456.789-00"
email: "maria@email.com"
# ... todos os campos
---YAML_END---
```

---

## ARQUITETURA DE VAULTS

### Hierarquia

```
inv_sa_02 (MASTER)
    │
    ├── secretario-agente-lke (satelite_caso)
    │
    ├── hermes_lab (satelite_analise)
    │
    └── caso-wasserman-anulatoria-adm-satelite (satelite_caso)
```

### Critérios para Criar Satélite

| Critério | Threshold |
|----------|-----------|
| Urgência | >= 4 |
| Documentos | > 20 |
| Partes envolvidas | > 3 |
| Tipo = recurso | automático |
| Valor causa | > R$ 100.000 |

**Regra:** Se 2+ critérios atendidos → criar satélite

---

## NOTAS TÉCNICAS (DO GEMINI)

### Arquitetura Forms API

```
Google Form ──► Google Sheet ──► Apps Script ──► Hermes
                (destino)        (onFormSubmit)   (webhook)
```

### Não Usar

- Apps Script API (para código remoto)
- Forms API sozinha (sem link com tabela)

### Usar

- Forms API (criar formulário)
- Sheets API (ler/escrever planilha)
- Apps Script vinculado (gatilhos)

### Limites

- Conta @gmail.com: 100 emails/dia
- Google Workspace: 1.500 emails/dia

---

## DECISÕES NECESSÁRIAS

### 1. Implementar Tabelas Novas?

**Proposta:** Sim, todas as 4

```sql
-- Executar após aprovação
CREATE TABLE atendimentos (...);
CREATE TABLE andamentos_processuais (...);
CREATE TABLE vaults (...);
CREATE TABLE vinculos_cliente_vault (...);
```

### 2. Popular com Dados Existentes?

**Proposta:** Sim

- 4 vaults (inv_sa_02, hermes_lab, secretario-agente-lke, caso-wasserman)
- 8 clientes (já existem)
- 1 processo com vínculo

### 3. Abordagem de Implementação?

**Opção A:** Fluxo manual primeiro
- Criar formulários manualmente
- Testar parser com emails reais
- Validar conceito
- Depois automatizar

**Opção B:** Automatizar desde início
- Forms API + Apps Script
- Mais complexo, mas completo
- Maior risco de retrabalho

**Recomendação:** Opção A (validar primeiro)

### 4. Criar Satélites Automaticamente?

**Proposta:** Implementar lógica de decisão

```python
def should_create_satellite(caso_info: dict) -> bool:
    criterios = {
        'urgencia': caso_info.get('urgencia', 1) >= 4,
        'documentos': caso_info.get('documentos', 0) > 20,
        'partes': len(caso_info.get('partes', [])) > 3,
        'recurso': 'recurso' in caso_info.get('tipo', '').lower(),
        'valor': caso_info.get('valor', 0) > 100000,
    }
    return sum(criterios.values()) >= 2
```

---

## PRÓXIMOS PASSOS

### Imediato (Esta Sessão)

- [ ] Aprovar criação das 4 tabelas
- [ ] Aprovar população de dados existentes
- [ ] Definir abordagem de implementação

### Curto Prazo (Esta Semana)

- [ ] Executar migração SQLite
- [ ] Criar Google Forms (manual)
- [ ] Testar fluxo com 1 cliente real

### Médio Prazo (Próxima Semana)

- [ ] Implementar parser YAML
- [ ] Implementar monitor Gmail
- [ ] Cronjobs de sincronização

---

## ARQUIVOS DE REFERÊNCIA

```
/media/peixoto/Portable/lke-processos-hub/sessions/20260418_184648_clientes_contacts_sync/
├── HANDOFF.md
├── FLUXO_COMPLETO_CLIENTES.md
├── TEMPLATES_EMAIL_YAML.md
├── MODELO_RELACIONAL_SQLITE.md
├── ARQUITETURA_VAULTS_SATELITES.md
└── RESUMO_CONSOLIDADO.md (este)
```

---

## CONTATO

- **Luiz Peixoto** - OAB/RJ 94719
- **Email:** luizpeixoto.adv@gmail.com
- **Telefone:** +55 21 96919-1621
