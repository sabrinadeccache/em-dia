# Em Dia — Controle de Certificações para Profissionais da Aviação

Planejamento da aplicação (Projeto Integrador IV, Centro Universitário Senac, Grupo 15).


## 1. Contexto acadêmico

**Disciplina:** Projeto Integrador IV, "Desenvolvimento de sistemas orientado a dispositivos móveis e baseados na web" (Centro Universitário Senac, EAD).

**A Etapa 1 (planejamento) pede uma apresentação em PDF de aproximadamente 8 a 10 slides com:**
- capa (título e integrantes)
- problema e contexto
- objetivo da aplicação
- requisitos funcionais (de 4 a 6, objetivos e possíveis de implementar no prazo)
- tecnologias (front-end, servidor, banco de dados e controle de versão, com a finalidade de cada uma)
- diagrama de arquitetura (interface web responsiva ↔ serviço web REST ↔ banco de dados)
- operações REST
- organização das tarefas (cada integrante com participação identificável)
- justificativa do uso de nuvem (pode ser conceitual)
- link do repositório GitHub

O planejamento deve ser coerente com o que será implementado na Etapa 2.

**Repositório.** O enunciado chama o repositório de "opcional" no item 9, mas inclui o link na lista de slides, e o roteiro geral da Senac trata o GitHub como parte do desenvolvimento. O grupo vai criar o repositório e colocar o link real no slide. Enquanto ele não existir, o link fica como pendente.

**O roteiro geral da Senac (material da disciplina) também pede:**
- ciclo de vida justificado
- premissas
- requisitos com classificação
- modelo de dados
- webservice REST
- front-end responsivo
- justificativa de nuvem
- vídeo de até 3 minutos

Esses itens orientam o planejamento. Os relatórios extensos e o vídeo pertencem às etapas seguintes e não são entregáveis desta Etapa 1.

**Critérios de avaliação:**
- organização do texto
- conteúdo próprio
- rigor técnico
- profundidade
- português e terminologia (termos em inglês em itálico)
- evolução e contribuição de cada integrante no repositório
- aderência do vídeo às histórias de usuário

**Integrantes (Grupo 15, em ordem alfabética):** Caique de Come Rodrigues, Carolina Ariadne de Sousa Cubas Reis, Endryo William Aprato Pires, Esther Marques Silva, Marcos Henrique da Silva Xavier, Reinaldo Acioli de Amorim Oshiro, Rodrigo Arroyo Nepomuceno, Sabrina Deccache Pacheco.

## 2. Visão geral da solução

**Título:** Em Dia — Controle de Certificações para Profissionais da Aviação.

**Problema.** Controladores de tráfego aéreo, pilotos e comissários de voo dependem de certificações, habilitações e treinamentos válidos para exercer suas funções, e cada documento possui um prazo de vencimento distinto. Quando esses registros ficam dispersos entre papéis, planilhas e mensagens, o acompanhamento dos prazos torna-se impreciso, e uma renovação esquecida pode resultar no afastamento temporário do profissional de suas atividades.

**Objetivo.** Desenvolver uma aplicação web responsiva que centralize os registros de certificações, habilitações e treinamentos de profissionais da aviação, ofereça uma visão clara da situação de cada validade e apoie o planejamento antecipado das renovações.

**Público-alvo.** Controladores de tráfego aéreo, pilotos e comissários de voo. No formulário de cadastro, o usuário escolhe a profissão para filtrar as categorias predefinidas e pode registrar outros tipos por meio da opção "Outro". A profissão não é gravada na conta nem no registro; ela serve apenas como filtro. Cada usuário vê e gerencia apenas os próprios registros.

## 3. Requisitos

### 3.1 Requisitos funcionais

