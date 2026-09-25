# Guia de implementação: site em HTML para Payload CMS

Para o agente que vai transformar o site da FlytLab, hoje em HTML puro, em um site Next +
Payload CMS. Este guia junta o que aprendemos colocando no ar o site da Unique By House
(setembro de 2026): o que funcionou, o que quebrou em produção e as regras que nasceram
disso. Leia inteiro antes de começar. Quando este guia e o seu instinto discordarem, siga
o guia e registre a discordância no débito do projeto para o Filipe decidir.

---

## 0. Como trabalhar com o Filipe

- **O Filipe não é programador.** Decisão técnica (stack, biblioteca, estrutura, abordagem)
  é sua: escolha, siga e registre em uma linha o que escolheu e por quê. Pergunte só o que é
  de negócio: aparência, conteúdo, funcionalidade para o visitante, custo, prazo, dados de
  cliente, e qualquer ação irreversível ou pública.
- **Local primeiro.** Tudo é construído e validado na máquina. Commit, GitHub, publicação e
  qualquer acesso a servidor só quando o Filipe pedir, com todas as letras. Chave SSH na
  máquina não é autorização.
- **Sem travessão** (o traço longo) em nada que o Filipe ou o cliente vá ler. Use vírgula,
  dois pontos ou ponto.
