# ms-v7 — Social Autopilot

Um arquivo de voz de marca + 3 prompts → posts nativos por rede, na sua voz, agendados pelo provedor que você escolher. Nada vai ao ar sem o seu "sim".

Adaptado de *The Social Autopilot* (Zubair Trabzada, AI Workshop) — os originais estão em `docs/`.

## Em 10 linhas

```
1. abra o Claude Code nesta pasta
2. /ms-config        → entrevista de voz de marca + redes + provedor (uma vez)
3. /ms-post          → 3 perguntas (rede, tema, objetivo) → 3 opções → escolhe
4.                   → visual gerado (flux2-klein) → "Visual ok?"
5.                   → post final + horário → "Agendo? (sim / rascunho / não)"
6. /ms-semana        → retro + tabela de 5-7 posts → aprova → fila agendada
7. /ms-reaproveita   → transcrição/artigo/URL → um post por rede
```

## Configurável

- **Marcas:** quantas quiser em `brands/<slug>/`; `marca_ativa` no `config.yaml`.
- **Redes:** cada uma com handle e `ativa: true/false`.
- **Publicação:** `metricool` (MCP, validado) · `blotato` (MCP, não testado) · `copiar` (sem API, sempre funciona).
- **Imagem/vídeo:** `flux2-klein` (padrão) · `magnific` · `pixflow` · `video-explicativo`.

Detalhes de cada provedor: `pipeline/publicar.md` e `pipeline/visual.md`. Plano de implementação: `docs/PLANO.md`.