| ID | Requisito | Descrição |
|---|---|---|
| RF01 | Cadastrar registro | O usuário cadastra uma certificação, habilitação ou treinamento. Campos: profissão (filtro), categoria, nome livre (obrigatório se a categoria for "Outro"), número, órgão emissor, data de emissão, data de validade (obrigatória) e observações. |
| RF02 | Listar registros | Lista os registros do usuário em ordem crescente de data de validade, exibindo a situação de cada um. |
| RF03 | Consultar detalhes | Exibe todos os dados de um registro e a sua situação. |
| RF04 | Editar registro | Altera as informações e as datas de um registro. A nova validade substitui a anterior. |
| RF05 | Excluir registro | Remove um registro depois de uma confirmação explícita na tela. |
| RF06 | Painel de validades | Exibe os totais de registros válidos, a vencer em até 30 dias e vencidos, com destaque para os que vencem nos próximos 30 dias. |

**Acesso à aplicação (condição transversal, não é um RF).** O usuário cria a própria conta com e-mail e senha (cadastro público pelo Supabase Auth), entra e sai da aplicação. As telas de acesso são: **Criar conta**, **Entrar** e **Sair** (botão no cabeçalho). Todas as demais telas e todas as rotas da API exigem sessão. A integração e os testes do acesso têm responsáveis definidos na seção 9.

**Regra de situação.** A situação é calculada pelo servidor em toda consulta e nunca é armazenada.
- **Referência de "hoje":** a data atual no fuso `America/Sao_Paulo`, calculada num único ponto do código e usada pela lista, pelos detalhes e pelo painel.
- **Datas:** todas as datas são do tipo `date`, sem horário.
- **Vencido:** a validade é anterior a hoje.
- **A vencer:** a validade está entre hoje e hoje + 30 dias (inclusive). Vencer hoje ainda conta como "a vencer".
- **Válido:** a validade é posterior a hoje + 30 dias.
- **Antecedência:** 30 dias, regra definida pelo grupo e guardada numa única constante.
- **Casos de teste obrigatórios:** validade ontem (vencido), hoje (a vencer), hoje + 30 (a vencer), hoje + 31 (válido).

**Validações (no servidor e no banco):**

| Campo | Regra |
|---|---|
| categoria | obrigatória; precisa existir. No cadastro, ou quando a categoria é trocada na edição, ela precisa estar ativa. Na edição que mantém a categoria atual, uma categoria desativada é aceita. |
| nome livre | obrigatório se a categoria for "Outro" (identificada pelo código `outro`, não pelo nome); proibido nas demais; até 80 caracteres |
| número | opcional; até 40 caracteres |
| órgão emissor | opcional; até 80 caracteres |
| data de emissão | opcional; não pode ser posterior à validade |
| data de validade | obrigatória |
| observações | opcional; até 500 caracteres |

### 3.2 Requisitos não funcionais

| ID | Requisito |
|---|---|
| RNF01 | Interface responsiva, com prioridade para telas de celular (*mobile first*). |
| RNF02 | Isolamento dos dados: cada usuário só lê, altera, exclui e contabiliza os próprios registros, garantido no servidor e no banco (*Row Level Security*). Detalhes na seção 8. |
| RNF03 | Privacidade: coleta mínima, sem arquivos ou imagens; finalidade informada ao usuário; aviso no campo de observações; procedimento de exclusão da conta. Detalhes na seção 4. |
| RNF04 | API REST com respostas em JSON e códigos HTTP adequados (200, 201, 204, 400, 401, 404). |
| RNF05 | Aplicação hospedada em nuvem e acessível por navegador. |

### 3.3 Histórias de usuário (base para o vídeo)

- Como **controlador de tráfego aéreo**, quero cadastrar a validade da minha proficiência em inglês para não perder o prazo de revalidação.
- Como **piloto**, quero ver no painel o que vence nos próximos 30 dias para agendar as renovações com antecedência.
- Como **comissário de voo**, quero editar a validade de um treinamento depois de renová-lo para manter meus registros atualizados.

## 4. Premissas, privacidade e escopo

**Premissas desta versão:**
- As datas de validade são informadas pelo próprio usuário. A aplicação compara datas e não interpreta normas de cada profissão nem calcula prazos regulamentares.
- Os alertas de vencimento aparecem no próprio painel, com destaque para os registros que vencem nos próximos 30 dias (regra definida pelo grupo).
- A edição de um registro substitui a validade anterior. O histórico de renovações está previsto para versões futuras.

