# DIAGNOSTICO: LKE Processos Hub

**Data:** 13 de abril de 2026  
**Sessao:** T0.1 - Diagnostico e Planejamento  
**Repositorio:** https://github.com/peixoto-ops/lke-processos-hub

---

## 1. RECURSOS EXISTENTES MAPEADOS

### 1.1 Vault Obsidian (inv_sa_02)

**Localizacao:** `/media/peixoto/Portable/inv_sa_02/`

O vault ja funciona como **Single Source of Truth** com estrutura organizada:

```
inv_sa_02/
├── 00 - Entrada/           # Arquivos recebidos, analises preliminares
├── 01 - Linha do Tempo/
│   ├── 00 - AUTOS/         # 44+ pastas de processos organizadas por numero CNJ
│   ├── 01 - Clientes/      # 39 fichas de clientes em markdown
│   └── 03 - Processos/     # Relatorios gerados pelo automate_reports
├── 02 - Conhecimento/      # Pesquisas, midia, arquitetura do sistema
├── 03 - Saida/             # Outputs, peticoes, documentos finais
├── 04 - Artigos/           # Material doutrinario
└── ARQUIVO_PROCESSOS_NORMALIZADOS/  # Processos processados com estrutura padrao
```

**Volumes atuais:**
- **44 processos** em 00 - AUTOS/
- **39 clientes** em 01 - Clientes/
- **28 relatorios** gerados em 03 - Processos/

### 1.2 Script automate_reports

**Localizacao:** `~/bin/automate_reports` (symlink para lke-skills-repo)

**Funcionalidade:**
- Consome API Comunica DJE (`https://comunicaapi.pje.jus.br/api/v1/comunicacao`)
- Aceita lista de processos (arquivo) OU processo unico (-n)
- Gera relatorios usando Fabric pattern `make_dje_report`
- Output: Markdown com timestamp no nome

**Fluxo atual:**
```
curl API DJE → JSON → Fabric (make_dje_report + qualifica_peixoto.md) → relatorio.md
```

### 1.3 Repositorio comunica_dje

**Organizacao:** `p31x070/comunica_dje`  
**Tecnologias:** Python, PostgreSQL (Docker), Obsidian, Signal

**Componentes relevantes:**
- Scripts Python para coleta de historico e atualizacoes diarias
- CLI unificada (`main.py`) com comandos: `setup`, `populate`, `ingest`, `generate-notes`
- Banco de dados PostgreSQL para consultas avancadas
- Integracao com Obsidian via vault dedicado (`03_Obsidian_Vault/`)
- Notificacoes Signal para alertas de pipeline

### 1.4 Fabric Patterns

**Localizacao:** `~/.config/fabric/patterns/`  
**Padroes juridicos identificados:**
- `lke_git_commiter` - Commits semanticos juridicos
- `analyze_legal_direct` / `analyze_legal_rational`
- Contextos customizados em `/media/peixoto/Portable/lke_master_vault/contexts`

### 1.5 Projetos Paralelos Relacionados

**ecosystem-dashboard** (`/media/peixoto/Portable/ecosystem-dashboard/`):
- Sistema de health scoring para casos juridicos
- Integracao Google Workspace (OAuth2 configurado)
- Dashboard Streamlit em desenvolvimento
- Roadmap: T1.5.2 (integracao real com Google Sheets)

---

## 2. GAPS E OPORTUNIDADES IDENTIFICADOS

### 2.1 Problemas Atuais

| Gap | Impacto | Prioridade |
|-----|---------|------------|
| Relatorios gerados em diretorio separado (03 - Processos/) | Desconexo dos autos principais | ALTA |
| Sem vinculo automatico cliente ↔ processos | Busca manual, risco de erro | ALTA |
| Fichas de clientes em formato nao estruturado | Dificil consulta e cruzamento | MEDIA |
| Sem indice centralizado de processos ativos | Visao fragmentada do portfolio | ALTA |
| Relatorios sem padronizacao de metadados | Dificil busca e rastreamento | MEDIA |

### 2.2 Oportunidades de Integracao

