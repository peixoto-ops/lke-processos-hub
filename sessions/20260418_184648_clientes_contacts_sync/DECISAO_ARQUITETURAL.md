# Decisão Arquitetural: Fluxo de Dados de Clientes

## Respostas do Usuário

1. **Edita fichas no Obsidian?** → SIM, frequentemente
2. **Usará Google Forms?** → SIM, com página here.now temporária
3. **Relação clientes-processos?** → Muitos-para-muitos (clientes com vários processos, processos com vários clientes, clientes sem processos)

---

## Decisão: Caminho Híbrido (A + B)

### Arquitetura Final

```
                    ┌─────────────────────┐
                    │   Google Forms      │
                    │ (cadastro novo)     │
                    └─────────┬───────────┘
                              │
                              ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Obsidian      │◄──►│    SQLite       │───►│ Google Contacts │
│ (fichas/edição) │    │ (centralizador) │    │ (comunicação)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                      │
        │                      │
        ▼                      ▼
   Edição manual         Relacionamentos
   Normalização          Centralização
   Documentação          Backup único
```

### Fluxo de Dados

**Entrada 1: Obsidian (edição manual)**
```
Obsidian → SQLite (importa/atualiza)
SQLite → Google Contacts (sync)
```

**Entrada 2: Google Forms (novo cliente)**
```
Google Forms → SQLite (cria cliente)
SQLite → Obsidian (gera ficha)
SQLite → Google Contacts (cria contato)
```

**Atualização:**
```
Edição no Obsidian → SQLite (detecta mudança) → Contacts
Edição no Contacts → SQLite (detecta mudança) → Obsidian (pergunta)
```

---

## Modelo de Dados: Relacionamento N:N

### Diagrama Entidade-Relacionamento

```
CLIENTES              VINCULOS              PROCESSOS
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ id           │◄────│ cliente_id   │     │ id           │
│ nome         │     │ processo_id  │────►│ numero       │
│ cpf          │     │ qualidade    │     │ tribunal     │
│ email        │     │ posicao      │     │ classe       │
│ telefone     │     │ criado_em    │     │ assunto      │
│ endereco     │     └──────────────┘     │ fase         │
│ google_res   │                          │ situacao     │
│ obsidian_path│                          └──────────────┘
│ ultimo_sync  │
└──────────────┘
```

### Qualidades Possíveis (campo `qualidade`)
- `autor` - Cliente é autor da ação
- `reu` - Cliente é réu
- `advogado` - Cliente é advogado (casos de cooperação)
- `interessado` - Cliente interessado (sem vínculo processual direto)
- `consultante` - Cliente de consulta inicial

---

## Normalização das Fichas Obsidian

### Template Proposto

```markdown
---
# IDENTIFICAÇÃO
cliente_id: (preenchido pelo sistema)
nome_completo: NOME COMPLETO
cpf: 000.000.000-00
rg: 00.000.000-0

# CONTATO
email: email@exemplo.com
telefone: 21999999999
endereco: Rua Exemplo, 123

# CLASSIFICAÇÃO
tipo: cliente|consultante|interessado
area: civil|trabalhista|familia|criminal
status: ativo|inativo|arquivado

# METADADOS
criado_em: 2026-04-18
atualizado_em: 2026-04-18
google_contact_id: people/xxxxx
sync_status: synced|pending|conflict

# VINCULOS (preenchido pelo sistema)
processos:
  - numero: 0000000-00.2025.8.19.0000
    qualidade: autor
  - numero: 0000000-00.2025.8.19.0001
    qualidade: reu
---

# Ficha do Cliente: NOME COMPLETO

## Dados Pessoais

- **Nome Completo:** NOME COMPLETO
- **CPF:** 000.000.000-00
- **RG:** 00.000.000-0
- **Nacionalidade:** Brasileira
- **Estado Civil:** Solteiro
- **Profissão:** Advogado
- **Data de Nascimento:** 01/01/1990

## Contato

- **Email:** email@exemplo.com
- **Telefone:** (21) 99999-9999
- **Endereço:** Rua Exemplo, 123, Bairro, Cidade/UF

## Observações

[Notas livres sobre o cliente]

## Histórico de Interações

- 2026-04-18: Cadastro inicial via Google Forms
```

---

## Localização das Fichas

### Estrutura Proposta

```
/media/peixoto/Portable/inv_sa_02/
└── 01 - Linha do Tempo/
    └── 01 - Clientes/
        ├── A-Z/
        │   ├── ALEXANDRE_BARATA_LYDIA.md
        │   ├── ANDRE_LUIZ_LORETO_VIVAS.md
        │   └── ...
        └── _templates/
            └── template_cliente.md
```

### Convenção de Nomes
- Arquivo: `NOMECOMPLETO.md` (sem acentos, underscores)
- Dentro do arquivo: `nome_completo` com acentos

---

## Google Forms + here.now

### Fluxo Proposto

1. Criar Google Form com campos do template
2. Publicar página here.now com:
   - Link para o formulário
   - Instruções de preenchimento
   - Prazo de validade
3. Enviar link temporário ao cliente
4. Resposta do Form → Google Sheets → Script → SQLite
5. SQLite gera:
   - Ficha Obsidian
   - Contato Google Contacts
   - Notificação para você

### Campos do Google Form

- Nome completo (obrigatório)
- CPF (obrigatório, validação)
- Email (obrigatório)
- Telefone (obrigatório)
- Endereço (opcional)
- Tipo de atendimento (dropdown)
- Descrição do caso (texto livre)

---

## Próximos Passos (Ordem de Prioridade)

### 1. Documentar Decisão Arquitetural
- [x] Criar este documento
- [ ] Validar com usuário

### 2. Normalizar Template
- [ ] Criar template padrão de ficha
- [ ] Documentar campos obrigatórios/opcionais
- [ ] Migrar fichas existentes para novo padrão

### 3. Criar Google Form
- [ ] Configurar formulário
- [ ] Conectar com Google Sheets
- [ ] Testar fluxo

### 4. Implementar Scripts
- [ ] `sync_obsidian_to_sqlite.py` - Importa fichas
- [ ] `sync_sqlite_to_contacts.py` - Atualiza Google
- [ ] `sync_forms_to_sqlite.py` - Processa Forms
- [ ] `generate_obsidian_note.py` - Cria ficha do SQLite

### 5. here.now
- [ ] Criar template de página de formulário
- [ ] Documentar processo de publicação

---

## Status: IDEAÇÃO

**Este documento é de planejamento, não de implementação.**

**Não criar scripts ainda.**
**Não migrar dados ainda.**
**Aguardar validação do usuário.**