**Privacidade:**
- **Finalidade.** Os dados são usados exclusivamente para que o próprio usuário acompanhe as suas validades. Isso é informado na tela de criação de conta.
- **Dados coletados:**
  - da conta: e-mail e senha (a senha é guardada pelo Supabase Auth);
  - de cada registro: os campos do RF01.
  - Arquivos e imagens não são coletados.
- **Campo de observações.** Traz a orientação "Não registre diagnósticos, dados de saúde ou outras informações pessoais desnecessárias".
- **Exclusão.** O usuário pode excluir cada registro (RF05). A exclusão da conta é feita por solicitação, sem tela própria nesta versão:
  1. O usuário envia o pedido para o e-mail de contato informado na aplicação.
  2. A responsável (Sabrina, na coordenação) não confia no endereço declarado na mensagem: ela envia um pedido de confirmação **para o e-mail cadastrado na conta**.
  3. A exclusão só é feita depois que o titular confirma a partir desse endereço. Ela é feita pelo painel administrativo do Supabase Auth, e os registros são apagados em cascata.
  4. O titular é avisado da conclusão. Pedidos sem confirmação em 15 dias são arquivados sem exclusão.

  A exclusão pela própria tela é uma evolução futura.
- **Demonstração.** O vídeo e os testes usam apenas contas e dados fictícios.
- Essas medidas orientam o planejamento de privacidade, mas não constituem, sozinhas, comprovação de conformidade integral com a LGPD.

**Fora do escopo (evoluções futuras):**
- histórico de renovações
- exclusão da conta pela própria tela
- envio de cópias digitais dos documentos
- notificações por e-mail e aplicativos de mensagem
- antecedência do alerta escolhida pelo usuário
- integração com órgãos emissores
- articulação com plataformas de preparação profissional, como a OnFreq, para apoiar a revalidação da proficiência linguística

## 5. Tecnologias

| Camada | Tecnologia | Finalidade |
|---|---|---|
| Front-end | Next.js (React) + TypeScript + Tailwind CSS | Interface web responsiva |
| Servidor | Route Handlers do Next.js (`src/app/api/...`) | Serviço web REST: recebe as requisições, confere a sessão, valida e aplica as regras |
| Banco de dados | Supabase (PostgreSQL) | Armazena categorias e registros |
| Autenticação | Supabase Auth | Criação de conta, entrada e saída; identifica o usuário em cada requisição |
| Controle de versão | Git + GitHub | Histórico do código e contribuição de cada integrante (*branches* e *pull requests*) |
| Hospedagem | Vercel | Publicação da aplicação na nuvem |

## 6. Arquitetura

```
INTERFACE WEB RESPONSIVA   (Next.js / React, no navegador ou no celular)
        ↕  HTTPS + JSON
SERVIÇO WEB REST            (/api/... — Route Handlers do Next.js)
        ↕  cliente Supabase (API de dados), com a sessão do usuário
BANCO DE DADOS              (PostgreSQL no Supabase, com RLS)
```

**Dois fluxos distintos:**
1. **Autenticação.** As telas Criar conta, Entrar e Sair usam o Supabase Auth. A sessão fica num *cookie* que acompanha as requisições à API.
2. **Dados.** Certificados, categorias e painel passam sempre pelos Route Handlers; o front-end não consulta as tabelas diretamente.
   - Em cada rota, o servidor lê a sessão e recusa a requisição sem ela (401).
   - Em seguida, consulta o banco com o cliente Supabase autenticado como o próprio usuário. Não se usa a chave de serviço, para que a RLS continue valendo.

## 7. Serviço REST