- **Arquivo temporário vai para `F:\temp\AAAA-MM-DD\<assunto>\`.** Nunca no C:, nunca na
  pasta do projeto.
- **Toda pendência vai para o débito do projeto** (`.planning/DEBITO.md`): técnica, de
  conteúdo, de conta, de custo ou de terceiro. O que não está no débito some.
- **Pedir teste ao Filipe sempre com o endereço da tela junto.**
- **Relatório honesto.** Teste que falhou é falha, passo pulado é passo pulado. "Passou nos
  testes" não prova que a tela é boa de usar: o que o visitante ou o cliente vai tocar, o
  Filipe valida com os próprios olhos.
- **Segredo nunca no chat nem no Git.** Segredo do cliente fica em
  `C:\Users\nucle\.claude\clientes\<cliente>.env`, um arquivo por cliente. Em script, leia
  de lá sem imprimir.

---

## 1. Referências (leia quando o guia mandar)

Todas estão nesta máquina.

| O quê | Onde |
|---|---|
| Site da Unique, a implementação de referência | `\\wsl.localhost\Ubuntu-22.04\home\filipe\projects\site-unique-house` |
| Regras do projeto da Unique | `AGENTS.md` na raiz dele |
| Publicação, migrations e VPS | `.planning/DEPLOY.md` dele (arquivo local, fora do Git) |
| Débito resolvido e aberto (armadilhas que custaram horas) | `.planning/DEBITO.md` dele |
| Auditoria de segurança e correções | `docs/security-audit/` dele |
| Atribuição de origem do lead (UTM) | `docs/RASTREIO-UTM.md` dele |
| Módulo de tracking e consentimento | `F:\github-projects\tracking-payload-module` (o `README.md` tem a instalação completa) |
| Verificação de tracking pela rede | `F:\github-projects\tracking-payload-module\docs\verificacao-de-rede.md` |
| Plugin de e-mail pelo painel | `F:\github-projects\payload-smtp-panel` (README e `.planning/DEBITO.md`) |

Use a Unique como exemplo de COMO fazer, nunca como fonte de conteúdo, nome, cor ou
identificador. Nada da Unique entra neste site: nem texto, nem ID de pixel, nem e-mail.

---

## 2. Stack e decisões fixas

Mesma stack da Unique, que já está provada em produção:

- **Payload 3** com **Next 15** (App Router), TypeScript, **pnpm**, **Postgres** pelo
  `@payloadcms/db-postgres`.
- **Desenvolvimento no WSL** (`Ubuntu-22.04`), em `~/projects/flytlab-site`. Docker e pnpm
  rodam lá; o Windows não tem Docker. A pasta `F:\github-projects\flytlab-site` guarda o
  HTML de origem e este guia. Se preferir outra organização, decida e registre o motivo.
- **Fotos no disco** (pasta `media/` em dev, volume Docker em produção). Sem serviço
  externo de storage, a menos que o Filipe peça.
- **Schema por migrations**, com o `push` do adaptador desligado desde o primeiro dia
  (seção 6).
- **Build nunca na VPS.** Imagem Docker gerada no GitHub Actions, publicada no GHCR, e a
  VPS só baixa e sobe. Um build na VPS da Unique saturou a máquina e derrubou os serviços
  por 32 minutos.
- **Padrão de instruções:** `AGENTS.md` na raiz é a fonte real; `CLAUDE.md` contém apenas
  a linha `@AGENTS.md`.
- **Arquivos locais fora do Git:** `.planning/DEBITO.md` e `.planning/HANDOFF.md` nunca
  entram em commit. Coloque `.planning/` no `.gitignore` antes do primeiro commit.

**Portas locais.** Já estão ocupadas por outros projetos: 3000 e 5432 (Unique), 3100 e
5433 (Pollus), 3300 (CRM), 5434 (módulo de tracking). Escolha portas livres, confira com
`ss -ltn` no WSL e registre no `AGENTS.md`. Quando o Windows acessar o WSL, use
`127.0.0.1`, nunca `localhost`: no Windows o `localhost` resolve para IPv6 e não chega ao
servidor.

---

## 3. Fase 0: inventário do HTML (antes de qualquer código)

Produza `.planning/INVENTARIO.md` com:

1. **Mapa de páginas:** cada arquivo HTML, a URL que ele tem hoje e a URL que vai ter no
   Next. **Toda URL que mudar precisa de redirect 301** (seção 8), senão perde posição no
   Google.
2. **Tipos de conteúdo:** o que se repete (produtos, serviços, projetos, posts, equipe,
   depoimentos) vira **coleção**; o que é único e global (cabeçalho, rodapé, contato, redes
   sociais, SEO padrão, e-mail) vira **global**; blocos de página que o cliente vai querer
   reordenar viram **blocks**.
3. **Formulários:** quais existem, quais campos, para onde devem ir, e se têm anexo.
4. **Imagens:** quantas, peso atual, quais são fundo de tela inteira, quais são miniatura.
5. **SEO atual:** title, description, Open Graph, dados estruturados, textos alternativos,
   `robots.txt` e sitemap, se existirem. Nada disso pode piorar na migração.
6. **Scripts de terceiros:** pixel, GA4, Google Ads, chat, mapas. Todos vão sair do HTML e
   passar pelo módulo de tracking com consentimento (seção 10). Nenhum script de rastreio
   entra solto no layout.
7. **Fontes e cores:** de onde vêm e em que formato.

Mostre o inventário ao Filipe antes da Fase 2. É a hora de ele corrigir o que não viu.

---

## 4. Fase 1: ambiente local

1. Crie o projeto no WSL (template oficial do Payload com Postgres, ou estrutura igual à da
   Unique).
2. Postgres em `docker compose` local, porta livre, volume nomeado. Um banco para o site e,
   quando chegar o tracking, **um banco separado para o consentimento** na mesma instância
   (a Unique cria os dois por `scripts/init-bancos.sql`).
3. `.env` local fora do Git, e `.env.example` versionado só com os NOMES das variáveis. As
   variáveis que a Unique usa servem de lista inicial: `DATABASE_URI`, `PAYLOAD_SECRET`,
   `NEXT_PUBLIC_SERVER_URL`, `SITE_NOINDEX`, `MEDIA_DIR`, `C15T_DATABASE_URI`, `C15T_URL`,
   `TRACKING_WORKER`, `PAYLOAD_TRACKING_ENCRYPTION_KEYS`, `PAYLOAD_TRACKING_ACTIVE_KEY_ID`,
   `PAYLOAD_TRACKING_BROWSER_TOKEN_KEYS`, `PAYLOAD_TRACKING_BROWSER_TOKEN_ACTIVE_KEY_ID`,
   `FORM_LIMITE_ENVIOS`, `FORM_LIMITE_JANELA_MIN`, `FORM_LIMITE_GLOBAL`,
   `FORM_SEGUNDOS_MINIMOS`.
4. `serverURL: process.env.NEXT_PUBLIC_SERVER_URL` declarado no `payload.config.ts`. Sem
   isso o Payload deduz a URL de cada requisição, e atrás de proxy isso manda link de
   e-mail e redirecionamento do `/admin` para o endereço interno do contêiner.
5. `AGENTS.md` do projeto desde o primeiro dia: comandos, portas, estrutura, regras
   numeradas. Use o da Unique como modelo de formato.
6. Crie o usuário administrador local e guarde o login em
   `C:\Users\nucle\.claude\clientes\<cliente>.env`, nunca em arquivo do projeto.

**Pronto quando:** `pnpm dev` sobe, o `/admin` abre, e o Filipe consegue entrar por
`http://127.0.0.1:<porta>/admin`.

