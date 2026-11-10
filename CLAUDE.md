# Em Dia

Controle de Certificações para Profissionais da Aviação. Projeto Integrador IV do curso de
ADS do Centro Universitário Senac EAD ("Desenvolvimento de sistemas orientado a dispositivos
móveis e baseados na web"), Grupo 15. Coordenação e desenvolvimento: Sabrina Deccache Pacheco.

**Este projeto é separado da OnFreq.** Não ler, citar nem alterar nada da OnFreq
(`~/Desktop/Projeto Plataforma/Project`) a partir daqui, e não usar o Supabase dela
(`eplis-plataform`). O Em Dia terá o seu próprio projeto no Supabase e na Vercel. A ideia é
que, no futuro, ele vire parte de uma empresa de tecnologia educacional ATC da Sabrina.

Responder sempre em português do Brasil.

## Onde está cada coisa

- `docs/planejamento.md` — planejamento completo e aprovado (requisitos, histórias, premissas,
  privacidade, arquitetura, REST, modelo de dados, categorias, tarefas). **É a fonte das
  decisões: não reabrir sem motivo novo.**
- `docs/etapa1-slides/` — fontes da apresentação da Etapa 1 (tipo Slides do claude.ai,
  1920×1080, estilos inline; `deck.json` define a ordem).
- `README.md` — resumo público para o professor e o grupo.

## Decisões fechadas (resumo)

- Stack: Next.js (App Router) + TypeScript + Tailwind; Route Handlers como serviço REST;
  Supabase (PostgreSQL, Auth, RLS com `using` e `with check`); Vercel; GitHub com branch + PR.
- 7 rotas: `GET/POST /api/certificados`, `GET/PUT/DELETE /api/certificados/:id`,
  `GET /api/painel`, `GET /api/categorias?profissao=`. JSON; 401 sem sessão; 404 para registro
  inexistente ou de outro usuário; 400 para dados inválidos.
- `usuario_id` sempre vem da sessão, nunca do navegador. As rotas usam o cliente Supabase com a
  sessão do usuário, nunca a chave de serviço.
- Situação calculada no servidor, nunca gravada. Fuso `America/Sao_Paulo`, tipo `date`.
  Vencido: validade < hoje. A vencer: hoje ≤ validade ≤ hoje + 30. Válido: validade > hoje + 30.
  Os 30 dias ficam numa constante só. Testar ontem, hoje, hoje + 30 e hoje + 31.
- Categorias por profissão (controlador, piloto, comissário), cada uma com `codigo` estável e
  único. A única categoria comum é `outro`, que exige `nome_livre`. Um trigger
  `before insert or update` garante a regra do "Outro" e a de categoria ativa. Exceção: uma
  edição que mantém uma categoria já desativada é aceita.
- Guarda apenas os dados do documento, sem arquivos (LGPD), com aviso no campo de observações.
  A exclusão da conta é feita a pedido, com confirmação pelo e-mail cadastrado.
- Ciclo incremental: 0 Planejamento (Etapa 1) → 1 Acesso e cadastro (RF01–RF02) →
  2 Detalhes e edição (RF03–RF05) → 3 Painel e publicação (RF06, responsividade, Vercel) →
  4 Vídeo de até 3 minutos.
- A Sabrina escreve todo o código. Os colegas fazem tarefas de apoio registradas no repositório
  (tabela no README). Sem licença de propósito: os direitos sobre o código ficam com ela.

## Convenções

- Commits em português, com a linha `Co-Authored-By` do Claude.
- `.env.local` nunca vai para o git. `privado/` e `CLAUDE.local.md` também não, porque o
  repositório é público.
