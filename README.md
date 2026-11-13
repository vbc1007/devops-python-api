DevOps Python API Descrição da Aplicação e Objetivo Esta aplicação é uma API desenvolvida em Python. O objetivo deste projeto é demonstrar a implementação de um ciclo de vida de desenvolvimento de software (SDLC) completo com práticas de DevOps. O repositório conta com integração contínua (CI) para garantir a qualidade do código e entrega contínua (CD) para automatizar a publicação na nuvem (AWS), eliminando intervenções manuais na implantação.

Tecnologias Utilizadas Linguagem: Python 3.x

Qualidade e Testes: Ruff (Lint e Formatação), Pytest (Testes unitários e cobertura)

Containerização: Docker

CI/CD: GitHub Actions

Cloud Provider (AWS): Amazon ECR (Elastic Container Registry) para armazenamento de imagens e Amazon EC2 (Elastic Compute Cloud) para hospedagem da aplicação.

Instruções para Execução Local Clone o repositório: git clone https://github.com/Da20-Costa/devops-python-api.git cd devops-python-api

Crie e ative um ambiente virtual: python -m venv venv source venv/bin/activate

Instale as dependências: pip install -r requirements.txt pip install -r requirements-dev.txt

Execute a aplicação: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

Procedimento de Construção e Execução (Docker) Construa a imagem Docker: docker build -t devops-python-api:local .

Execute o contêiner: docker run -d --name devops-python-api -p 8000:8000 devops-python-api:local

Verifique os logs se necessário: docker logs devops-python-api

Pipelines de Integração e Entrega Contínua (CI/CD) O projeto utiliza o GitHub Actions para orquestrar as pipelines, divididas em etapas lógicas:

Integração Contínua (CI) Executada a cada Push ou Pull Request para a branch main.

Qualidade de Código: Instala dependências, executa o ruff para verificar formatação e falhas estáticas.

Testes Automatizados: Executa o pytest para rodar a suíte de testes e gerar relatórios de cobertura em XML.

Smoke Test em Container: Constrói a imagem Docker, sobe o contêiner e realiza requisições HTTP na rota /health para garantir que a aplicação inicializa corretamente antes do deploy.

Entrega Contínua (CD) Executada apenas após um Push (merge) bem-sucedido na branch main e aprovação na etapa de CI.

Push para o ECR: Autentica na AWS usando OIDC/Secrets, constrói a imagem com a tag correspondente ao commit da versão (github.sha) e envia para o Amazon ECR.

Deploy na EC2: Estabelece uma conexão SSH segura com a instância EC2.

Atualização do Contêiner: A instância autentica no ECR, baixa a nova imagem, para e remove o contêiner da versão antiga, e inicia a nova versão mapeando a porta 8000.

Arquitetura de Implantação A infraestrutura foi desenhada da seguinte forma:

O desenvolvedor realiza o push do código no GitHub.

O GitHub Actions assume o processo, realizando o build e enviando a imagem Docker para um repositório privado no Amazon ECR.

Via protocolo SSH, o GitHub Actions acessa a instância Amazon EC2.

A instância EC2 possui uma IAM Role anexada que a autoriza a baixar a imagem do ECR (permissão ReadOnly) sem a necessidade de chaves fixas no servidor.

O contêiner é substituído e passa a expor a aplicação na porta 8000.

Acesso à API Publicada A aplicação está disponível e acessível publicamente através do endereço:

URL Base: http://3.90.235.203:8000

Health Check: http://3.90.235.203:8000/health

Instruções para Verificar a Versão Implantada Para validar qual versão da aplicação está rodando atualmente em produção:

Acesse a rota de Health Check ou a rota raiz da API no navegador.

Identifique a tag do commit do último merge na branch main do repositório.

Se desejar verificar internamente na EC2, conecte-se via SSH e execute: docker ps A coluna IMAGE mostrará a URI do ECR terminada com o hash do último commit.

Evidências da Execução das Pipelines (Adicione os prints de tela que comprovem o funcionamento do fluxo nos caminhos abaixo)

Pipeline de CI/CD finalizada com sucesso (GitHub Actions): ./docs/ci-cd-success.png

Imagem armazenada no Amazon ECR: ./docs/ecr-image.png

Aplicação respondendo na EC2 (Browser/Postman): ./docs/api-response.png

Nota de Segurança: Por boas práticas, nenhuma credencial, token ou chave primária foi embutida neste código-fonte ou documentação. Todos os segredos operacionais estão isolados utilizando o GitHub Actions Secrets e AWS IAM Roles.

# API Python para demonstrar DevOps

Repositório didático para apresentar, em uma única aplicação, três ideias que se conectam:

1. uma API HTTP escrita em Python;
2. integração contínua com GitHub Actions;
3. execução reproduzível com Docker . 

O projeto utiliza FastAPI, pytest, Ruff, GitHub Actions e Docker. Os dados ficam em memória para manter a atenção da aula no fluxo DevOps, sem introduzir banco de dados.

## O que os estudantes devem observar

