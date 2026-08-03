# Configuração Natixis LLM CLI (via 9Router)

Este documento descreve como configurar uma interface CLI para acessar modelos através do seu gateway local **9Router** (`http://localhost:20128`).

## 1. Pré-requisitos
Certifique-se de que a máquina possui:
- Python 3 instalado (`python3`).
- Biblioteca `requests` instalada:
  ```bash
  pip3 install requests
  ```

## 2. Script de Interface (`natixis-llm.py`)
Salve este script em `C:\DEV\natixis\natixis_dev\natixis-llm.py`:

```python
import sys, requests, json

# Gateway 9Router local
URL = "http://localhost:20128/v1/chat/completions"

def chat(prompt):
    data = {
        "model": "gemini-2.0-flash", 
        "messages": [{"role": "user", "content": prompt}]
    }
    try:
        r = requests.post(URL, json=data, timeout=30)
        print(r.json()['choices'][0]['message']['content'])
    except Exception as e:
        print(f"Erro ao conectar ao 9Router: {e}", file=sys.stderr)

if __name__ == "__main__":
    chat(" ".join(sys.argv[1:]))
```

## 3. Configuração do Alias
Para facilitar o uso, adicione um alias ao seu arquivo `.bashrc` ou `.bash_profile` na máquina da Natixis:

```bash
alias natixis-gemini='python3 /c/dev/natixis/natixis_dev/natixis-llm.py'
```

Após adicionar, aplique a alteração:
```bash
source ~/.bashrc
```

## 4. Teste de Conexão
Para verificar se tudo está configurado corretamente, execute:
```bash
natixis-gemini "Olá, Natixis! Teste de conexão concluído."
```