---

## 5. Fase 2: modelagem e conteúdo

**Regra de ouro: conteúdo vem do Payload.** Nenhum texto que o cliente possa querer trocar
fica fixo em componente: título, parágrafo, botão, telefone, endereço, link de rede social,
texto de rodapé, texto do banner de consentimento, mensagem de sucesso do formulário.

- **Coleções e globals** conforme o inventário. Campos com rótulo e descrição em
  português, pensando no cliente que vai editar, não no programador.
- **`slug`** em toda coleção que vira página, único, gerado do título e editável.
- **Grupo SEO em toda coleção que vira página** (seção 8) e um global de SEO padrão.
- **Coleção de mídia** com `imageSizes` pensados para o layout real (seção 7) e **texto
  alternativo obrigatório**.
- **Controle de acesso** (seção 11) definido na criação de cada coleção, não depois.
- **Importação do conteúdo do HTML por script** (`scripts/importar-html.ts` ou similar),
  idempotente: rodar duas vezes não duplica. Nada de digitar conteúdo à mão no painel.
- **Backup de conteúdo** com script (`pnpm backup-banco`, veja o da Unique): dump
  higienizado que exclui usuários, sessões, configuração de e-mail, preferências e todas as
  tabelas de tracking. O banco de consentimento nunca entra em dump, porque tem IP de
  visitante. **Por que as tabelas de tracking ficam de fora:** na Unique, um dump de dev
  levado para produção carregou a configuração de tracking de teste, e um pixel que não era
  do cliente disparou por mais de 24 horas com as visitas reais dele. Tracking se configura
  por ambiente, com o script da seção 10, depois de cada restauração.

---

## 6. Migrations (a parte que mais derrubou produção)

- `push: false` no adaptador desde o primeiro dia. Toda mudança de coleção ou global:
  `pnpm payload migrate:create <nome>`, **ler o SQL gerado** (procurar `DROP`, `RENAME`,
  troca de tipo, `SET NOT NULL`) e commitar junto com o código.
- **A linha `dev` (batch -1) em `payload_migrations` derruba a produção.** Rodar `next dev`
  com push contra um banco grava essa linha; em produção o `migrate` abre um prompt sem
  terminal e chama `process.exit(0)`, e o contêiner reinicia em loop. Nunca aponte o dev
  para o banco de produção. Antes de cada publicação:
  `select name, batch from payload_migrations order by created_at desc limit 3`.
- **Coleção com `versions` tem cada campo em duas tabelas** (`x` e `_x_v_version_*`).
  Migração escrita à mão que altera só uma passa no dev e quebra em produção na primeira
  gravação.
- **Todo `select` e o `task_slug` da fila são ENUM do Postgres.** Valor novo exige
  `ALTER TYPE ... ADD VALUE` na migração. Comparar colunas não pega; comparar os valores de
  `pg_enum` pega.
- **Ensaie a migração** numa cópia do banco (dump, banco local, migrate, diff de schema)
  antes de publicar.

---

## 7. Imagens e desempenho

- **WebP** para foto. Qualidade 85 na maioria, 90 para foto de detalhe (textura, tecido,
  close de produto). PNG só para logo com transparência, e mesmo logo costuma ficar muito
  menor em WebP sem perda.
- **O Payload recomprime todo WebP enviado**, com perda leve. Meça o peso pelo arquivo que
  está sendo servido, nunca pelo arquivo que você converteu antes do upload.
