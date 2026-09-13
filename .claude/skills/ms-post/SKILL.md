---
name: ms-post
description: O post do dia (Prompt 2 do Social Autopilot). Lê config.yaml + brand-voice da marca ativa, faz 3 perguntas (rede, tema, objetivo), pesquisa o que funciona na rede esta semana, escreve 3 opções na voz da marca, gera o visual e agenda pelo provedor configurado SÓ depois do "sim". Use para "faz o post de hoje", "cria um post pro Instagram", "post sobre X".
---

# /ms-post — Um prompt, qualquer rede

## 0. Carregar contexto (sem perguntar nada)

1. Ler `config.yaml`. Se `marca_ativa: null` → dizer "nenhuma marca configurada, rode /ms-config" e parar.
2. Ler `brands/<marca_ativa>/brand-voice.md` inteiro.
3. Listar redes com `ativa: true`. Ler o provedor de publicação da marca (ou o padrão). Se for MCP, checar se está conectado (uma chamada de leitura, ex.: `getBrandSettings`); se não estiver, seguir como `copiar` e avisar.
4. Contar posts do mês em `posts/` (pastas `YYYY-MM-*` com status `agendado|publicado`). Se ≥ `avisar_cota_em × cota_mensal`, avisar antes de continuar.

## 1. Três perguntas — em texto livre, UMA por vez, esperar a resposta

1. **Rede:** listar só as ativas. Se o usuário pedir uma rede inativa, avisar e oferecer o texto em modo `copiar`.
2. **Tema:** algo que aconteceu / algo pra ensinar / uma oferta / "pesquise o que está em alta no nicho e me traga 5 ângulos".
3. **Objetivo:** crescer seguidores / DMs e leads / vender uma oferta / puxar conversa.

Se o usuário já deu as três respostas na mensagem inicial, não perguntar de novo.

## 2. Pesquisa (só se o tema for "em alta" ou o usuário pedir)

Provedor em `config.yaml → provedores.pesquisa`. Buscar "o que está funcionando nesta rede, para este tipo de conteúdo, esta semana" (ganchos, formatos, duração). Resumir em 5 linhas com as fontes. Se o tema for "em alta": propor 5 ângulos e esperar a escolha.

## 3. Escrever 3 opções

Cada opção obrigatoriamente:
- segue a seção da rede em `brand-voice.md` (gancho, tamanho, hashtags, formato);
- inclui **≥ 1 história, número ou frase do arquivo** que ninguém mais poderia postar;
- termina com **exatamente 1 CTA** do arquivo, compatível com o objetivo;
- zero AI-speak e zero palavra proibida do arquivo;
- Reel/TikTok = roteiro com marcação de tempo (`[0-3s]` gancho · `[3-20s]` tensão · `[20-40s]` virada · fechamento) + texto na tela.

**Checklist antes de mostrar** (rodar em silêncio, corrigir o que falhar): gancho ≤ 8 palavras · limite de hashtags da rede · 1 CTA · sem proibidas · sem número inventado · "um seguidor acreditaria que a pessoa escreveu isso?".

Mostrar as 3, numeradas, e perguntar qual (ou "mistura a 1 com a 3").

## 4. Gravar

`posts/<AAAA-MM-DD>-<slug>/post.md`:
```markdown
---
marca: <slug>
rede: instagram
formato: reel | carrossel | foto | texto
objetivo: ...
status: rascunho   # rascunho → visual-ok → agendado | publicado | pronto-para-copiar
provedor: metricool
agendado_para: null
id: null
uuid: null
---
<texto final>
```

## 5. Visual

Seguir `pipeline/visual.md` com o provedor de `config.yaml`. Se `imagem.padrao: nenhum` ou o usuário disser "sem visual", pular. Mostrar o arquivo, perguntar "Visual ok?". Copiar o aprovado para a pasta do post; `status: visual-ok`.

## 6. Publicar

Seguir `pipeline/publicar.md` na seção do provedor. Hospedar a mídia se o provedor exigir URL. Mostrar **post final + horário proposto** e perguntar:

`Agendo? (sim / rascunho / não)`

- `sim` → agenda com publicação automática; gravar `agendado_para`, `id`, `uuid`; `status: agendado`.
- `rascunho` → cria como rascunho no provedor; `status: rascunho-no-provedor`.
- `não` / provedor `copiar` → entregar o texto em bloco de código; `status: pronto-para-copiar`.

**Nunca chamar a ferramenta de criação de post sem a resposta "sim" ou "rascunho" nesta mesma conversa.**

## 7. Fechar

`git add posts/ && git commit -m "post: <data>-<slug> (<rede>, <status>)"`. Dizer em 2 linhas o que ficou e onde.