1. **Cross-reference automatico**: Vincular cada relatorio ao cliente e autos correspondentes
2. **Indice dinamico**: Gerar lista de processos ativos com status, proximas acoes e prazos
3. **Dashboard consolidado**: Visao unificada de portfolio processual no Obsidian
4. **Pipeline automatizado**: Coleta diaria de comunicacoes DJE com geracao de relatorios

---

## 3. PROPOSTA DE ARQUITETURA

### 3.1 Fluxo Proposto

```
┌─────────────────────────────────────────────────────────────────┐
│                    LKE PROCESSOS HUB                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  API CNJ/DJE ──► Python Collector ──► Fabric Patterns          │
│        │              │                    │                    │
│        │              ▼                    ▼                    │
│        │      PostgreSQL (cache)    Relatorios MD              │
│        │              │                    │                    │
│        │              ▼                    ▼                    │
│        └──────► Obsidian Vault (inv_sa_02) ◄──────────────────┘│
│                          │                                      │
│                          ▼                                      │
│              ┌───────────────────────┐                          │
│              │  00 - AUTOS/          │ ◄── Vinculo cliente      │
│              │  01 - Clientes/       │ ◄── Lista processos      │
│              │  03 - Processos/      │ ◄── Relatorios           │
│              │  INDEX_PROCESSOS.md   │ ◄── Indice central       │
│              └───────────────────────┘                          │
│                          │                                      │
│                          ▼                                      │
│              ecosystem-dashboard ◄── Health scoring             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Componentes a Desenvolver

#### A. Modulo de Indexacao (Python)
- Scan do vault para mapear processos existentes
- Extracao de metadados dos relatorios (numero CNJ, datas, status)
- Geracao de `INDEX_PROCESSOS.md` com tabela dinamica

#### B. Vinculo Cliente-Processo
- Parser das fichas de cliente em `01 - Clientes/`
- Identificacao de processos vinculados
- Atualizacao automatica de backlinks no Obsidian

#### C. Integracao Fabric Patterns
- Pattern `make_dje_report` melhorado com:
  - Metadados estruturados (YAML frontmatter)
  - Tags CDD (Classificacao Decimal do Direito)
  - Link automatico para pasta do cliente

#### D. Bridge comunica_dje ↔ inv_sa_02
- Configuracao do vault alvo
- Adaptacao do schema PostgreSQL para estrutura LKE
- Pipeline de sincronizacao

---

## 4. MATRIZ DE DECISAO: PROXIMOS PASSOS

| Fase | Tarefa | Dependencia | Estimativa |
|------|--------|-------------|------------|
| T1.1 | Criar indice de processos existentes | Nenhuma | 2h |
| T1.2 | Parser de fichas de clientes | T1.1 | 1.5h |
| T1.3 | Integrar vinculo cliente-processo | T1.2 | 2h |
| T1.4 | Adaptar make_dje_report com YAML | Fabric | 1h |
| T1.5 | Bridge comunica_dje ↔ inv_sa_02 | comunica_dje | 3h |
| T1.6 | Dashboard Obsidian (Dataview) | T1.1-T1.3 | 2h |

---

## 5. RISCOS E MITIGACOES

| Risco | Probabilidade | Mitigacao |
|-------|---------------|-----------|
| Mudanca na API CNJ | Baixa | Camada de abstracao |
| Conflito com estrutura existente do vault | Media | Backup + testes |
| Performance com muitos processos | Media | Indice incremental |
| Dependencia do Fabric | Baixa | Manter como opcao |

---

## 6. PROXIMA SESSAO: T1.1

**Objetivo:** Implementar modulo de indexacao

**Pre-requisitos:**
1. Vault inv_sa_02 acessivel em `/media/peixoto/Portable/inv_sa_02/`
2. Python 3.10+ com `pathlib`, `re`, `datetime`
3. Git configurado no repositorio lke-processos-hub

**Comando de inicio seguro:**
```bash
cd /media/peixoto/Portable/lke-processos-hub
python3 scripts/index_processos.py --dry-run
```

**Arquivos a criar:**
- `scripts/index_processos.py`
- `scripts/parser_clientes.py`
- `config/vault_paths.yaml`

---

**Documento gerado por Hermes Agent**  
**Sessao:** T0.1 - Diagnostico Inicial  
**Repositorio:** peixoto-ops/lke-processos-hub
