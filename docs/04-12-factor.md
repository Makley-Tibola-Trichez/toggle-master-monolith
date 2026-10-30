# Avaliação 12-Factor App

Avaliação da aplicação frente à metodologia [12-Factor App](https://12factor.net/pt_br/). Legenda: **Atende**, **Parcial** (atende com ressalvas) ou **Não atende**.

## Resumo

| # | Fator | Situação |
| :-- | :-- | :-- |
| I | Base de código | Atende |
| II | Dependências | Atende |
| III | Configurações | Parcial |
| IV | Serviços de apoio | Atende |
| V | Build, release, run | Parcial |
| VI | Processos | Atende |
| VII | Vínculo de porta | Atende |
| VIII | Concorrência | Parcial |
| IX | Descartabilidade | Parcial |
| X | Paridade dev/prod | Parcial |
| XI | Logs | Parcial |
| XII | Processos administrativos | Parcial |

## Detalhamento

### I. Base de código — Atende
Um único repositório Git versiona a aplicação e é a origem de todos os ambientes (local e EC2).

### II. Dependências — Atende
`requirements.txt` declara as bibliotecas com versões fixas (`requirements.txt:1-4`) e o `Dockerfile` as instala de forma isolada (`Dockerfile:10`); na EC2 usa-se um *virtualenv*.
*Ajuste*: declarar `click` (importado em `app.py:2`) e considerar um arquivo de bloqueio (*lock*) com as dependências transitivas.

### III. Configurações — Parcial
A conexão com o banco vem de variáveis de ambiente (`app.py:9-12`), sem credenciais no código.
*Ressalvas*:
- O `docker-compose.yaml` traz usuário e senha fixos (`docker-compose.yaml:9-12`, `:19-21`).
- `DB_PORT` é exigida pelo `entrypoint.sh` (`entrypoint.sh:12`), mas ignorada pela aplicação, que usa a porta padrão.
- Nenhuma validação na aplicação: se uma variável faltar, o erro só aparece na primeira conexão.

*Ajuste*: usar arquivo `.env` fora do Git (ou segredos do ambiente), validar variáveis na inicialização e fazer `app.py` ler `DB_PORT`.

### IV. Serviços de apoio — Atende
O PostgreSQL é tratado como recurso anexado, localizado por `DB_HOST`. Trocar o contêiner local pelo RDS exige apenas mudar a configuração.

### V. Build, release, run — Parcial
Há separação mínima: o `Dockerfile` faz o *build* e o Gunicorn executa. Mas não há versionamento de *releases*, pipeline de CI/CD nem imagens publicadas; na EC2 o deploy é manual por `git clone` e `pip install`.
*Ajuste*: pipeline que gere imagem versionada e a implante de forma repetível.

### VI. Processos — Atende
A aplicação não guarda estado em memória ou disco entre requisições; todo estado está no PostgreSQL.

### VII. Vínculo de porta — Atende
A aplicação se exporta por porta (5000) e é servida diretamente pelo Gunicorn (`Dockerfile:20`, `entrypoint.sh:37`).

### VIII. Concorrência — Parcial
Por ser sem estado, escala horizontalmente por processos. Porém o Gunicorn roda com a configuração padrão (um único *worker* síncrono) e não há definição de quantidade de *workers* ou de réplicas.
*Ajuste*: definir `--workers` conforme CPU e prever múltiplas instâncias atrás de um balanceador.

### IX. Descartabilidade — Parcial
A inicialização é rápida e o `entrypoint.sh` espera o banco antes de subir. Pontos fracos: cada requisição abre e fecha uma conexão nova (`app.py:14-21`), sem pool; e o encerramento gracioso depende do comportamento padrão do Gunicorn, sem tratamento explícito.
*Ajuste*: pool de conexões e *timeout*/graceful shutdown configurados.

### X. Paridade dev/prod — Parcial
Localmente roda em contêiner com `postgres:13`; na EC2 do guia roda direto com *virtualenv* e RDS, e a versão do PostgreSQL do RDS pode diferir. Além disso, o Compose monta o código como volume (`docker-compose.yaml:6-7`), o que não existe em produção.
*Ajuste*: rodar a mesma imagem Docker nos dois ambientes e alinhar a versão do PostgreSQL.

### XI. Logs — Parcial
A aplicação escreve em `stdout` com `print` (`app.py:24`, `:39`, `:41`, `:43`), e `PYTHONUNBUFFERED` está ativo (`Dockerfile:4`), o que é compatível com o fator. Falta, porém, log estruturado, níveis e log de requisições/erros nas rotas.
*Ajuste*: usar o módulo `logging` com formato estruturado e agregar logs fora da instância (por exemplo, CloudWatch).

### XII. Processos administrativos — Parcial
A criação da tabela é um comando administrativo pontual: `flask init-db` (`app.py:45-47`). No Docker ele roda automaticamente (`entrypoint.sh:30-33`), mas com `|| true`, então uma falha é ignorada, e `init_db` captura e apenas imprime as exceções (`app.py:40-43`). O guia de deploy na EC2 do `README.md` (Gunicorn direto) **não** executa o comando, e a tabela pode não existir; no deploy do grupo ele foi executado por meio do `entrypoint.sh`, e a criação da tabela foi conferida com `\dt` no `psql`.
*Ajuste*: ferramenta de migração (por exemplo, Alembic) e falhar a inicialização quando a migração falhar.
