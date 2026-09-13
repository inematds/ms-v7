# Pipeline · Publicar

Como cada provedor de publicação funciona. A skill lê `config.yaml` → `marcas.<ativa>.publicacao.provedor` (ou `provedores.publicacao.padrao`) e segue a seção correspondente.

**Regra que vale para todos:** mostrar o post final + horário proposto e perguntar em texto livre:
`Agendo? (sim / rascunho / não)`. Só agir com a resposta. Nunca publicar sem "sim".

---

## copiar

Sempre funciona, não precisa de nada conectado.

1. Entregar o texto final em bloco de código, um por rede, com o nome do arquivo de mídia ao lado.
2. Gravar em `posts/<data>-<slug>/post.md` com `status: pronto-para-copiar`.
3. Se o usuário publicar à mão, ele diz "publicado" → status vira `publicado` com data.

---

## metricool

**Status:** validado ponta a ponta em 2026-08-19 (Reel Instagram + TikTok). Referência completa: `~/projetos/pubmetricool/docs/`.

**Conectar (uma vez):**
```bash
claude mcp add --transport http metricool https://ai.metricool.com/mcp -s user
# depois: /mcp → metricool → Authenticate (OAuth no navegador)
```

**Ordem obrigatória de chamadas:**
1. `getBrandSettings` → confirmar `blogId`, fuso e redes conectadas batem com `config.yaml`. Se não baterem, parar e avisar.
2. `getBestTimeToPostByNetwork(network)` → propor 1 horário (maior peso, dentro dos próximos 7 dias).
3. Mostrar post + horário → esperar `sim | rascunho | não`.
4. `createScheduledPost` com o payload abaixo. `sim` → `autoPublish: true, draft: false`. `rascunho` → `draft: true`.
5. Gravar `id` **e** `uuid` da resposta no `post.md` (são obrigatórios para `updateScheduledPost`, que exige payload completo — o que faltar é apagado).

**Payloads validados:**
```jsonc
// Reel IG + TikTok (validado)
{
  "text": "legenda...",
  "media": ["https://github.com/inematds/midia/releases/download/v1/x.mp4"],
  "providers": [{"network":"instagram"},{"network":"tiktok"}],
  "publicationDate": {"dateTime":"2026-09-15T12:00:00","timezone":"America/Sao_Paulo"},
  "autoPublish": true,
  "draft": false,
  "instagramData": {"type":"REEL","showReelOnFeed":true},
  "tiktokData": {"privacyOption":"PUBLIC_TO_EVERYONE","title":"..."}
}
// Carrossel IG (⚠ NÃO validado — confirmar no primeiro carrossel real)
{ "media": ["url1.png", "...", "urlN.png"], "providers":[{"network":"instagram"}], "instagramData": {"type":"POST"} }
```

**Limites do plano Free:** 1 marca · 1 conta por rede · sem LinkedIn/X · **20 posts/mês** (não documentado se rascunho ou multi-rede conta 1 ou N) · analytics de 30 dias · **não há delete no MCP** (cancelar no app). "Publicar agora" = `dateTime` = agora + 5 min.

**Analytics (para a retro do `/ms-semana`):** `getAnalyticsAvailableMetrics(network, connector)` → IDs (ex.: `IGPO01`) → `getAnalyticsDataByMetrics`. Um conector por chamada. TikTok vídeo usa conector `posts`.

---

## blotato

**Status:** não testado aqui. Dados da página pública (2026-08-19), ver `~/projetos/pubmetricool/docs/comparativo-blotato.md`.

- MCP hospedado + REST `backend.blotato.com/v2`; API inclusa no Starter ($29/mês); trial de 7 dias **sem** API.
- 20 contas no total, sem o modelo "1 por rede". Tem resposta a DM/comentário (Metricool não tem).
- Conectar: criar conta → Settings → conectar redes → API → URL do MCP → `claude mcp add --transport http blotato <url> -s user`.
- Antes de usar em produção: repetir o teste de rascunho da Task 3 do plano e **anotar aqui** o payload que funcionou.

---

## hospedagem

Provedores MCP só aceitam **URL direta de arquivo**. Padrão: GitHub Release em `inematds/midia` (os arquivos não entram no histórico do repo).

```bash
gh release upload v1 <arquivo> --repo inematds/midia --clobber
# → https://github.com/inematds/midia/releases/download/v1/<arquivo>
curl -sI <url> | head -1   # tem que responder 200/302 na hora do agendamento
```

Nome do arquivo: `<data>-<slug>-<rede>.<ext>` para não sobrescrever posts antigos com `--clobber`.