- **`imageSizes` pensados para o layout:** miniatura, card, tablet, desktop, e o master só
  para download ou zoom. Na Unique a galeria servia o master sem redimensionar na página
  inteira; trocar para o tamanho "tablet" derrubou o peso sem perda visível.
- **Sirva a variante certa, nunca o original.** A Unique usa `<img>` com a variante do
  Payload escolhida por `srcImagem()` (`src/utilities/imagem.ts`), sempre com `width`,
  `height` e `alt`, `loading="lazy"` abaixo da dobra e prioridade alta na imagem principal.
  Atenção: o Payload só gera variante MENOR que o original. Foto de 1441 px não ganha a
  variante de 1920, e pedir essa variante cai no original pesado. Por isso a função tenta a
  preferida e desce para a próxima que existe.
- **PNG de foto é armadilha:** o Payload gera as variantes no mesmo formato da origem. Fotos
  de loja em PNG de 1,7 MB viraram 100 KB depois de convertidas.
- **Metas de peso** (as mesmas que propusemos na Unique): miniatura até 40 KB, variante de
  tablet até 100 KB, variante de desktop até 200 KB, e o caminho crítico da página (o que
  carrega antes de rolar) até 800 KB. Página acima disso vai para o débito com o número
  medido.
- **Fontes** servidas pelo próprio site (`next/font`), sem pedido a servidor externo.

---

## 8. SEO

Nada do que o HTML tem hoje pode piorar. Conferir item por item contra o inventário.

- **`generateMetadata` em toda rota**, lendo do CMS: title, description, canonical absoluto,
  Open Graph (título, descrição, imagem 1200x630) e Twitter card. Grupo SEO por documento
  com fallback para o global de SEO padrão e, por último, para o próprio conteúdo. Modelo:
  `src/utilities/metadata.ts` e `src/app/(frontend)/layout.tsx` da Unique (template de
  título `%s · Marca`, `metadataBase` com `NEXT_PUBLIC_SERVER_URL`, `locale` `pt_BR`).
- **Description entre 110 e 165 caracteres**, para o Google não cortar.
- **Página com filtro por parâmetro** (`/produtos?categoria=x`) aponta o canonical para a
  página sem filtro, para não virar conteúdo duplicado.
- **`lang="pt-BR"`** no html, um H1 por página, hierarquia de títulos coerente, texto
  alternativo em toda imagem.
- **Dados estruturados (JSON-LD):** `Organization` ou `LocalBusiness` no layout, e o tipo
  certo em cada página (`Product`, `Service`, `Article`, `BreadcrumbList`, `FAQPage`),
  sempre montado do conteúdo do CMS. Validar no Rich Results Test. **Escape os caracteres
  que fecham a tag `<script>`** ao serializar: um nome de loja com `</script>` virou XSS
  armazenado na Unique (achado A2; veja `src/components/site/JsonLd.tsx`).
- **`sitemap.ts`** com todas as rotas públicas e **`lastModified` real**, igual ao da Unique
  (`src/app/sitemap.ts`): páginas fixas usam o maior `updatedAt` do conteúdo que as
  alimenta, rotas sem fonte ficam sem data, e rota que só redireciona fica de fora.
- **`robots.ts`** apontando o sitemap, bloqueando `/admin` e `/api` e os parâmetros de
  campanha (`utm_*`, `gclid`, `fbclid`, `msclkid`), como o da Unique. Robôs de IA liberados,
  salvo pedido contrário do Filipe. No proxy de produção, `/admin` responde com
  `X-Robots-Tag: noindex, nofollow`. **`SITE_NOINDEX=true`** em qualquer prévia pública e
  **vazio em produção**. Confira isso no primeiro dia no ar.
- **Redirects 301** de todas as URLs antigas do HTML para as novas, no `next.config`. Teste
  cada uma. Domínio com `www` redireciona para o sem `www` (ou o contrário, mas um só), no
  proxy, preservando o caminho.
