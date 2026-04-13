# DIAGNOSTICO INICIAL - LKE Processos Hub
## Sessao T0.1 | 13 de abril de 2026

---

## 1. SUMARIO EXECUTIVO

Este diagnostico estabelece a base para um sistema de gestao de processos juridicos integrado. O projeto **lke-processos-hub** representa a evolucao do sistema LKE (Legal Knowledge Engineering) versao 5.0, consolidando infraestrutura dispersa em uma arquitetura coesa.

**Repo criado:** https://github.com/peixoto-ops/lke-processos-hub  
**Kanban:** https://github.com/orgs/peixoto-ops/projects/18/views/1

---

## 2. RECURSOS EXISTENTES

### 2.1 Vault Principal (inv_sa_02)

**Localizacao:** `/media/peixoto/Portable/inv_sa_02/`

| Metrica | Valor |
|---------|-------|
| Pastas de processos | 44 |
| Perfis de clientes | 39 |
| Relatorios gerados | 28 |
| Estrutura | Johnny.Decimal |

**Estrutura atual:**
```
inv_sa_02/
├── 01 - Linha do Tempo/
│   └── 03 - Processos/
│       └── [44 pastas de casos]
├── 02 - Calendario/
├── 03 - Cadastro/
│   └── [fichas de clientes]
└── 04 - Relatorios/
    └── [relatorios gerados]
```

### 2.2 Pipeline de Automacao

**Script principal:** `~/bin/automate_reports`  
**Fonte:** `/media/peixoto/Portable/lke-skills-repo/10-19_Infraestrutura_Core/infra/bin/04-workflow/automate_reports`

**Repositorio de apoio:** `p31x070/comunica_dje`
- API CNJ para comunicacoes processuais
- PostgreSQL para cache
- Notificacoes Signal

### 2.3 Patterns Fabric

Integracao com Fabric patterns para:
- Extracao de metadados
- Formatacao de relatorios
- Classificacao CDD

---

## 3. ADENDOS CRITICOS (CONVERSA SOBRE VAULT TEMPLATE)

### 3.1 Tipologia de Vaults LKE

A conversa documentada em `conversa_sobre_vault_template.md` (69KB) estabelece uma **arquitetura de tres niveis**:

| Tipo | Funcao | Exemplo |
|------|--------|---------|
| **Cofre Master** | Controle e orquestracao | inv_sa_02 |
| **Cofres Satelites** | Casos/projetos especificos | ekwrio, case-dianne-nicola |
| **Cofre de Pesquisa** | Zettelkasten/base de conhecimento | patterns-juridicos |

### 3.2 Estrutura Taxonomica Proposta

Apos extensa discussao, foi validada a seguinte estrutura para **Cofres Satelites**:

```
NOME_DO_CASO/
├── .obsidian/          # Configuracoes nativas (raiz)
├── .opencode/          # Config OpenCode (raiz)
├── _bmad/              # Buffer de processamento
├── _bmad-output/       # Saida de processamento
├── 00_SYSTEM/
│   ├── copilot/        # Motor IA: prompts, memory
│   ├── templates/      # Modelos Templater/Zotero
│   └── scripts/        # Automacoes locais
├── 10_ASSETS/
│   ├── 11_inbox/       # Chegada bruta
│   ├── 12_docs_cliente/# Documentos higienizados
│   ├── 13_autos/       # Pecas baixadas
│   └── 19_base/        # Tabela de organizacao
├── 20_ANALYSIS/
│   ├── 21_analise_documental/
│   ├── 22_mrsa_teses/  # Matrizes MRSA
│   ├── 23_pesquisa_cdd/# Notas Zotero
│   └── 24_copilot_chats/
├── 30_DRAFTING/
│   ├── 31_propostas/
│   └── 32_peticoes/
├── 40_OUTPUT/
│   ├── 41_protocolos/  # PDFs finais
│   └── 42_comunicacao/
└── 90_META/
    ├── index_caso.md   # Frontmatter unificado
    ├── cronograma.md
    └── logs/
```

### 3.3 Modelo de Informacao Pontozero

**Arquivo de contexto:** `modelo_info_pontozero.yaml`

YAML padrao para ingestao automatica de novos clientes:

