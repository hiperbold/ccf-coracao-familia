# Coração Família — contexto do projeto

## Organização e relação

- O Centro de Convivência Familiar “Coração Família” (CCF) é uma organização sem fins lucrativos / ONG.
- Atua na Vila Santa Catarina, Zona Sul de São Paulo.
- Este é um projeto pro-bono do Filipe para a organização.
- O usuário forneceu como endereço: Rua Calasans, 200, São Paulo, SP, CEP 04369-100. Confirmar grafia/CEP com a organização antes de publicar.
- Instagram: https://www.instagram.com/coracaofamilia/
- Facebook: https://www.facebook.com/coracaofamilia/
- Referência adicional fornecida: https://share.google/gYrAjL0YEsZpvqOnq
- Contato (voluntariado), fornecido pelo Filipe em 25/09/2026: e-mail `voluntariado.cf@gmail.com`; WhatsApp `+55 11 98121-8419`. Usar em link `mailto:` e `https://wa.me/5511981218419` onde o site precisar de contato real (nav "Contato", CTA de inscrição sem link próprio do evento, etc.).

## Marca fornecida

- `icon-ccf.png` — ícone
- `logo-ccf-hq.png` — logo em alta qualidade
- `logo-ccf-lq.png` — logo em qualidade web
- **Favicon**: peça própria, não é um recorte do `icon-ccf.png`. Deve ser um círculo vermelho sólido com um ícone de coração na cor branca por cima (símbolo simplificado, não o mascote ilustrado). Ainda não gerado; gerar como SVG/PNG dedicado quando o site for implementado.

## Intenção do produto

O projeto será uma “Agenda do Coração Família”, não um site institucional extenso. A frente pública deve apresentar a marca, links sociais, formas de contato e um breve sobre o CCF. O principal conteúdo é uma lista cronológica de eventos atuais e futuros; o Filipe prefere essa visão a uma grade de calendário, especialmente para uso em celular.

Cada evento terá sua própria página. Um evento pode ocupar um dia (com ou sem horários definidos) ou vários dias; eventos de vários dias podem ter uma programação própria para cada dia. A página exibe apenas as informações preenchidas.

O local padrão é a sede do CCF, mas deve ser possível alterá-lo em cada evento pelo painel administrativo.

Neste primeiro momento não haverá cadastro/login de visitante: a frente pública é só a agenda aberta, sem conta de usuário. Cadastro/autenticação, se vier a existir, é só para quem administra o conteúdo (painel do CMS).

## Direção preliminar, ainda não aprovada como especificação

- Usar um CMS com conteúdo de eventos estruturado, em vez de adotar uma plataforma de ingressos completa.
- Payload CMS é a recomendação inicial para esta experiência customizada: o projeto pode unir site público e painel administrativo, mantendo autenticação, permissões, mídia e dados de eventos num só sistema.
- Quando a implementação do Payload CMS começar, seguir `GUIA-IMPLEMENTACAO-PAYLOAD.md` (raiz deste repo): guia consolidado com o que funcionou e quebrou na implementação de referência (site Unique By House), regras de trabalho com o Filipe, armadilhas conhecidas do Payload 3 e checklist de publicação. Ler inteiro antes de começar; quando o guia e o instinto do agente discordarem, seguir o guia e registrar a discordância no débito do projeto.
- Cada evento deve aceitar datas/dias e faixas horárias estruturadas; cada dia pode conter seus próprios itens de programação e convidados. O local padrão fica nas configurações do CCF, com substituição opcional no evento.
- A lista deve incluir eventos que ainda estejam acontecendo, além dos que começam no futuro. Considerar eventos sem hora marcada.
- Definir hospedagem, rotina de backup, custo operacional e quem atualizará a agenda antes de publicar.

## Informações úteis a avaliar por evento

Além de título, imagem/banner, resumo/descrição, datas e horários, programação, convidados e local:

