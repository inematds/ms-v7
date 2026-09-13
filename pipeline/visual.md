# Pipeline · Visual

A skill lê `config.yaml` → `provedores.imagem` / `provedores.video` e o bloco **ESTILO VISUAL** do `brand-voice.md` da marca ativa.

**Formatos por rede:**

| Uso | Tamanho | Proporção |
|---|---|---|
| Reel / TikTok / Shorts (vídeo ou capa) | 1080×1920 | 9:16 |
| Feed Instagram / carrossel | 1080×1350 | 4:5 |
| X / LinkedIn / Facebook imagem | 1200×1500 ou 1080×1350 | 4:5 |
| YouTube comunidade | 1080×1080 | 1:1 |

**Saída:** `~/projetos/output/ms-v7/<data>-<slug>/`. A imagem aprovada é copiada para `posts/<data>-<slug>/`.

**Aprovação:** mostrar o caminho (ou enviar via SendUserFile) e perguntar em texto "Visual ok?" antes de hospedar.

---

## flux2-klein (padrão, custo 0)

Prompt = `ESTILO VISUAL` do brand-voice + descrição da cena do post + formato. Sem texto dentro da imagem (o modelo erra tipografia).

**Carrossel:** o texto NÃO vem do gerador. Gerar o fundo com flux e sobrepor o texto com HTML → PNG (skill `web-artifacts-builder` ou um `index.html` renderizado com `chromium --headless --screenshot`), ou Pillow/ImageMagick. Estrutura: slide 1 = frase-gancho grande · slides 2..N = 1 ideia por slide · último = CTA.

## magnific (fallback, custa crédito)

Ler `~/.claude/runbooks/magnific-modelos.md` antes de escolher. Resumo: `z-image` 5cr · `flux-2-klein` 10cr · `gpt-2` 15cr (quando precisa de texto na imagem) · `seedream-5-pro` 100cr. **Nunca `auto`.** Usar quando: referência de personagem (rosto consistente), edição guiada, upscale, vetorizar. `simulate_cost` antes de lote > 10. Avisar se um lote passar de 5.000 créditos.

## video

| Provedor | Quando | Custo |
|---|---|---|
| `pixflow-motion` | foto(s) → movimento de câmera/parallax, sem IA generativa | 0 |
| `video-explicativo` / `hyperframes` | explicativo HTML→MP4 com narração | 0 |
| `magnific-kling` (`kling-25`) | cena impossível de filmar | 140cr / 5s 720p |

Narração: inemavox TTS (engine `chatterbox`, voz `rachel`). Música/SFX: inemavox/dlp. Vídeo de avatar do Nei: skill `avatar-heygen-nei`.
