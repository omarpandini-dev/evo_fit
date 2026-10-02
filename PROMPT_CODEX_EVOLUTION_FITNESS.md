# Prompt para implementar o sistema Evolution Fitness

Cole o texto abaixo no Codex **dentro do repositório do projeto**, depois de adicionar o mockup HTML como referência visual (por exemplo, `mockup-evolution.html` na raiz).

---

Você é responsável por implementar um MVP funcional, de ponta a ponta, para a operação comercial e de estoque de uma empresa que vende aparelhos de musculação. Trabalhe no repositório aberto. Leia primeiro as instruções locais (`AGENTS.md`, se houver) e o mockup HTML que colocarei no projeto. Use o mockup como referência de identidade visual, navegação e telas; transforme-o em uma aplicação React conectada a uma API real, sem simplesmente embutir o HTML da demonstração. Se o nome ou caminho do mockup for diferente, localize-o no repositório.

## Decisões já tomadas

- Stack: PostgreSQL, API Node.js com TypeScript, frontend React com TypeScript. Estruture uma API REST estável que permita no futuro criar um aplicativo Flutter.
- Frontend e API no **mesmo repositório**, em **serviços Docker separados e portas diferentes**: API na porta interna `3000`; frontend em produção na porta interna `8080`. O servidor de desenvolvimento React pode usar `5173`.
- Implantação prevista: VPS Hostinger, usando Easypanel. O PostgreSQL será criado por mim como serviço dedicado na VPS. O Compose de produção deve subir somente API e frontend, conectando a API ao banco externo por variáveis de ambiente; não inclua um PostgreSQL de produção dentro desse Compose.
- Escopo funcional atual: CRM comercial, clientes/projetos, catálogo, propostas, pedidos e estoque. Suprimentos e entregas podem mostrar o acompanhamento dos próprios pedidos conforme os dados registrados; não finja integrações externas ou automações que não existem.
- Neste momento, **somente vendedores acessam o sistema**. Cada vendedor vê, pesquisa, altera e exporta **apenas os próprios clientes e os dados vinculados a eles** (projetos, oportunidades, propostas, pedidos, acompanhamento e indicadores). O catálogo e o estoque são compartilhados. **Todos os vendedores autenticados podem consultar e fazer a manutenção do estoque**. Não há tela administrativa nem acesso de vendedor a carteiras alheias.
- Interface em português do Brasil; valores em BRL e datas adequadas ao Brasil. Não inclua cobrança, pagamentos, aplicativo Flutter ou integração com loja virtual nesta entrega.

## Entregáveis do repositório

Crie uma estrutura legível, por exemplo `api/`, `web/`, `db/` ou `scripts/`, `compose.yaml`, `compose.test.yaml` (se útil para PostgreSQL de teste local), Dockerfiles, `.dockerignore`, `.gitignore`, `.env.example`, **`.env_teste` versionado com valores de exemplo não secretos**, e `README.md`. Pode escolher bibliotecas maduras para Node/React e migrações, explicando escolhas importantes no README. Não deixe rotas fictícias, dados codificados na UI, botões sem efeito para operações prometidas ou arquivos com apenas pseudocódigo.

### Banco de dados e inicialização