- **Página 404** própria, com navegação de volta.
- **Links internos nunca levam UTM** (seção 10).
- Depois de publicar: enviar o sitemap no Search Console. Isso é tarefa do Filipe; deixe no
  débito.

---

## 9. Formulários e e-mail

**Use o plugin `@hiperbold/payload-smtp-panel`** (`F:\github-projects\payload-smtp-panel`).
É o sistema de e-mail da Unique transformado em pacote: tela de configuração no painel,
segredo cifrado, SMTP tradicional e envio por API (SendKit e Resend), função `sendEmail`
que nunca quebra o formulário, botão de teste e cópia oculta opcional.

- **Instalação por tarball**, não pelo GitHub: a pasta compilada não vai para o
  repositório. No plugin: `pnpm build && pnpm pack`. Copie o `.tgz` para a raiz do site,
  referencie como `file:` no `package.json` e na linha `COPY` do `Dockerfile`, e siga as
  mesmas regras do tarball do tracking (seção 10). Nunca reaproveite o nome de um `.tgz` com
  conteúdo diferente.
- **Este site é o primeiro a usar o plugin de verdade.** Até agora ele só passou em tipos e
  em 28 testes automatizados; ninguém abriu a tela dentro de um admin real. Sua primeira
  tarefa com ele é instalar, abrir o painel e conferir abas, campo mascarado, botão de
  teste e a migração que cria a tabela. Defeito encontrado: corrija no repositório do
  plugin, com teste, gere versão nova e registre no `.planning/DEBITO.md` de lá.
- **`access` do plugin só para administrador.** Quem edita aquela tela escolhe para onde vão
  os leads.
- **Salvar a tela de e-mail sem digitar a senha precisa manter a senha.** Confira no banco
  depois de salvar: esse bug derrubou os formulários da Unique em produção.
- **Hospedagem pode bloquear as portas de SMTP** (25, 465, 587, 2525). A VPS da Unique
  bloqueia. Sintoma: tempo esgotado antes de autenticar. Teste de dentro do servidor com
  `nc -zv -w 5 <host> 465` antes de culpar a senha; bloqueado, use o transporte por API.
- **E-mails do próprio Payload** ("esqueci minha senha" do admin) só saem com
  `useAsPayloadEmailAdapter: true` no plugin. Ligue.
- **Anti-spam em toda rota pública de formulário**, como em `src/utilities/antiSpam.ts` da
  Unique: 5 envios por IP e no máximo 20 e-mails no total a cada 10 minutos, 3 segundos
  mínimos de preenchimento, campo isca, e o IP lido como na linha A1 da seção 11. O limite
  fica em memória e zera ao reiniciar, o que basta para site de uma instância só. O e-mail do
  lead leva a jornada de atribuição (primeiro e último toque, página de origem), como em
  `src/utilities/leadEmail.ts` da Unique. Confira que as variáveis `FORM_*` estão repassadas
  no `docker-compose` de produção; na Unique elas ficaram meses no `.env` sem chegar ao
  contêiner.
- **Nunca envie o formulário real em produção para testar:** cai na caixa do cliente e
  polui lead e conversão. Teste local com o envio desligado no painel (o e-mail vai só para
  o log) ou com `fetch` simulado, e em produção use o botão de teste do painel para um
  endereço seu.

---

## 10. Tracking, consentimento e atribuição

**Use o módulo `@hiperbold/payload-tracking`** (`F:\github-projects\tracking-payload-module`,
versão 1.1.5 ou a mais recente do `CHANGELOG.md`). **Siga o `README.md` do módulo** na
ordem: variáveis de ambiente, plugin no config, backend c15t na rota do próprio site,
provider no layout, banner de consentimento, worker. Use os arquivos da Unique como
exemplo pronto: `src/tracking/opcoes.ts`, `src/tracking/c15t.ts`, `src/tracking/worker.ts`,
`src/tracking/c15tLog.ts`, `src/instrumentation.ts`, `src/instrumentation-node.ts`,
`scripts/configurar-tracking.mjs`.

Regras que custaram caro:

