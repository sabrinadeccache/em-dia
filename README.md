# Em Dia

**Controle de Certificações para Profissionais da Aviação**

Aplicação web responsiva para controladores de tráfego aéreo, pilotos e comissários de voo. Ela reúne num só lugar as certificações, habilitações e treinamentos de cada profissional, mostra a situação de cada validade e ajuda a planejar as renovações com antecedência.

Projeto Integrador IV, "Desenvolvimento de sistemas orientado a dispositivos móveis e baseados na web", do curso de Análise e Desenvolvimento de Sistemas do Centro Universitário Senac. Grupo 15.

> **Situação:** Etapa 1, planejamento. A implementação começa na Etapa 2. O planejamento completo está em [`docs/planejamento.md`](docs/planejamento.md).

## Requisitos funcionais

| ID | Requisito |
|---|---|
| RF01 | Cadastrar certificação, habilitação ou treinamento |
| RF02 | Listar os registros em ordem de vencimento, com a situação de cada um |
| RF03 | Consultar os detalhes de um registro |
| RF04 | Editar as informações e as datas de um registro |
| RF05 | Excluir um registro, mediante confirmação |
| RF06 | Painel com os totais de registros válidos, a vencer em até 30 dias e vencidos |

As datas de validade são informadas pelo usuário. A aplicação compara datas e não interpreta normas de cada profissão.

## Arquitetura

```
INTERFACE WEB RESPONSIVA   Next.js (React), TypeScript, Tailwind CSS
        ↕  HTTPS + JSON
SERVIÇO WEB REST            Route Handlers do Next.js (/api/...)
        ↕  cliente Supabase, com a sessão do usuário
BANCO DE DADOS              PostgreSQL no Supabase, com RLS por usuário
```

A autenticação usa o Supabase Auth, e a hospedagem fica na Vercel.

## Serviço REST

| Método | Rota | Finalidade |
|---|---|---|
| GET | `/api/certificados` | Listar os registros do usuário |
| GET | `/api/certificados/:id` | Consultar um registro |
| POST | `/api/certificados` | Cadastrar um registro |
| PUT | `/api/certificados/:id` | Editar um registro |
| DELETE | `/api/certificados/:id` | Excluir um registro |
| GET | `/api/painel` | Totais por situação |
| GET | `/api/categorias?profissao=` | Categorias da profissão |

## Equipe

| Integrante | Responsabilidade |
|---|---|
| Sabrina Deccache Pacheco | Coordenação; banco, API, acesso, telas e publicação |
| Caique de Come Rodrigues | Revisão do modelo de dados e das categorias |
| Carolina Ariadne de Sousa Cubas Reis | Revisão de texto e normas dos slides e do README |
| Endryo William Aprato Pires | Documentação das rotas REST (`docs/api.md`) |
| Esther Marques Silva | Testes de isolamento e de datas (`docs/testes.md`) |
| Marcos Henrique da Silva Xavier | Testes de responsividade em celulares e navegadores |
| Reinaldo Acioli de Amorim Oshiro | Roteiro dos testes de aceitação (`docs/aceitacao.md`) |
| Rodrigo Arroyo Nepomuceno | Roteiro, gravação e edição do vídeo (`docs/video.md`) |

## Como contribuir

1. Crie um *branch* a partir do `main`, com um nome que diga o que ele faz (por exemplo, `docs/api`).
2. Faça os *commits* nesse *branch*.
3. Abra um *pull request* para o `main`.

Problemas encontrados nos testes viram *issues*, com capturas de tela quando ajudarem. Assim a contribuição de cada integrante fica registrada.
