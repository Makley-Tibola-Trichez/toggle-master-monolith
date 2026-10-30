# Por que a aplicação é um monólito

## Evidências no código

Uma aplicação monolítica concentra todas as responsabilidades em uma única unidade de código, implantada e escalada como um bloco. No ToggleMaster:

| Característica | Evidência |
| :-- | :-- |
| Um único arquivo e processo para toda a lógica | `app.py` contém configuração, acesso a dados, inicialização do esquema, CLI e todas as rotas. |
| Camadas misturadas | Cada rota abre a conexão, executa SQL e monta a resposta (`app.py:53-140`), sem separar camada de dados, de domínio e de apresentação. |
| Um único banco compartilhado | Uma tabela `flags` em um PostgreSQL, acessada diretamente por todas as rotas. |
| Implantação única | Uma imagem/processo (Gunicorn) serve todos os endpoints (`Dockerfile:22-23`, `entrypoint.sh:37`). |
| Escala acoplada | Aumentar a capacidade de leitura de flags exige replicar também a escrita e a administração, pois tudo é o mesmo serviço. |

Hoje o domínio é pequeno (só flags), mas as fases seguintes do projeto preveem separá-lo em serviços distribuídos. O formato atual é o ponto de partida dessa evolução.

## Vantagens para um MVP

- **Rapidez de entrega**: um repositório, um artefato, um deploy.
- **Simplicidade operacional**: sem rede entre serviços, sem orquestração, sem contratos entre APIs.
- **Fácil de rodar e depurar**: `docker compose up` sobe tudo; o fluxo de uma requisição está em um único lugar.
- **Consistência de dados simples**: uma única base, sem transações distribuídas.
- **Custo baixo**: uma EC2 e um RDS atendem a necessidade inicial (ver a estimativa em [ToggleMaster_Relatorio_Entrega_Fase1.docx](ToggleMaster_Relatorio_Entrega_Fase1.docx)).

## Desvantagens e riscos

- **Escala só em bloco**: não é possível escalar apenas o trecho de maior carga (por exemplo, a leitura de flags).
- **Ponto único de falha**: uma falha no processo, na instância ou no banco derruba a API inteira. A arquitetura atual usa uma única EC2 em uma única zona de disponibilidade (ver [05-arquitetura-aws.md](05-arquitetura-aws.md)).
- **Acoplamento**: mudanças em uma parte do `app.py` podem afetar as demais; a ausência de testes automatizados agrava o risco.
- **Evolução limitada**: adotar outra tecnologia para uma parte do sistema, ou dividir times por domínio, exige refatoração.
- **Deploy de risco maior**: qualquer alteração exige reimplantar tudo.
- **Reuso limitado de recursos**: conexão nova por requisição, sem pool (`app.py:14-21`), pesa mais conforme o volume cresce.

## Conclusão

Para validar a ideia com rapidez e custo baixo, o monólito é uma escolha adequada. Para cargas maiores, alta disponibilidade e evolução por domínio, será necessário separar a aplicação, e os ajustes listados em [04-12-factor.md](04-12-factor.md) indicam por onde começar.