1. **Tarball versionado no repositório do site**, só o da versão em uso. Trocar de versão:
   copiar o `.tgz`, trocar o nome no `package.json` e na linha `COPY` do `Dockerfile`,
   `pnpm install`, e gerar migration se o schema do módulo mudou. **Nunca reaproveitar o
   nome** de um `.tgz` com conteúdo diferente: o pnpm não reextrai e o lockfile guarda a
   integridade. `minimumReleaseAgeExclude: ['@hiperbold/payload-tracking']` no
   `pnpm-workspace.yaml`, senão o pnpm 11 recusa o pacote local.
2. **Opções do tracking em um arquivo só** (`src/tracking/opcoes.ts`). Plugin, worker e c15t
   precisam das mesmas; cópia separada diverge em silêncio.
3. **Banco do consentimento separado** do banco do site, com `migrateC15t()` chamado no
   `onInit` do `payload.config.ts` a cada boot (com `try/catch`, para o site subir mesmo se
   falhar). Na Unique o schema do consentimento nasceu vazio em produção: o site subiu
   normal e só o banner quebrou, o pior tipo de falha porque ninguém vê.
   - **Monte o `TrackingProvider` só depois de conhecer o estado do consentimento.** Na
     Unique, banner e provider pediam token ao mesmo tempo na primeira visita, cada um criava
     um visitante diferente, o consentimento ficava gravado num e o cookie apontava para o
     outro. Resultado: aceite registrado e nenhum hit saindo, sem erro em tela. Veja
     `src/components/site/Consentimento.tsx` da Unique.
   - **Banner próprio, textos vindos do CMS**, com duas categorias (Medição e Publicidade),
     botão de recusar com o mesmo peso visual do de aceitar, e link "Rever minhas escolhas" na
     página de privacidade.
4. **O worker de entregas só roda em produção.** O banco de dev acaba tendo as tags reais do
   cliente, e worker ligado em dev manda evento de teste para a conta dele. Para testar
   entrega: `TRACKING_WORKER=1`, sabendo disso. Em produção, sem worker, a fila cresce em
   silêncio e a Conversions API não recebe nada.
5. **Pixel, GA4, Google Ads e eventos se configuram por script idempotente**
   (`scripts/configurar-tracking.mjs` da Unique): IDs públicos no script, segredos (token da
   Conversions API, API secret do GA4) lidos do `.env` do cliente, nunca impressos.
6. **Página vista:** o GA4 conta as próprias páginas (`send_page_view: true` e a opção
   "mudanças de página com base no histórico" LIGADA na propriedade). O módulo recusa
   destino `page_view` para o GA4; página vista vai só para a Meta (PageView).
7. **Validação de tracking é pela rede, nunca pelo `dataLayer`.** A versão 1.1.0 passou em
   todos os testes sem nenhum hit do Google sair, porque os testes olhavam o `dataLayer`.
   Rode o roteiro de `docs/verificacao-de-rede.md` do módulo: antes do aceite nada sai;
   depois do aceite GA4, Meta e Ads disparam uma vez; navegação interna não duplica;
   revogar sem recarregar zera; reconceder volta. Em produção, `proxyLocal: false` e mock
   de `**/api/tracking/collect*`, senão o teste vira entrega real na conta do cliente.
8. **Não polua as contas do cliente** (GA4, Ads, Meta). Em teste, bloqueie os hits finais
   ou use o mock. Tag Assistant e Pixel Helper são conferência final do Filipe, não o seu
   teste.
9. **Atribuição de origem por cookie próprio, nunca por UTM em link interno.** A Unique
   guarda 13 parâmetros no cookie `ubh_rastreio` (90 dias, primeiro e último toque) e anexa
   ao lead. Leia `docs/RASTREIO-UTM.md` da Unique e replique com nome de cookie deste site.
   UTM em link interno quebra a sessão do GA4 e o pareamento do Google Ads.
10. **A leitura pública do consentimento por id** precisa ficar fechada no proxy de produção,
    com o servidor lendo pelo endereço interno (`C15T_URL=http://127.0.0.1:3000/api/c15t`).
    Veja o achado A5 da auditoria da Unique e o `Caddyfile` dela.

