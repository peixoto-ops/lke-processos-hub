# INSTRUCOES PARA SESSAO T1.1

## Como Iniciar com Seguranca

---

### 1. PREPARACAO DO AMBIENTE

```bash
# Navegar para o repositorio
cd /media/peixoto/Portable/lke-processos-hub

# Verificar estado do git
git status

# Puxar atualizacoes (se houver)
git pull origin main
```

### 2. VERIFICACAO DE DEPENDENCIAS

```bash
# Verificar Python
python3 --version  # Esperado: 3.10+

# Verificar yq (YAML processor)
which yq || sudo snap install yq

# Verificar gh (GitHub CLI)
gh auth status

# Verificar Fabric
which fabric || echo "Instalar: go install github.com/danielmiessler/fabric/cli/fabric@latest"
```

### 3. TESTE DO LKE-INGEST

```bash
# Verificar se o script existe
ls -la ~/bin/lke-ingest

# Verificar versao
lke-ingest --version  # Esperado: v1.2.0

# Verificar ajuda
lke-ingest --help
```

### 4. TESTE DRY-RUN

```bash
# Criar arquivo de teste
cat > /tmp/test_cliente.txt << 'EOF'
Cliente: Maria da Silva
Telefone: +55 21 99999-8888
Origem: Indicacao - Dr. Carlos Santos
Assunto: Cobranca indevida cartao de credito
Urgencia: Alta
EOF

# Processar (dry-run - sem criar repositorio)
lke-ingest -f /tmp/test_cliente.txt
```

### 5. QUARENTENA DE SCRIPTS LEGADOS

```bash
# Criar diretorio de quarentena
mkdir -p ~/.lke-legacy

# Listar scripts LKE existentes
ls -la ~/bin/lke* | grep -v "lke-ingest"

# Mover scripts identificados como legados
# (APOS confirmacao manual)
mv ~/bin/lke_ingestor ~/.lke-legacy/ 2>/dev/null
mv ~/bin/lke_template_engine ~/.lke-legacy/ 2>/dev/null
mv ~/bin/lke_template_engine_clean ~/.lke-legacy/ 2>/dev/null
mv ~/bin/lke_template_engine_simple ~/.lke-legacy/ 2>/dev/null
```

### 6. IMPLEMENTACAO DE TELEMETRIA

Adicionar ao inicio de cada script:

```bash
LKE_LOG_FILE="$HOME/.lke_metrics/tools_usage.csv"

log_tool_usage() {
    local tool_name=$(basename "$0")
    local caller="${LKE_AGENT_CALLER:-luiz-terminal}"
    local timestamp=$(date -Iseconds)
    local args="$*"
    
    mkdir -p "$(dirname "$LKE_LOG_FILE")"
    
    if [ ! -f "$LKE_LOG_FILE" ]; then
        echo "timestamp,tool_name,caller,arguments" > "$LKE_LOG_FILE"
    fi
    
    echo "$timestamp,$tool_name,$caller,\"$args\"" >> "$LKE_LOG_FILE"
}

log_tool_usage "$@"
```

---

## CHECKLIST DA SESSAO

- [ ] Ambiente preparado
- [ ] Dependencias verificadas
- [ ] lke-ingest testado
- [ ] Scripts legados em quarentena
- [ ] Telemetria implementada
- [ ] Documentacao atualizada

---

## KANBAN

Acompanhar progresso em:
https://github.com/orgs/peixoto-ops/projects/18/views/1

---

## EM CASO DE ERRO

1. Verificar logs: `cat ~/.lke_metrics/tools_usage.csv`
2. Consultar diagnostico: `docs/DIAGNOSTICO_COMPLETO.md`
3. Abrir issue: `gh issue create --repo peixoto-ops/lke-processos-hub`

---

**Documento gerado para Sessao T1.1**  
**Data:** 13 de abril de 2026