- O código funciona localmente e no runner do GitHub.
- A mesma suí te de testes é executada pelo desenvolvedor e pelo pipeline.
- O Dockerfile descreve o ambiente necessário para executar a API.
- O pipeline só testa o container depois que qualidade e testes foram aprovados.
- Logs, cobertura e relatórios funcionam como evidências, não apenas como mensagens coloridas.

## Estrutura do repositório

```text
.
├── .github/workflows/ci.yml   # pipeline de integração contínua
├── app/
│   ├── __init__.py
│   └── main.py                # API FastAPI
├── docs/
│   ├── API.md
│   ├── DOCKER.md
│   ├── GITHUB-ACTIONS.md
│   └── ROTEIRO-AULA.md
├── tests/test_api.py          # testes automatizados
├── .dockerignore
├── .python-version
├── compose.yaml
├── Dockerfile
├── pyproject.toml             # configuração de pytest, cobertura e Ruff
├── requirements.txt           # dependências de execução
└── requirements-dev.txt       # dependências de desenvolvimento e CI
```

## 1. Executar localmente

Pré-requisitos: Git e Python 3.13.

### Linux e macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --requirement requirements-dev.txt
uvicorn app.main:app --reload
```

### Windows PowerShell

```powershell
py -3.13 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --requirement requirements-dev.txt
uvicorn app.main:app --reload
```

Acesse:

- API: <http://localhost:8000>
- documentação Swagger: <http://localhost:8000/docs>
- verificação de saúde: <http://localhost:8000/health>

## 2. Experimentar a API

```bash
curl http://localhost:8000/health

curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Estudar integração contínua"}'

curl http://localhost:8000/tasks

curl -X PATCH http://localhost:8000/tasks/1/done
```

Endpoints:

| Método | Caminho | Objetivo | Resposta esperada |
|---|---|---|---|
| GET | `/` | Apresentar a API | `200` |
| GET | `/health` | Informar saúde e versão | `200` |
| GET | `/tasks` | Listar tarefas | `200` |
| POST | `/tasks` | Criar uma tarefa | `201` |
| PATCH | `/tasks/{id}/done` | Concluir uma tarefa | `200` ou `404` |

Veja a explicação do código em [docs/API.md](docs/API.md).

## 3. Executar as verificações locais

```bash
ruff check .
ruff format --check .
pytest --cov=app --cov-report=term-missing
```

Esses comandos são deliberadamente iguais aos usados no GitHub Actions. Se uma verificação falhar localmente, ela também deverá falhar no pipeline.

Para aplicar a formatação automaticamente:

```bash
ruff format .
```

## 4. Publicar no GitHub e acionar a CI

O diretório já é inicializado como repositório Git local. Crie um repositório vazio no GitHub e execute:

```bash
git remote add origin https://github.com/SEU-USUARIO/devops-python-api.git
git push -u origin main
```

O arquivo `.github/workflows/ci.yml` dispara o pipeline em três situações:

- `push` na branch `main`;
- `pull_request` destinado à `main`;
- execução manual pela aba **Actions**.

Na aba **Actions**, abra a execução e faça a leitura nesta ordem:

1. workflow;
2. job;
3. primeiro step que falhou;
4. comando e mensagem que explicam a falha.

A explicação detalhada do YAML está em [docs/GITHUB-ACTIONS.md](docs/GITHUB-ACTIONS.md).

## 5. Executar com Docker

```bash
docker build -t devops-python-api:v1 .
docker run --rm -p 8000:8000 --name devops-python-api devops-python-api:v1
```

Em outro terminal:

```bash
curl http://localhost:8000/health
docker ps
docker logs devops-python-api
```

Com Docker Compose:

```bash
docker compose up --build
docker compose down
```

Veja a leitura detalhada do Dockerfile em [docs/DOCKER.md](docs/DOCKER.md).

## Falhas intencionais para demonstrar

### Falha de teste

Em `app/main.py`, altere temporariamente a versão do health check para `2.0.0`. O teste espera `1.0.0`, então o pytest e o pipeline devem falhar.

### Falha de qualidade

Adicione um import não utilizado no início de `app/main.py`:

```python
import os
```

O step `ruff check .` localizará o problema.

### Falha de rede no container

No `Dockerfile`, troque `--host 0.0.0.0` por `--host 127.0.0.1`. O processo iniciará, mas o serviço não ficará acessível pela porta publicada no host.

## Próximos desafios

- Adicionar um endpoint `DELETE /tasks/{id}` e seus testes.
- Executar os testes em uma matriz com Python 3.12, 3.13 e 3.14.
- Publicar a imagem em um registry somente após a aprovação dos testes.
- Substituir o armazenamento em memória por um banco de dados.

## Referências oficiais

- [GitHub Actions: criar e testar Python](https://docs.github.com/pt/actions/tutorials/build-and-test-code/building-and-testing-python)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/setup-python](https://github.com/actions/setup-python)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Docker para Python](https://docs.docker.com/language/python/)
