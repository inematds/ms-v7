---
name: ms-semana
description: A semana no autopilot (Prompt 3). Retro da semana passada com analytics do provedor, 3 perguntas para a semana, plano de 5-7 posts em tabela para aprovar, depois escreve tudo, gera visuais e agenda a fila SÓ depois do "sim". ~15 min por semana. Use para "planeja minha semana", "posts da semana", "faz a fila da semana".
---

# /ms-semana — A semana inteira em 15 minutos

Perguntas em texto livre, uma por vez. Mesmo carregamento de contexto do `/ms-post` (passo 0): `config.yaml`, `brand-voice.md` da marca ativa, redes ativas, provedor, contagem de cota.

## 1. Retro (a partir da 2ª semana)

- Ler `posts/semanas/` — pegar a semana anterior.
- Se o provedor tem analytics (Metricool: `getAnalyticsAvailableMetrics(rede, conector)` → IDs → `getAnalyticsDataByMetrics`, **um conector por chamada**, últimos 7 dias, janela máx. 30 dias no Free): dizer em 5 linhas quais posts foram melhor (alcance, salvamentos, comentários) e **o que muda esta semana** (formato, gancho, horário).
- Se não tem analytics: perguntar "qual post da semana passada foi melhor?" e usar a resposta.
- Gravar a retro no topo de `posts/semanas/<AAAA-WW>.md`.

## 2. Três perguntas (uma por vez)

1. Quais redes esta semana (só as ativas)?
2. Temas ou notícias da sua semana que devem entrar — ou "pesquise ângulos em alta"?
3. Objetivo nº 1 da semana?

## 3. Plano — mostrar e ESPERAR edições

- **5 a 7 posts**, nunca acima da cota mensal restante.
- Mix que a rede premia (Instagram: 2 Reels + 2 carrosséis + 1 foto; TikTok: 2 roteiros; X: posts + 1 thread; LinkedIn: histórias + 1 opinião contrária) — só para redes ativas.
- Horário: `getBestTimeToPostByNetwork` se disponível; senão, o horário padrão do `brand-voice.md` ou 12h/19h.
- Alternar gatilhos: no máximo 40% de "aversão à perda"; cada pilar aparece no máximo 2×.

Tabela obrigatória:

| # | Dia | Hora | Rede | Formato | Gancho (≤ 8 palavras) | Pilar | CTA |
|---|---|---|---|---|---|---|---|

Perguntar: "Edita algo ou aprovo o plano?" Aplicar edições e mostrar de novo até o "ok".

## 4. Escrever tudo

Para cada linha: mesmas regras do `/ms-post` passo 3 (história/número/frase do arquivo, 1 CTA, checklist). Gravar cada um em `posts/<data>-<slug>/post.md`. Gerar os visuais (`pipeline/visual.md`) e pedir "Visuais ok?" com a lista de arquivos.

## 5. Fila completa → aprovação → agendar

Mostrar a fila: dia · hora · rede · texto completo · visual. Perguntar:

`Agendo a fila? (sim / rascunho / não)`

- `sim` → um `createScheduledPost` (ou equivalente do provedor) por post, com autoPublish; gravar `id`/`uuid` em cada `post.md`.
- `rascunho` → todos como rascunho no provedor.
- `não` / `copiar` → entregar tudo em blocos de código, um por post.

Nunca agendar sem a resposta explícita nesta conversa. Parar no primeiro erro do provedor, mostrar o erro, não tentar o resto.

## 6. Fechar

Atualizar `posts/semanas/<AAAA-WW>.md` com a tabela final e os status. `git add -A && git commit -m "semana <AAAA-WW>: N posts (<status>)"`. Dizer em 3 linhas: quantos agendados, cota restante, quando rodar de novo.