1. Implemente migrações **versionadas e reproduzíveis** que criem toda a estrutura PostgreSQL: usuários vendedores; clientes e contatos; projetos/oportunidades; produtos/catálogo; propostas, seus itens e versões ou histórico; pedidos e itens; saldos e reservas de estoque; movimentos de estoque auditáveis; e acompanhamento de suprimentos/entregas quando usados nas telas. Inclua chaves estrangeiras, restrições, índices pertinentes, timestamps e identificação do vendedor responsável.
2. Crie um comando claro, como `npm run db:init`, que execute as migrações no banco configurado, sem eliminar dados existentes. Rodar esse comando novamente deve ser seguro. Migração de produção e carga de demonstração são comandos **separados**.
3. Crie `npm run db:seed:test` com dados determinísticos e repetíveis: pelo menos dois vendedores; clientes distintos em cada carteira; projetos e oportunidades; produtos de exemplo; propostas em diferentes estados; pedidos; saldos, reservas e movimentos; além de acompanhamentos de suprimento/entrega que permitam explorar as telas. Documente logins e senhas **apenas de demonstração** no README, com hashes de senha no banco. Dados de exemplo são fictícios e não devem ser apresentados como cadastro ou catálogo oficial da empresa.
4. Proteja `db:seed:test` e qualquer comando de reset: exija `APP_ENV=test`, `ALLOW_DEMO_SEED=true` e uma verificação adicional do banco de destino para impedir execução acidental no banco de produção. O seed deve ser idempotente. Nunca execute o seed automaticamente no startup de produção. Se criar comando de reset, faça-o funcionar somente em banco descartável de teste e documente seu caráter destrutivo.
5. Em `.env_teste`, inclua um exemplo completo de configuração de banco dedicado, com placeholders como `PGHOST=postgres-evolution`, `PGPORT=5432`, `PGDATABASE=evolution_fitness_teste`, `PGUSER=evolution_app`, `PGPASSWORD=SUBSTITUIR_POR_SENHA_FORTE`, `PGSSLMODE=prefer`, `APP_ENV=test`, `ALLOW_DEMO_SEED=true`, além das variáveis necessárias de autenticação, URL pública da API, CORS e portas. Gere também `.env.example` para produção, sem senhas reais. Mantenha arquivos locais com segredos reais fora do Git. Se usar `DATABASE_URL` em vez dos campos `PG*`, forneça o exemplo completo equivalente e explique a codificação da senha na URL.

### Catálogo real da Evolution Fitness e imagens

Além do seed fictício de teste, implemente uma **importação opcional, reproduzível e idempotente** do catálogo público da Evolution Fitness. Examine `https://evolutionfitness.com.br/` e, especialmente, as páginas de categorias e produtos em `https://loja.evolutionfitness.com.br/`. A loja apresenta linhas como Residencial, Comercial, Evolution Pro e Acessórios, com páginas individuais contendo descrições, preços, variações e, em alguns casos, ficha técnica e galeria de imagens. Não trate o catálogo fictício do mockup como catálogo oficial.

- Crie um comando separado, por exemplo `npm run catalog:import:evolution`, que percorra todas as categorias e páginas de listagem acessíveis (inclusive paginação/carregamento adicional), visite cada produto e importe **todos os produtos e campos publicamente disponíveis que puder extrair com confiança**. Inclua nome, código/SKU ou modelo quando houver, marca, linha/categoria/subcategoria, descrição, destaques, especificações técnicas e dimensões, peso, capacidade, voltagem e outras variantes, preço público atual/preço anterior quando publicados, URL original, links de ficha técnica e galeria de imagens. Guarde campos variáveis em uma estrutura extensível sem perder os campos essenciais consultáveis. Não invente dados ausentes: registre `null`/ausente e relate lacunas.
- Modele imagens em uma tabela/estrutura própria com URL original, posição, texto alternativo e indicação da imagem principal; mostre a imagem principal na listagem e a galeria no detalhe, com fallback visual quando indisponível. Se for possível e permitido utilizar as imagens no projeto, faça download validado para armazenamento persistente (volume Docker ou armazenamento configurável), com limite de tamanho, formato e tratamento de falhas; preserve também a URL de origem. Se não for possível baixar, armazene a URL pública e explique a dependência externa. Não use imagem genérica apresentada como fotografia real do produto.
- Distinga **preço público de referência** da loja do preço usado em uma proposta comercial; o vendedor pode definir o valor da proposta sem que uma nova importação altere propostas ou pedidos já emitidos. A disponibilidade exibida na loja **não é o saldo do estoque interno**: o importador nunca deve criar entrada, saída, reserva ou saldo com base no site.
- Identifique os produtos pela URL/código de origem e faça atualização controlada: relatório de criados, atualizados, ignorados e erros; data da última coleta; opção `--dry-run`; sem duplicação e sem apagar produtos locais ou sobrescrever alterações internas de forma silenciosa. Não rode coleta na inicialização da API ou durante o deploy. Trate falhas de rede, bloqueios e páginas alteradas, com ritmo de requisições moderado. Preveja correção manual de dados importados por comando operacional documentado; não crie uma tela administrativa para vendedores.
- Se o ambiente do Codex permitir acesso ao site, execute o importador em **banco de teste** e confira alguns registros e imagens contra as páginas de origem. Se não permitir, entregue o importador executável, testes com páginas capturadas/fixtures verificáveis e um formato alternativo de importação por JSON/CSV para carga autorizada; informe no resultado quantos produtos reais foram efetivamente importados, sem declarar uma carga completa que não ocorreu.