Débitos abertos do módulo que afetam um site novo: veja `DEBITO.md` do módulo (D-POS-001:
fila acumula sem aviso quando o worker não roda; D-POS-002: TTL de snapshot documentado e
não aplicado).

---

## 11. Segurança

Leia `docs/security-audit/` da Unique inteiro antes de escrever controle de acesso. Os sete
achados de lá, todos corrigidos, são a lista do que este site já tem que nascer sem:

| Achado | O problema | Como nascer certo |
|---|---|---|
| A1 | Limite de envio dos formulários contornado forjando `CF-Connecting-IP`, `X-Real-IP` ou `True-Client-IP` | O anti-spam usa o ÚLTIMO valor de `X-Forwarded-For`, escrito pelo proxy; o proxy apaga os outros três cabeçalhos; e existe teto global de envios |
| A2 | XSS armazenado no JSON-LD | Escapar o fechamento de `<script>` ao serializar |
| A3 | `PAYLOAD_SECRET` vazio virava chave pública (o hash de texto vazio é conhecido) | O boot recusa segredo ausente, curto (menos de 32 caracteres) ou igual ao de exemplo; o build usa um valor de mentira marcado com `SITE_EM_BUILD=1` que não chega ao contêiner final (`src/utilities/ambiente.ts`) |
| A4 | Compose de dev com senha fixa do banco, Adminer ligado e porta aberta | Senha vem do `.env` e o compose recusa subir sem ela; Adminer em profile opcional; portas presas em `127.0.0.1` |
| A5 | Leitura pública do consentimento por id | Proxy devolve 404 para a leitura por id; o servidor lê pelo endereço interno |
| A6 | Documentação e status do c15t expostos, com versão e IP | Proxy devolve 404 para `/api/c15t/docs` e `/api/c15t/status` |
| A7 | Todo usuário do painel era administrador | Papéis admin e editor desde o início; conta nova nasce editora; só a primeira conta de um banco vazio nasce admin |

Além disso, o mínimo:

- **Access control em toda coleção e global**, com função de papel (`ehAdmin` na Unique).
  Configuração sensível (e-mail, tracking, SEO global) é só de administrador.
- **A Local API ignora access control por padrão.** Em código que age em nome de um
  usuário, passe `overrideAccess: false` e o `user`. `overrideAccess: true` só em rotina
  de sistema, de propósito.
- **`access.create` ignora o `Where`**: no create, decida pelo `data`.
- **Field access nega em silêncio**: o campo é descartado e a operação segue.
- **`admin.readOnly` e `admin.hidden` são só tela**, não trava.
- **Upload com lista de tipos permitidos** e limite de tamanho. Formulário com anexo
  (currículo, por exemplo) valida tipo e tamanho no servidor.
- **Cabeçalhos de segurança no proxy** (HSTS, `X-Content-Type-Options`, `Referrer-Policy`,
  `frame-ancestors`), como no `Caddyfile` da Unique.
- **Segredo nunca em arquivo versionado nem em imagem Docker.** O build do Next precisa de
  um `PAYLOAD_SECRET` de mentira no Dockerfile; o valor real só existe no ambiente de
  produção.
- **LGPD:** página de privacidade com conteúdo do CMS, banner de consentimento antes de
  qualquer rastreio, e nenhum dado pessoal em log.
- **Ao fim da Fase 3**, peça uma auditoria de segurança por área (login, acesso,
  formulários, upload, tracking), com revisão de quem não escreveu o código.

---

## 12. Fase 3: front-end

- **Fidelidade ao HTML aprovado.** A migração não é redesenho: mesmo layout, mesmas cores,
  mesmas fontes, mesmo espaçamento. Diferença visual só com aprovação do Filipe.
- Componentes React a partir das seções do HTML, alimentados pelo CMS. Server Components por
  padrão; client só onde há interação.
- **Rotas renderizadas por requisição** (`force-dynamic`), como na Unique: o que o cliente
  muda no painel aparece no site na hora, sem publicar de novo.
- **Nunca rode `pnpm dev` e `pnpm build` ao mesmo tempo** na mesma pasta: os dois usam a
  pasta `.next` e ela corrompe ("Cannot find module"). Pare um antes do outro; se já
  corrompeu, apague a `.next` (a Unique tem `pnpm devsafe` para isso).
