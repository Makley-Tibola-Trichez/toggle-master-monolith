# Documentação do ToggleMaster (Tech Challenge – Fase 1)

O ToggleMaster é uma plataforma de *Feature Flag as a Service*: uma API REST que permite ativar ou desativar funcionalidades em produção sem um novo deploy. Nesta fase ele é um MVP monolítico (Flask + PostgreSQL), executado localmente com Docker Compose e implantado manualmente na AWS (EC2 + RDS).

Esta pasta reúne a documentação exigida pelo enunciado da Fase 1.

## Conteúdo

| Documento | Exigência do enunciado |
| :-- | :-- |
| [05-arquitetura-aws.md](05-arquitetura-aws.md) | Diagrama de arquitetura (VPC, sub-redes pública e privada, EC2, RDS, Security Groups) e como as credenciais do banco são gerenciadas. |
| [04-12-factor.md](04-12-factor.md) | Discussão 12-Factor App aplicada ao projeto. |
| [03-monolito.md](03-monolito.md) | Por que a aplicação é um monólito, com vantagens e desvantagens para um MVP. |
| [ToggleMaster_Relatorio_Entrega_Fase1.docx](ToggleMaster_Relatorio_Entrega_Fase1.docx) | Relatório de entrega: participantes, links, decisões e estimativa de custos. |

## Convenções

- Referências ao código seguem o formato `arquivo:linha`.
- Campos entre `<...>` são informações que só o grupo conhece e precisam ser preenchidas.