### Autenticação e isolamento de dados

1. Implemente login, logout e consulta da sessão/usuário atual. Faça hash seguro de senhas; armazene segredo de autenticação em variável de ambiente; valide autenticação e autorização **na API**. Crie um comando operacional documentado para cadastrar/ativar/desativar vendedores sem exigir uma interface administrativa neste MVP.
2. Derive a identidade do vendedor da sessão, nunca de `sellerId` enviado pelo frontend. Aplique o filtro de propriedade em consultas, busca, detalhe, mutações, relatórios e exportações. Ao tentar acessar o ID de um recurso de outra carteira, não revele seus dados. Verifique também vínculos aninhados, como item de proposta, pedido e acompanhamento.
3. Considere que estoque e catálogo são globais. Qualquer vendedor autenticado pode ler o estoque e registrar **entrada, saída e ajuste**, sempre com produto, quantidade, justificativa, data e autor. Os movimentos devem ser auditáveis; não permita editar ou apagar o histórico para reescrever saldos. Faça a alteração de saldo e o registro do movimento na mesma transação. Evite saldo negativo e retirada que consuma quantidades já reservadas; trate concorrência entre dois vendedores. Defina regras claras para reserva na conversão de proposta em pedido e liberação/cancelamento, de modo que o saldo disponível seja coerente.
4. Não exponha segredos nas respostas, logs, bundle do navegador ou documentação. Valide entradas e trate erros de forma consistente. Configure CORS para a origem permitida e escute `0.0.0.0` no contêiner.

### API e fluxos funcionais

- Endpoints REST com validação, paginação/filtros e respostas consistentes para clientes/contatos, projetos/oportunidades, catálogo, propostas/itens, pedidos, estoque/movimentos, acompanhamento e dashboard/relatórios.
- O vendedor pode criar e editar clientes da própria carteira, registrar oportunidade/projeto, montar proposta com produtos e valores, revisar seu status e converter proposta aceita em pedido sem duplicação acidental. Modele estados e transições permitidas. Como só há vendedores, o aceite da proposta pode ser registrado manualmente pelo vendedor, com data e histórico; não pressuponha aprovação por gerente.
- O pedido deve ter vínculo verificável com cliente, vendedor, proposta e itens. Suprimento e entrega devem apresentar status reais gravados no banco, com atualização manual por comando operacional ou interface apropriada somente se a regra de acesso for segura e estiver implementada. Não simule transportadora, ERP ou fábrica.
- Dashboard e relatórios usam dados reais filtrados por vendedor, exceto indicadores agregados explicitamente de estoque compartilhado. Adicione `/health` para verificação do serviço. Documente a API (OpenAPI/Swagger ou documentação equivalente).

### Frontend

- Recrie as telas pertinentes do mockup em React, com layout responsivo: login, painel, CRM, clientes/projetos, catálogo, propostas, pedidos, estoque, acompanhamento de suprimentos/entregas e relatórios pertinentes ao escopo.
- Conecte formulários, listas, detalhes, filtros e ações à API real. Trate carregamento, erro, validação e listas vazias. Mostre o vendedor atual e faça logout.
- No catálogo, permita pesquisar e filtrar produtos reais importados por linha/categoria/modelo, abrir os detalhes técnicos, ver a foto principal e navegar pelas demais fotos. Mostre a origem e a data do preço público de referência quando presentes.
- A tela de estoque é compartilhada entre vendedores e oferece formulário funcional para entrada, saída e ajuste, incluindo justificativa e histórico com autor. Mostre saldo físico, reservado e disponível conforme a regra implementada.
- Não entregue apenas um mockup interativo com estado em memória. Após atualizar a página, os dados devem persistir no PostgreSQL.