| Método | Rota | Operação | Finalidade | RF |
|---|---|---|---|---|
| GET | `/api/certificados` | Listar | Registros do usuário, ordenados pela validade, com a situação | RF02 |
| GET | `/api/certificados/:id` | Consultar | Um registro específico | RF03 |
| POST | `/api/certificados` | Cadastrar | Cria um registro (201) | RF01 |
| PUT | `/api/certificados/:id` | Atualizar | Altera um registro existente | RF04 |
| DELETE | `/api/certificados/:id` | Excluir | Remove um registro (204); a confirmação fica na tela | RF05 |
| GET | `/api/painel` | Resumo | Totais de registros válidos, a vencer e vencidos | RF06 |
| GET | `/api/categorias?profissao=piloto` | Listar categorias | Categorias ativas da profissão, mais as de "todas" e "Outro" (apoio ao RF01) | — |

**Comportamento esperado:**
- sem sessão: 401;
- registro de outro usuário ou inexistente: 404, sem revelar se o registro existe;
- dados inválidos: 400, com a mensagem do campo;
- o `usuario_id` nunca vem do navegador: o servidor o obtém da sessão, e um `usuario_id` enviado no corpo é ignorado.

## 8. Modelo de dados (relacional)

**categorias**
- `id` (PK)
- `codigo` (único e estável; por exemplo `proficiencia_icao`, `cma`, `outro`)
- `nome` (`not null`)
- `profissao` (`not null`): controlador, piloto, comissário ou todas
- `ativa` (booleano, `not null`)

**Carga inicial de categorias (proposta, a validar pela Sabrina com a regulação vigente):**

| Profissão | Categorias (`codigo`) |
|---|---|
| Controlador | Proficiência linguística ICAO — EPLIS (`proficiencia_eplis`); CMA — 3ª classe (`cma_3`); Certificado de Habilitação Técnica — CHT (`cht_atco`); Habilitação de órgão operacional — TWR, APP, ACC (`habilitacao_orgao`); Curso ou estágio de atualização (`atualizacao_atco`); TRM — gerenciamento de recursos de equipe (`trm`) |
| Piloto | Proficiência linguística — ANAC (`proficiencia_anac`); CMA — 1ª ou 2ª classe (`cma_piloto`); Habilitação de classe (`habilitacao_classe`); Habilitação de tipo (`habilitacao_tipo`); Habilitação IFR (`habilitacao_ifr`); Treinamento periódico em simulador (`treinamento_simulador`); CRM — gerenciamento de recursos de tripulação (`crm_piloto`) |
| Comissário | CMA — 2ª classe (`cma_2`); Habilitação ou licença de comissário (`licenca_comissario`); Treinamento periódico de emergência e sobrevivência (`treinamento_emergencia`); Primeiros socorros (`primeiros_socorros`); CRM — gerenciamento de recursos de tripulação (`crm_comissario`) |
| Todas | Outro (`outro`), com nome livre; é a única categoria comum, oferecida para qualquer profissão |

As categorias são apenas rótulos para organizar os registros. A aplicação não associa a elas nenhuma regra de prazo.

**certificados**
- `id` (PK, uuid)
- `usuario_id` (`not null`, FK para `auth.users`, exclusão em cascata)
- `categoria_id` (`not null`, FK para `categorias`)
- `nome_livre` (até 80)
- `numero` (até 40)
- `orgao_emissor` (até 80)
- `data_emissao` (`date`)
- `data_validade` (`date`, `not null`)
- `observacoes` (até 500)
- `criado_em`, `atualizado_em` (`not null`)

**Restrições:**
- `check (data_emissao is null or data_emissao <= data_validade)`;
- limites de tamanho por `check` ou `varchar(n)`;
- índice em `(usuario_id, data_validade)`;
- **gatilho de integridade** (`before insert or update` em `certificados`), porque `check` não consulta outra tabela. Ele recusa a gravação quando:
  - a categoria é `outro` e `nome_livre` está vazio;
  - a categoria não é `outro` e `nome_livre` está preenchido;
  - a categoria está inativa num `insert`, ou num `update` que troca a categoria.

  O servidor faz as mesmas validações antes, para devolver uma mensagem clara (400). O gatilho garante a regra mesmo numa escrita feita diretamente pela API de dados do Supabase, com a sessão do usuário, sem passar pelos Route Handlers.