- Responsivo conferido em 360, 768, 1280 e 1920 de largura.
- Estados vazios: se o cliente apagar um campo opcional, a página não quebra nem mostra
  "undefined".

---

## 13. Validação local antes do GitHub

O Filipe valida local antes de qualquer commit no GitHub. Entregue com esta lista conferida
e com os números medidos:

1. `pnpm build` de produção passa na máquina (build local é a prova, nunca na VPS).
2. Typecheck e testes passam.
3. Todas as páginas do inventário abrem, com conteúdo vindo do CMS; editar no painel muda o
   site.
4. Redirects antigos testados um a um.
5. `sitemap.xml`, `robots.txt`, metadata, Open Graph e JSON-LD conferidos por página.
6. Peso das páginas principais medido (seção 7), Lighthouse de celular anotado.
7. Formulários: envio desligado vai para o log; anti-spam barra envio rápido e envio em
   excesso; plugin de e-mail conferido no admin.
8. Tracking: verificação de rede completa em build de produção local com HTTPS
   (`docs/verificacao-de-rede.md`), com a evidência salva.
9. `select name, batch from payload_migrations` sem linha `dev`.
10. Nenhum segredo em arquivo versionado (varredura antes do primeiro commit).

Mande ao Filipe o endereço local de cada tela que ele deve olhar.

---

## 14. GitHub e publicação (só quando o Filipe pedir)

Modelo da Unique, descrito em `.planning/DEPLOY.md` dela:

- Repositório **privado** no GitHub da Hiperbold.
- Push na `main` → GitHub Actions gera a imagem → GHCR → na VPS
  `docker compose pull && docker compose up -d`. Migrations rodam sozinhas no boot
  (`prodMigrations`).
- Caddy na frente, com HTTPS, cabeçalhos de segurança e as rotas internas fechadas.
- Volume Docker para fotos e para o banco, e **backup diário** do banco do site e do banco
  de consentimento na VPS (na Unique: timer do systemd, 14 dias de retenção, mais o
  snapshot automático do provedor, porque backup no mesmo disco não protege de perder a
  máquina).
- Rota de saúde leve (`/api/ping` na Unique, sem consultar o banco) para o healthcheck do
  contêiner.
- **DNS apontado antes de subir o Caddy**, senão ele não consegue emitir o certificado.
- Servidor endurecido como o da Unique (`scripts/preparar-vps.sh` e
  `scripts/endurecer-ssh.sh`): a VPS dela recebeu mais de 3 mil tentativas de SSH em 24
  horas e ficou intermitente até o endurecimento.
- Voltar para a versão anterior é trocar a tag da imagem (`IMAGEM_TAG=sha-xxxxxxx
  docker compose up -d`); o hash aparece no resumo do Actions.
- Antes da primeira publicação: `SITE_NOINDEX` vazio, `NEXT_PUBLIC_SERVER_URL` com o domínio
  final, sem linha `dev` em `payload_migrations`, e ensaio da migração numa cópia.
- Depois de publicar: verificação de rede no domínio real, envio de teste pelo painel,
  sitemap no Search Console (tarefa do Filipe), e acompanhamento do primeiro lead real.
- Acesso a servidor, painel de hospedagem ou banco de produção **só com autorização
  explícita do Filipe**, a cada vez.

---

## 15. Entregas por fase

| Fase | Entrega | Quem aprova |
|---|---|---|
| 0 | `.planning/INVENTARIO.md` | Filipe |
| 1 | Ambiente local no ar, admin acessível | Filipe entra no admin |
| 2 | Coleções, globals, conteúdo importado | Filipe navega no admin |
| 3 | Site completo local, fiel ao HTML | Filipe navega no site local |
| 4 | Formulários, e-mail, SEO, tracking | Lista da seção 13 com evidência |
| 5 | GitHub e publicação | Só quando o Filipe pedir |

Ao fim de cada fase: `.planning/HANDOFF.md` atualizado (o que foi feito, como retomar) e
`.planning/DEBITO.md` com tudo o que ficou para trás.