### Docker, teste local e Easypanel

- Crie builds Docker reproduzíveis, preferencialmente em múltiplos estágios, para API e frontend. Use configuração do servidor web que permita recarregar rotas do React. Inclua healthchecks úteis. Não embuta segredos na imagem.
- `compose.yaml` de produção: `api` e `web`, com portas internas distintas `3000`/`8080`, configuração por ambiente e conexão ao PostgreSQL dedicado da VPS. Não publique PostgreSQL. Evite credenciais fixas no Compose. Se publicar portas do host para uso local, torne-as configuráveis e explique como encaminhar os serviços no Easypanel.
- Ofereça um caminho simples para teste local com PostgreSQL descartável e isolado (por exemplo `compose.test.yaml`), sem confundir esse banco com o serviço de produção. Documente a ordem exata: configurar variáveis, iniciar banco de teste, migrar, popular dados, iniciar API e frontend, executar testes.
- No README, descreva como criar o serviço PostgreSQL dedicado na VPS, obter host/porta/usuário/banco, fornecer as variáveis à API no Easypanel, implantar o Compose com os dois serviços, configurar domínio/roteamento para frontend `8080` e API `3000`, usar HTTPS, ajustar URL pública da API e origem CORS, executar migrações com segurança e verificar `/health`. Explique que `.env_teste` é exemplo e não deve ser aplicado diretamente à produção. O caminho interno de conexão do PostgreSQL depende da rede/serviço configurado no Easypanel: use placeholders e mostre como substituí-los, sem presumir hostname real.
- Explique estratégia básica de atualização de versão, migrações e backup do PostgreSQL. Não afirme que implantou na VPS: entregue arquivos e instruções, sem usar credenciais nem fazer deploy.
- Documente a importação opcional do catálogo (pré-requisitos, comando, execução de teste, `--dry-run`, fontes, imagens, volume persistente, relatório de erros e procedimento de atualização). Esclareça que o conteúdo público da loja pode mudar e que preço público e disponibilidade online não substituem preço negociado nem estoque interno.

### Testes e critérios de conclusão

Crie testes significativos de API/regra de negócio, especialmente:

1. Vendedor A vê sua carteira e não consegue listar, buscar por ID, editar, exportar ou obter indicadores da carteira B; o inverso também vale.
2. Ambos conseguem consultar e movimentar o mesmo estoque; movimento registra autor e motivo, persiste e atualiza saldos.
3. Saída maior que disponibilidade, saída de unidades reservadas e operações concorrentes não deixam o estoque inconsistente.
4. Proposta aceita vira pedido com itens e reservas corretos; repetição não gera dois pedidos; cancelamento libera reservas conforme regra.
5. Migrações e seed de teste podem rodar novamente sem duplicar cadastros; seed de teste falha quando as proteções ambientais não forem atendidas.
6. Importar o mesmo catálogo duas vezes não duplica produtos, variantes ou imagens; falha em uma página não corrompe as demais; preços importados não alteram propostas/pedidos históricos nem o saldo físico do estoque.

Execute os testes, build de API e frontend e validação de `docker compose config` quando o ambiente permitir. Corrija as falhas encontradas. Se algum comando não puder ser executado por falta de Docker ou serviço PostgreSQL nesta sessão, identifique exatamente a limitação e deixe o comando reproduzível no README. No final, informe o que foi implementado, os comandos executados e seus resultados, e qualquer limitação real.

## README obrigatório

O README deve permitir a outra pessoa partir de um clone limpo e repetir: configuração de `.env_teste` sem credenciais reais; subida do PostgreSQL de teste; migrações; seed; credenciais de demonstração; execução de API e frontend com suas URLs/portas; testes; builds Docker; uso diário do sistema; escopo dos perfis e isolamento; manutenção de estoque; e implantação/atualização no Easypanel com PostgreSQL dedicado. Liste todas as variáveis com exemplos seguros e distinga claramente ambiente de teste e produção.

Comece examinando o repositório e o mockup, tome decisões técnicas rotineiras e implemente o MVP inteiro. Se houver incompatibilidade concreta entre mockup e estas regras, estas regras prevalecem e a diferença deve ser registrada no README.