- inscrição ou contato (link, WhatsApp, telefone ou formulário), quando aplicável;
- gratuito ou valor, eventual prazo de inscrição e capacidade/vagas;
- público indicado ou faixa etária;
- orientações de acesso, acessibilidade, transporte ou o que levar, quando relevante;
- contatos e links associados;
- status de alteração, cancelamento ou lotação, para evitar que visitantes se desloquem com informação desatualizada;
- compartilhamento e opção de adicionar à agenda pessoal — confirmado pelo Filipe como funcionalidade real a implementar (botões já desenhados no design system), não só decoração.

Todos os campos opcionais devem desaparecer da página pública quando não forem preenchidos.

O bloco de local sempre vem com um mini-mapa (embed do Google Maps, sem chave de API — funciona só com o endereço) e um link direto de rota (`google.com/maps/dir/?api=1&destination=...`).

## Mídia (banners, fotos)

Banner de evento e fotos de convidado devem ser convertidas para WebP no momento do upload (painel administrativo), guardando só a versão web; o arquivo original enviado não fica armazenado, para economizar espaço em disco. Detalhar o pipeline exato (biblioteca de conversão, tamanhos gerados) quando a implementação do Payload começar.

Placeholders de imagem usados no design system ficam em `placeholder/` (fora do fluxo real do produto, só para a documentação viva): `Beatriz-Nunes.webp` (foto de convidado) e `banner-evento.webp` (banner de evento, 16:9).

## Pesquisa inicial de referências (2026-09-25)

- **Event Schedule** — https://github.com/eventschedule/eventschedule — referência funcional mais próxima: agenda pública, páginas de evento, blocos de programação com horários e eventos de vários dias. Também inclui venda de ingressos, check-in, pagamentos, newsletter e analytics; serve melhor como inspiração do que como produto a adotar sem uma avaliação de escopo/licença.
- **Event Organiser para WordPress** — https://wordpress.org/plugins/event-organiser/ — eventos únicos/recorrentes, locais, listas e agendas; alternativa de menor customização inicial, mas o modelo de programação por dia e perfis de convidados exigiria adaptações.
- **Mobilizon** — https://docs.mobilizon.org/about/ — plataforma livre e federada para grupos e eventos; boa referência de organização comunitária, mas orientada a grupos/participação e federação, mais ampla do que a agenda própria do CCF.
- **Hi.Events** — https://github.com/HiEventsDev/hi.events — plataforma abrangente de venda/gestão de ingressos; provavelmente mais do que a ONG precisa para publicar agenda informativa.
- **pretix** — https://github.com/pretix/pretix — plataforma madura de pré-venda e gestão de ingressos para eventos; mais indicada se houver venda/controle de inscrições.
- **Payload CMS** — https://payloadcms.com/docs/admin/overview — o painel administrativo, autenticação, coleções e controles de acesso são documentados; o site ainda precisa de um modelo e de uma interface próprios. O template oficial para sites pode ser o ponto de partida técnico.

Esta pesquisa é uma orientação inicial, não uma decisão de implementação.

## Estado atual do repositório (25/09/2026)

- `ccf_design_system.html` + `ccf_design_tokens.css`: design system vivo da Agenda, publicado em https://hiperbold.github.io/ccf-coracao-familia/ccf_design_system.html (repo público, para compartilhar com a ONG). **Aprovado pelo Filipe em 25/09/2026** (fontes, componentes, layout, alinhamentos). É a fonte de verdade para tudo que vier depois. Ainda não há código de aplicação (site/CMS funcional) — só a documentação viva do sistema visual e dos componentes.
- Próxima etapa (em andamento): modelos de página desktop (homepage e página interna de evento, 2 opções de cada). Depois: Filipe aprova um modelo desktop e a versão mobile correspondente; aprovado, cada modelo vira um `.html` próprio com assets, publicado do mesmo jeito (GitHub Pages).