**Categoria desativada:**
- **cadastro ou troca de categoria:** exige categoria ativa;
- **edição sem troca de categoria:** a categoria atual pode ser mantida, mesmo desativada, o que permite atualizar a validade de um registro antigo;
- os registros antigos continuam exibindo a categoria normalmente.

**Isolamento (RLS em `certificados`):**
- `select` e `delete`: `using (usuario_id = auth.uid())`;
- `insert`: `with check (usuario_id = auth.uid())`;
- `update`: `using (usuario_id = auth.uid()) with check (usuario_id = auth.uid())`, para que ninguém transfira um registro para outro usuário;
- `categorias`: leitura para usuários autenticados; nenhuma escrita pelo cliente.

**Teste de isolamento obrigatório.** Com duas contas, A e B, a conta B não pode listar, consultar, editar, excluir nem contar no painel os registros de A (respostas 404 nas rotas por `:id`, e lista e totais sem os registros de A).

**Testes de integridade obrigatórios:**
1. Editar a validade de um registro mantendo uma categoria desativada: aceito.
2. Escolher uma categoria desativada num novo registro: recusado.
3. Trocar a categoria de um registro para uma desativada: recusado.
4. Escrita direta pela API de dados do Supabase, com sessão válida e dados inválidos ("Outro" sem nome livre; nome livre em outra categoria; categoria inativa): recusada pelo gatilho.

**Relacionamentos:** um usuário tem N certificados; uma categoria tem N certificados.

## 9. Ciclo de vida e organização

**Ciclo de vida: incremental.** Cada incremento entrega uma parte utilizável de ponta a ponta (banco, API e tela), testada e revisada antes do próximo. Isso permite mostrar o sistema funcionando desde o primeiro incremento e corrigir o rumo cedo.

| Incremento | Entrega utilizável | Testes no incremento |
|---|---|---|
| 0. Planejamento | Etapa 1: requisitos, arquitetura, repositório criado | — |
| 1. Acesso + cadastro e lista | Criar conta, entrar e sair; categorias; RF01 e RF02 | Validações do RF01; gatilho com escrita direta; sem sessão = 401; isolamento entre duas contas na lista |
| 2. Detalhes, edição e exclusão | RF03, RF04 e RF05 | 404 em registro de outro usuário; confirmação de exclusão; edição com categoria desativada |
| 3. Painel e acabamento | RF06; responsividade; publicação na Vercel | Limites de data (ontem, hoje, hoje + 30, hoje + 31); totais isolados por usuário; teste em celular |
| 4. Demonstração | Vídeo de até 3 minutos, com as histórias de usuário | Roteiro executado com dados fictícios |

**Responsabilidades (proposta, a ajustar com o grupo).** A execução técnica (banco de dados, serviço REST, acesso e front-end) fica com a Sabrina, por um revezamento combinado entre os trabalhos da disciplina. Cada colega assume uma tarefa de apoio que pode ser feita a distância e deixa registro no repositório (*commit*, *pull request* ou *issue*):

| Integrante | Tarefa | Registro no repositório |
|---|---|---|
| Sabrina Deccache Pacheco | Coordenação; desenvolvimento do banco, da API, do acesso e das telas; publicação na Vercel; atendimento aos pedidos de exclusão de conta | Código de todos os incrementos |
| Caique de Come Rodrigues | Revisão do modelo de dados e da carga de categorias | Revisão do *pull request* do banco |
| Carolina Ariadne de Sousa Cubas Reis | Revisão de texto e de normas (português, itálico em termos estrangeiros, referências) dos slides e do README | *Commits* no README e na pasta `docs/` |
| Endryo William Aprato Pires | Documentação das rotas REST, com exemplos de requisição e resposta | `docs/api.md` |
| Esther Marques Silva | Roteiro e execução dos testes de isolamento (duas contas) e dos limites de data | `docs/testes.md` e *issues* com os resultados |
| Marcos Henrique da Silva Xavier | Testes de responsividade em celulares (iOS e Android) e em navegadores | *Issues* com capturas de tela |
| Reinaldo Acioli de Amorim Oshiro | Roteiro dos testes de aceitação a partir das histórias de usuário | `docs/aceitacao.md` |
| Rodrigo Arroyo Nepomuceno | Roteiro, gravação e edição do vídeo de demonstração (até 3 minutos) | Roteiro em `docs/video.md` e link do vídeo |