```yaml
---
id_projeto: "__ID_PROJETO__"
data_ingestao: "__DATA_HORA__"
status_pipeline: "00_triagem_inicial"
tipo_demanda: "indefinido"

cliente:
  nome_completo: null
  nome_whatsapp: "Extraido do contato"
  telefone: "+55 XX XXXXX-XXXX"
  documento_cpf_cnpj: null
  origem_contato: "Indicacao - Dr. X"

contexto_inicial:
  assunto_bruto: "Descricao inicial"
  ramo_direito_presumido: null
  cdd_sugerido: null
  urgencia: "media"

automacao_flags:
  informacoes_pendentes:
    - "Solicitar nome completo e CPF"
  cofre_gerado: true
---
```

**Nota sobre sintaxe:** Usa `__VARIAVEL__` (duplo underscore) para evitar conflito com Fabric.

---

## 4. GAPS IDENTIFICADOS

### 4.1 Vinculo Cliente-Processo

**Problema:** Relatorios sao gerados em diretorio separado, sem vinculo automatico com fichas de clientes.

**Impacto:** Busca manual, risco de desatualizacao.

### 4.2 Indice Centralizado

**Problema:** Nao existe um indice dinamico de todos os processos ativos.

**Impacto:** Dificuldade em visualizar carteira de clientes.

### 4.3 Metadados Estruturados

**Problema:** Fichas de clientes usam formato livre, sem frontmatter padronizado.

**Impacto:** Automacoes nao conseguem extrair dados de forma confiavel.

### 4.4 Normalizacao de Vaults

**Problema:** Scripts legados em `~/bin` com nomenclatura inconsistente.

**Impacto:** Poluicao do autocomplete, risco de shadowing.

**Lista de scripts identificados:**
- `lke-ingest` / `lke-ingestor` (redundancia)
- `lke_template_engine` / `lke_template_engine_clean` / `lke_template_engine_simple` (triplicacao)
- `lke_universe_ontology.json` (arquivo de dados com +x)

---

## 5. ARQUITETURA PROPOSTA

### 5.1 Visao Geral

