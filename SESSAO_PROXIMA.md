# PROXIMA SESSAO: T1.1 - Implementacao do Modulo de Indexacao

## Inicio Seguro

Antes de iniciar, verifique os pre-requisitos:

```bash
# 1. Navegue para o repositorio
cd /media/peixoto/Portable/lke-processos-hub

# 2. Verifique que o vault existe
ls -la /media/peixoto/Portable/inv_sa_02/

# 3. Verifique Python
python3 --version  # Esperado: 3.10+
```

## Comandos de Inicio

```bash
# Execute em modo de teste (sem modificar arquivos)
python3 scripts/index_processos.py --dry-run

# Execucao real (apos verificar output)
python3 scripts/index_processos.py
```

## Arquivos a Criar

| Arquivo | Proposito |
|---------|-----------|
| `scripts/index_processos.py` | Scanner do vault e geracao de indice |
| `scripts/parser_clientes.py` | Parser das fichas de clientes |
| `config/vault_paths.yaml` | JA CRIADO - caminhos do vault |

## Estrutura do Script index_processos.py

```python
#!/usr/bin/env python3
"""
LKE Processos Hub - Indexacao de Processos
Escaneia o vault Obsidian e gera indice centralizado
"""

import os
import re
from pathlib import Path
from datetime import datetime
import yaml

class VaultIndexer:
    def __init__(self, config_path: str):
        self.config = self._load_config(config_path)
        self.vault_root = Path(self.config['vault']['root'])
        
    def _load_config(self, path: str) -> dict:
        with open(path) as f:
            return yaml.safe_load(f)
    
    def scan_autos(self) -> list[dict]:
        """Escaneia pasta de autos e extrai metadados"""
        autos_path = self.vault_root / self.config['vault']['autos']
        processos = []
        
        for item in autos_path.iterdir():
            if item.is_dir() and self._is_cnj_number(item.name):
                processos.append({
                    'numero': item.name,
                    'path': str(item),
                    'files': len(list(item.rglob('*')))
                })
        return processos
    
    def _is_cnj_number(self, name: str) -> bool:
        """Valida formato CNJ: NNNNNNNN-NN.NNNN.N.NN.NNNN"""
        pattern = r'^\d{7}-\d{2}\.\d{4}\.\d\.\d{2}\.\d{4}$'
        return bool(re.match(pattern, name))
    
    def generate_index(self, processos: list[dict]) -> str:
        """Gera markdown com tabela de processos"""
        # TODO: Implementar geracao do INDEX_PROCESSOS.md
        pass

if __name__ == '__main__':
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument('--dry-run', action='store_true', help='Apenas simula, sem escrever')
    args = parser.parse_args()
    
    indexer = VaultIndexer('config/vault_paths.yaml')
    processos = indexer.scan_autos()
    print(f"Encontrados {len(processos)} processos")
    
    if not args.dry_run:
        # Gerar indice
        pass
```

## Checklist Pre-Sessao

- [ ] Vault acessivel em `/media/peixoto/Portable/inv_sa_02/`
- [ ] Python 3.10+ disponivel
- [ ] Dependencias: `pip install pyyaml`
- [ ] Repositorio git sincronizado

## Pos-Sessao

Apos concluir T1.1, commit com:

```bash
cd /media/peixoto/Portable/lke-processos-hub
legal_commit
# Mensagem sugerida: feat(index): modulo de indexacao de processos com scanner do vault
```

---

**Documento de transicao preparado para sessao T1.1**
