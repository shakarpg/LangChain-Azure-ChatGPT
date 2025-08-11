# Desafio DIO — Automatização da criação de testes unitários com LangChain + Azure ChatGPT

## 📌 Visão Geral

Este repositório demonstra como utilizar **LangChain** e **Azure OpenAI (ChatGPT)** para automatizar a criação de testes unitários em Python. O processo analisa funções e gera automaticamente arquivos de teste (`pytest`).

**Principais recursos:**
- Geração automática de testes via LangChain + Azure ChatGPT.
- Estrutura modular para fácil expansão.
- Integração com GitHub Actions para CI/CD.

---

## Estrutura do Repositório

```
├─ .github/workflows/tests.yml   # Pipeline GitHub Actions
├─ .gitignore
├─ README.md
├─ requirements.txt
├─ .env.example
├─ src/
│  ├─ app/
│  │  └─ sample_module.py        # Funções de exemplo
│  ├─ tests_generated/           # Saída dos testes gerados
│  └─ generate_tests.py          # Script principal
└─ images/                       # (opcional) Screenshots
```

---

## Pré-requisitos
- Python 3.10+
- Conta Azure OpenAI com chave e endpoint
- `pip install -r requirements.txt`

**requirements.txt:**
```
langchain>=0.0.200
openai>=0.27.0
python-dotenv
pytest
requests

aiohttp
```

---

## Configuração
1. Renomeie `.env.example` para `.env` e configure:
```
AZURE_OPENAI_API_KEY=YOUR_KEY
AZURE_OPENAI_ENDPOINT=https://your-resource-name.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o-chat
```

---

## Código de Exemplo

**src/app/sample_module.py**
```python
def soma(a: int, b: int) -> int:
    return a + b

def divide(a: float, b: float) -> float:
    return a / b
```

**src/generate_tests.py**
```python
import os
from dotenv import load_dotenv
from langchain.chat_models import AzureChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import LLMChain

load_dotenv()

llm = AzureChatOpenAI(
    deployment_name=os.getenv("AZURE_OPENAI_DEPLOYMENT_NAME"),
    openai_api_base=os.getenv("AZURE_OPENAI_ENDPOINT"),
    openai_api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    temperature=0.0,
    max_tokens=1000,
)

PROMPT = """
Você é um gerador de testes unitários em Python (pytest).
Código:
```
{source_code}
```
Gere o conteúdo do arquivo de teste.
"""

prompt = ChatPromptTemplate.from_template(PROMPT)
chain = LLMChain(llm=llm, prompt=prompt)

def generate_tests_for_source(source_code: str) -> str:
    return chain.run(source_code=source_code)

if __name__ == "__main__":
    with open("src/app/sample_module.py", "r") as f:
        source = f.read()
    tests_code = generate_tests_for_source(source)
    os.makedirs("src/tests_generated", exist_ok=True)
    with open("src/tests_generated/test_sample_module.py", "w") as fo:
        fo.write(tests_code)
    print("Testes gerados em src/tests_generated/")
```

---

## Execução Local
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python src/generate_tests.py
pytest src/tests_generated -q
```

---

## GitHub Actions

**.github/workflows/tests.yml**
```yaml
name: Generate and Run Tests
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - run: pip install -r requirements.txt
      - run: python src/generate_tests.py
        env:
          AZURE_OPENAI_API_KEY: ${{ secrets.AZURE_OPENAI_API_KEY }}
          AZURE_OPENAI_ENDPOINT: ${{ secrets.AZURE_OPENAI_ENDPOINT }}
          AZURE_OPENAI_DEPLOYMENT_NAME: ${{ secrets.AZURE_OPENAI_DEPLOYMENT_NAME }}
      - run: pytest src/tests_generated -q
```

---