**Pendência:** confirmar essas tarefas com cada integrante antes da entrega. Se alguém não puder assumir a sua, a tabela é atualizada para refletir apenas o que foi de fato feito.

**Revisão.** A qualidade é garantida por:
- testes automatizados em cada incremento (isolamento, datas e integridade);
- uma lista de conferência aplicada a cada *pull request*;
- a revisão do Caique no *pull request* do banco;
- os testes manuais da Esther (isolamento e datas) e do Marcos (celulares) ao fim de cada incremento.

**Regra do repositório.** Cada incremento é desenvolvido num *branch* próprio e integrado ao `main` por *pull request*, para que a evolução do projeto fique registrada.

## 10. Justificativa de nuvem

Propõe-se utilizar a Vercel, para a aplicação e o serviço REST, e o Supabase, para o banco de dados e a autenticação, a fim de reduzir a administração de infraestrutura: não há servidor próprio para instalar, atualizar ou manter. A publicação acontece automaticamente a cada atualização do código no repositório, e a aplicação fica acessível de qualquer lugar, inclusive pelo celular. O projeto começa dentro dos limites dos planos gratuitos. Custos, capacidade e mecanismos de cópia de segurança serão verificados antes de qualquer publicação para uso real.

## 11. Decisões em aberto

- Prazo da Etapa 1.
- Confirmação com o professor de que a divisão (execução técnica por uma integrante e tarefas de apoio para os demais) atende à "participação identificável".
- Aprovação da lista de categorias por profissão (Sabrina).
- Confirmação das tarefas de apoio com cada integrante.

## 12. Revisão técnica do planejamento

| Achado | Tratamento | Onde |
|---|---|---|
| P01 Acesso sem fluxo | Cadastro público com e-mail e senha; telas Criar conta, Entrar e Sair; condição transversal, não RF; responsáveis definidos | 3.1, 6, 9 |
| P02 Origem da profissão | Profissão escolhida no formulário só como filtro; `/api/categorias?profissao=` | 2, 3.1, 7 |
| P03 Responsabilidades provisórias | Execução técnica pela Sabrina (revezamento combinado no grupo) e uma tarefa de apoio identificável para cada colega, com registro no repositório. Pendente a confirmação de cada integrante e do professor | 9, 11 |
| P04 Nuvem | Justificativa reescrita de forma condicional | 10 |
| P05 GitHub | Divergência registrada; link real quando o repositório existir; vídeo e relatórios não entram na Etapa 1 | 1 |
| P06 Isolamento | `usuario_id` vem da sessão; RLS com `using` e `with check`; sem chave de serviço; teste com duas contas | 6, 7, 8 |
| P07 Privacidade | Finalidade, dados coletados, aviso nas observações, exclusão da conta por solicitação, dados fictícios | 4 |
| P08 Referência de "hoje" | Tipo `date`, fuso `America/Sao_Paulo`, cálculo único no servidor, quatro casos de teste | 3.1 |
| P09 "Outro" e campos | `codigo` estável, limites concretos, `not null`, categoria desativada | 3.1, 8 |
| P10 Comunicação com o Supabase | "Cliente Supabase (API de dados)"; fluxos de autenticação e de dados separados | 6 |
| P11 Ciclo incremental | Incrementos de ponta a ponta, com testes em cada um | 9 |
| P12 Categoria desativada na edição | Regra separada para cadastro ou troca (exige ativa) e para edição sem troca (aceita a atual); testes 1 a 3 | 3.1, 8 |
| P13 Regra do "Outro" só na aplicação | Gatilho `before insert or update` com as regras do "Outro" e da categoria ativa; teste de escrita direta | 8, 9 |
| P14 Exclusão sem verificação de identidade | Confirmação pelo e-mail cadastrado na conta, responsável definida, exclusão manual com aviso ao titular | 4, 9 |
