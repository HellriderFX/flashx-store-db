# FLASHX Loja — HTML + Cloudflare Pages + D1 (sem Neon)

Versão sem banco externo. A aplicação usa o Cloudflare D1 diretamente pelas Pages Functions. O projeto contém `index.html`, `products.json`, `build-pages.sh`, `_routes.json`, `wrangler.toml` e instruções na raiz. `functions/` também fica na raiz. O comando `sh build-pages.sh` cria `public/` copiando os arquivos estáticos da raiz; não depende de npm nem de um arquivo de script Node.

## O que está incluído
- Loja HTML completa com catálogo de 88 aplicativos.
- Pages Functions para catálogo, logo e ícones, login Admin, pedidos, status do cliente e edição administrativa.
- Cloudflare D1 como banco; não requer Neon, Postgres, Express ou driver de banco externo.
- Login Admin validado por `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_KEYWORD` e `JWT_SECRET` (secrets).
- Pedidos compartilhados no D1. Mensagens de status abrem WhatsApp para revisão/envio manual.

## Instalação
1. No Cloudflare **Workers & Pages → D1 SQL Database**, crie uma base de dados chamada, por exemplo, `flashx-store-db`; anote o nome/ID.
2. Na página da D1, abra **Console**/SQL e execute o arquivo `d1-setup.sql` completo. Ele cria tabelas e insere os 88 produtos sem substituir dados de produtos que já existirem.
3. Envie todos os arquivos deste ZIP para a raiz do GitHub. Não coloque dentro de uma pasta extra. A raiz deve mostrar `index.html`, `functions/`, `products.json`, `_routes.json`, `wrangler.toml` e `d1-setup.sql`.
4. Cloudflare **Workers & Pages → Create application → Pages → Connect to Git**; escolha o repo.
5. Build settings: preset `None`; deixe o campo **Build command vazio**; output directory `public`; root directory `/` ou vazio. A pasta `public/` já vem no ZIP com os arquivos estáticos; não é necessário rodar Node, npm ou script de build.
6. No projeto Pages, vá a **Settings → Bindings → Add → D1 database binding**. Em **Variable name**, escreva exatamente `FLASHX_DB`; em **D1 database**, escolha a base criada. Faça isso em Production; configure Preview também se usar preview deployments. Salve.
7. Adicione secrets em **Settings → Variables and Secrets** (marque Encrypt): `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_KEYWORD`, `JWT_SECRET` com pelo menos 32 caracteres. Faça novo deploy depois de criar bindings/secrets.
8. Teste `https://SEU-PROJETO.pages.dev/api/health`. Deve responder `{"ok":true,"database":"connected","provider":"Cloudflare D1"}`.

## Alternativa para executar o SQL pelo terminal
Depois de instalar e autenticar o Wrangler, execute na raiz do projeto:
`npx wrangler d1 execute flashx-store-db --remote --file=d1-setup.sql`
Use o nome exato que você deu à base. Este comando altera a base remota, então use apenas no banco correto.

## Importante
- A API usa binding D1 `FLASHX_DB`, não `DATABASE_URL`. Remova/ignore `DATABASE_URL` dos antigos secrets se já havia configurado; essa versão não usa Neon.
- O cadastro pessoal e a lista local de pedidos ficam no navegador. UUID do pedido é guardado naquele aparelho para consultar estado usando o telefone da compra; não é login de cliente entre aparelhos.
- O Admin atualiza o pedido no D1 e abre uma mensagem de WhatsApp pré-preenchida. O envio final é manual.
- PIX e confirmação de pagamento continuam manuais. Revise catálogo e preços antes de vender.
- `d1-setup.sql` usa `INSERT OR IGNORE`: se importar uma base já populada, não sobrescreve preços/configurações de apps existentes.