```
┌─────────────────────────────────────────────────────────────┐
│                    LKE PROCESSOS HUB                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  API CNJ    │───►│  Pipeline   │───►│   Vault     │     │
│  │ (comunica_  │    │ (lke-ingest)│    │  Master     │     │
│  │     dje)    │    └─────────────┘    └──────┬──────┘     │
│  └─────────────┘                              │             │
│                                               ▼             │
│                              ┌──────────────────────────┐   │
│                              │   COFRES SATELITES       │   │
│                              │  (um por caso/cliente)   │   │
│                              └──────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Componentes

#### Modulo 1: Indexacao
- Scanner do vault master
- Geracao de indice dinamico
- Integracao com Dashboard

#### Modulo 2: Vinculo Cliente-Processo
- Cross-reference automatico
- Atualizacao de metadados
- Alertas de desatualizacao

#### Modulo 3: Bridge comunica_dje
- Sincronizacao com API CNJ
- Cache local PostgreSQL
- Normalizacao de dados

#### Modulo 4: Fabric Enhanced
- Patterns com YAML frontmatter
- Tags CDD automaticas
- Pipeline de ingestao

---

## 6. MIGRACAO BROWNFIELD → GREENFIELD

### 6.1 Conceito

O projeto representa uma **migracao de infraestrutura existente (brownfield)** para uma **arquitetura nova (greenfield)**:

| Aspecto | Brownfield (atual) | Greenfield (proposto) |
|---------|-------------------|----------------------|
| Estrutura | Dispersa, multiplas | Unificada, tipada |
| Metadados | Texto livre | YAML frontmatter |
| Vinculos | Manual | Automatico |
| Scripts | Ad-hoc | CLI padronizado |

### 6.2 Estrategia de Migracao

**Fase 1:** Criar estrutura greenfield (este repositorio)  
**Fase 2:** Migrar dados gradualmente  
**Fase 3:** Validar integridade  
**Fase 4:** Desativar brownfield

---

## 7. INTEGRACOES EXISTENTES

### 7.1 Ecosystem Dashboard

- Health scoring de casos
- Integracao com Google Workspace
- OAuth2 configurado

### 7.2 Google Workspace

- Calendar para prazos
- Drive para documentos
- Sheets para metricas

### 7.3 CDD (Classificacao Decimal do Direito)

- Organizacao tematica
- Correlacoes entre microssistemas
- Hermeneutica juridica

---

## 8. DEPENDENCIAS E VALIDACOES

### 8.1 Dependencias

| Dependencia | Status | Notas |
|-------------|--------|-------|
| Python 3.10+ | OK | Sistema |
| yq (YAML processor) | Pendente | Validacao |
| Obsidian CLI | Pendente | Handoff |
| gh (GitHub CLI) | OK | Provisionamento |
| Fabric | OK | Patterns |
| xclip | OK | Clipboard |

### 8.2 Validacoes Necessarias

1. **lke-ingest.sh** - Script criado mas nao testado
2. **lke_vault_builder.py** - Referenciado mas nao implementado
3. **Obsidian CLI** - Comando `obsidian open` requer validacao
4. **Telemetria** - Funcao de log proposta mas nao implementada

---

## 9. ALTERNATIVAS CONSIDERADAS

### 9.1 Alternativa A: Manter Status Quo

**Vantagens:**
- Sem custo de migracao
- Processos ja funcionando

**Desvantagens:**
- Debito tecnico crescente
- Escalabilidade limitada

### 9.2 Alternativa B: Migracao Parcial

**Vantagens:**
- Menor risco
- Aprendizado incremental

**Desvantagens:**
- Coexistencia de sistemas
- Complexidade de mantencoes

### 9.3 Alternativa C: Migracao Completa (Recomendada)

**Vantagens:**
- Arquitetura limpa
- Automacoes robustas
- Escalabilidade

**Desvantagens:**
- Investimento inicial
- Curva de aprendizado

---

## 10. PROXIMOS PASSOS

### Sessao T1.1 (Sprint 1)

**Horario:** 09:00 - 09:50 (13/abr)

**Tarefas:**
1. Testar `lke-ingest` com dados reais
2. Validar provisionamento GitHub
3. Criar `~/.lke-legacy` para scripts antigos
4. Implementar telemetria basica

### Sessoes Futuras

| Sessao | Foco | Entregavel |
|--------|------|------------|
| T1.2 | Modulo de Indexacao | Script funcional |
| T1.3 | Vinculo Cliente-Processo | Prototipo |
| T1.4 | Bridge comunica_dje | Integracao |
| T1.5 | Fabric Enhanced | Patterns |

---

## 11. INSTRUCOES PARA PROXIMA SESSAO

### Inicio Seguro

```bash
# 1. Navegar para o repositorio
cd /media/peixoto/Portable/lke-processos-hub

# 2. Verificar estado atual
git status

# 3. Consultar kanban
# https://github.com/orgs/peixoto-ops/projects/18/views/1

# 4. Iniciar sessao T1.1
# Implementar: scripts/index_processos.py --dry-run
```

### Pre-requisitos

- [ ] Vault acessivel em `/media/peixoto/Portable/inv_sa_02/`
- [ ] Python 3.10+
- [ ] `pip install pyyaml`
- [ ] `yq` instalado (`sudo snap install yq`)
- [ ] Token GitHub valido

---

## 12. REFERENCIAS

### Documentos Relacionados

- `conversa_sobre_vault_template.md` - Discussao completa da arquitetura
- `SESSAO_PROXIMA.md` - Instrucoes detalhadas
- `config/vault_paths.yaml` - Configuracao de caminhos

### Repositorios

- `peixoto-ops/lke-processos-hub` - Este repositorio
- `p31x070/comunica_dje` - API CNJ
- `peixoto-ops/lke-skills-repo` - Scripts e automacoes

### Skills Hermes

- `central-pesquisas-peixoto-ops` - Sistema de pesquisas
- `sistema-health-scoring-casos-juridicos` - Health scoring
- `sistema-versionamento-cognitivo-lke` - Commits semanticos

---

**Documento gerado em:** 13 de abril de 2026  
**Sessao:** T0.1  
**Autor:** Hermes Agent (com input de Luiz Peixoto)
