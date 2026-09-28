# ms-v7 — Social Autopilot

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Un archivo de voz de marca + 3 prompts → publicaciones nativas por red, con tu voz, programadas por el proveedor que elijas. **Nada se publica sin tu «sí».**

Adaptado de *The Social Autopilot* (Zubair Trabzada, AI Workshop). Los originales (prompt pack, plantilla de voz, transcripción del video) están en `docs/`. El plan de implementación está en `docs/PLANO.md`.

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/ms-v7/guia/es/**

---

## 1. Cómo funciona (en 1 minuto)

```
brand-voice.md  ─┐
config.yaml     ─┼─►  /ms-post  ──► 3 preguntas ──► 3 opciones ──► visual ──► "¿Programo? (sí/borrador/no)"
proveedores      ─┘        │
                          ├─► /ms-semana       retro + tabla de 5-7 publicaciones + cola de la semana
                          └─► /ms-reaproveita  transcripción / artículo / URL → una publicación por red
```

- **Voz de marca** (`brands/<slug>/brand-voice.md`): quién eres, público, ofertas, frases, historias reales, reglas por red. Es lo que hace que la publicación suene como tú y no como IA.
- **Config** (`config.yaml`): qué marca está activa, qué redes reciben publicaciones, quién publica (Metricool, Blotato o copiar a mano), quién genera imágenes/videos.
- **Skills** (`.claude/skills/`): los prompts del pack empaquetados como comandos de Claude Code.
- **Pipelines** (`pipeline/`): recetas paso a paso de cada proveedor.

---

## 2. Inicio (requisitos previos)

| Qué | Cómo | ¿Obligatorio? |
|---|---|---|
| Claude Code | `claude.com/claude-code` — plan Pro o superior | sí |
| Este repo | `git clone git@github.com:inematds/ms-v7.git ~/projetos/ms-v7` | sí |
| Python 3 + PyYAML | `pip install pyyaml` (solo para validar `config.yaml`) | recomendado |
| Proveedor de publicación | Metricool (MCP, gratis) o Blotato (MCP, de pago) — ver §4 | no (sin él, modo `copiar`) |
| Generador de imágenes | flux2-klein local (predeterminado) o Magnific (MCP) | no |
| Host de medios | `gh` autenticado en la cuenta `inematds` (release en `inematds/midia`) | solo con proveedor MCP |

Abre Claude Code **dentro de la carpeta**:

```bash
cd ~/projetos/ms-v7
claude
```

Lee `CLAUDE.md` automáticamente y pasa a conocer las reglas y los comandos.

---

## 3. Configurar la primera marca

En Claude Code:

```
/ms-config
```

El comando pregunta **una cosa a la vez**, en texto libre. Tiene tres bloques:

**A. Voz de marca** → crea `brands/<slug>/brand-voice.md` a partir de `brands/_template/`.
- Si la marca ya está descrita en algún lugar (una skill, un sitio, un doc), indica dónde: completa los datos de antemano y solo pregunta lo que falta.
- Lo que siempre pregunta porque nadie puede inventarlo: 5 frases que realmente dices, 3 historias reales con números, pruebas verificables y CTAs exactos.
- Rechaza respuestas vagas («muchos alumnos» → «¿cuántos?»).
- Termina con una **prueba de voz**: escribe un pie de foto solo con el archivo y pregunta «¿escribirías esto?».

**B. Redes y proveedores** → completa `config.yaml`.
- Para cada red: handle y si está `ativa`. Solo las redes activas reciben publicaciones.
- Proveedor de publicación: `metricool` · `blotato` · `copiar`.
- Imagen: `flux2-klein` (predeterminado) o `magnific`. Video: `nenhum` hasta que haga falta.

**C. Ajustes posteriores**: «activa LinkedIn», «cambia a blotato», «usa la marca X» — edita solo la clave solicitada.

También puedes editar `config.yaml` a mano. Validar:

```bash
python3 -c "import yaml; yaml.safe_load(open('config.yaml'))" && echo ok
```

### Varias marcas

Cada marca es una carpeta en `brands/`. Solo una está activa (`marca_ativa` en `config.yaml`). Para cambiar: «usa la marca X» o edita la clave.

---

## 4. Conectar un proveedor de publicación

### Metricool (validado, plan Free)

```bash
claude mcp add --transport http metricool https://ai.metricool.com/mcp -s user
```

Dentro de Claude Code: `/mcp` → **metricool** → **Authenticate** → autoriza en el navegador. Luego, en `/ms-config`, indica el `blogId` (lo descubre con `getBrandSettings`).

Límites del Free: 1 marca, 1 cuenta por red, **sin LinkedIn ni X**, **20 publicaciones/mes**, analytics de 30 días, sin opción de «borrar publicación» mediante MCP (cancélala en la app).

### Blotato (no probado aquí)

Crea una cuenta → conecta las redes → copia la URL del MCP → `claude mcp add --transport http blotato <url> -s user`. Antes de usarlo en producción, crea una publicación en `rascunho` y anota en `pipeline/publicar.md` el payload que funcionó.

### Copiar (sin API)

No hay nada que conectar. Las skills entregan el texto listo en un bloque de código y lo pegas en la red. Es la opción predeterminada mientras no haya ningún proveedor configurado.

### Host de medios

Los proveedores MCP solo aceptan **URL directas** de imágenes/videos. La opción predeterminada es un GitHub Release:

```bash
gh release upload v1 arquivo.png --repo inematds/midia --clobber
# → https://github.com/inematds/midia/releases/download/v1/arquivo.png
```

Las skills lo hacen automáticamente; solo necesitas tener `gh` conectado a la cuenta `inematds`.

---

## 5. Uso diario

| Comando | Qué sucede | Tiempo |
|---|---|---|
| `/ms-post` | 3 preguntas (red, tema, objetivo) → investigación opcional → 3 opciones → eliges → visual → «¿Programo?» | 3-5 min |
| `/ms-semana` | Retro de la semana pasada (analytics) → 3 preguntas → tabla de 5-7 publicaciones → editas/apruebas → escribe todo → visuales → «¿Programo la cola?» | 15 min |
| `/ms-reaproveita` | Pega una transcripción/artículo/URL → «¿qué redes?» → una publicación independiente por red → cola | 5 min |

**La confirmación de aprobación** acepta tres respuestas:

- `sim` → programa la publicación automática para la hora propuesta.
- `rascunho` → la crea como borrador en el proveedor (no la publica).
- `não` → solo el texto, para copiarlo.

Cada publicación queda en `posts/<data>-<slug>/post.md` con su estado, el `id`/`uuid` del proveedor y el visual aprobado. Las semanas quedan en `posts/semanas/`.

---

## 6. Publicar en git

El trabajo **termina con el push**. El webhook es responsable del deploy, no tú.

```bash
cd ~/projetos/ms-v7
git config user.email   # tem que ser inematds@gmail.com
git add -A
git commit -m "post: 2026-09-15-ler-documentacao (instagram, agendado)"
git push
```

Reglas:
- Repo: `inematds/ms-v7`. Autor y committer: `inematds <inematds@gmail.com>`. Si `git config user.email` no coincide, corrígelo **localmente** (`git config user.email inematds@gmail.com`), sin tocar la configuración global.
- Las skills ya crean el commit al final de cada publicación/semana. Solo tienes que hacer el `git push`.
- Los videos y las imágenes fuera de `posts/` se ignoran mediante `.gitignore`. Los medios públicos van en un release del repo `inematds/midia`, nunca en el historial de este repo.
- Versión en `VERSION` y en la parte superior de `CLAUDE.md` (`vX.XX.YY`: patch incrementa `YY`; feature incrementa `XX` y conserva `YY`; solo major pone los demás en cero).
- Falla real → una línea en `FALHAS.md` (`| data | o que quebrou | menor correção | prompt \| infra |`).

---

## 7. Publicar en el portal (inema.club)

El portal apunta a una **página landing + guía** servida por GitHub Pages **de este mismo repo** (nunca de un repo separado).

**Paso 1 — crear la guía** (ya existe en `guia/index.html`; volver a generarla si cambia el proyecto):

```
/projetos-landing-guia
```

Genera `guia/index.html` (self-contained, estándar INEMA dark ámbar) con `guia/assets/` al lado.

**Paso 2 — activar GitHub Pages mediante GitHub Actions** (el workflow `.github/workflows/pages.yml` ya está en el repo; no uses el build «legacy» por branch, que se bloquea):

```bash
gh repo edit inematds/ms-v7 --visibility public --accept-visibility-change-consequences   # Pages exige que el repo sea público en el plan free
gh api -X POST repos/inematds/ms-v7/pages -f build_type=workflow \
  || gh api -X PUT repos/inematds/ms-v7/pages -f build_type=workflow
git add guia && git commit -m "docs: guia landing" && git push   # el push activa el deploy
gh run list --repo inematds/ms-v7 --limit 3                       # consultar el estado
```

> Los commits que modifican `.github/workflows/` deben enviarse por SSH (`git@github.com:inematds/ms-v7.git`): el token HTTPS de la cuenta `inematds` no tiene el alcance `workflow`. El remote de este repo ya es SSH.

URL resultante: `https://inematds.github.io/ms-v7/guia/es/`. Verifica con `curl -sI <url> | head -1` (debe responder 200).

> Antes de hacerlo público: `brands/` contiene tu voz de marca e historias personales, y `posts/` contiene el historial. Si no quieres que eso sea público, mueve la guía a una rama `gh-pages` que solo incluya `guia/`, o mantén el repo privado y aloja la guía en otro lugar.

**Paso 3 — agregar al portal:**

```
/atualiza-portal https://inematds.github.io/ms-v7/guia/es/
```

La skill crea la tarjeta en las 3 superficies INEMA (portal `inema.club`, inemabuscas y catálogo PRO), crea el commit y hace push en los 3 repos. El deploy en Vercel es automático mediante el webhook; no hace falta (ni se debe) consultar el dashboard.

---

## 8. Mapa del repo

```
ms-v7/
├── CLAUDE.md                  reglas fijas + comandos (Claude Code lo lee automáticamente)
├── README.md                  este archivo
├── VERSION                    1.0.0
├── FALHAS.md                  registro de fallas, una línea cada una
├── config.yaml                marca activa · redes · proveedores · reglas
├── brands/
│   ├── _template/brand-voice.md   plantilla de voz (PT-BR, todas las redes)
│   └── <slug>/brand-voice.md      una carpeta por marca
├── pipeline/
│   ├── publicar.md            Metricool (validado) · Blotato · copiar · alojamiento
│   └── visual.md              formatos por red · flux2-klein · Magnific · video
├── posts/
│   ├── <AAAA-MM-DD>-<slug>/post.md   cada publicación, con estado e id/uuid
│   └── semanas/<AAAA-WW>.md          plan + retro de cada semana
├── .claude/skills/
│   ├── ms-config/   ms-post/   ms-semana/   ms-reaproveita/
├── guia/                      landing + guía (GitHub Pages) → inematds.github.io/ms-v7/guia/
└── docs/                      originales del pack + PLANO.md
```

---

## 9. Problemas comunes

| Síntoma | Causa probable | Solución |
|---|---|---|
| «ninguna marca configurada» | `marca_ativa: null` | `/ms-config` |
| La skill entrega solo texto, no programa | proveedor = `copiar` o MCP desconectado | `/mcp` → Authenticate, o `/ms-config` «cambia a metricool» |
| `createScheduledPost` rechaza el medio | La URL no es un archivo directo o no responde 200 | `curl -sI <url>`; volver a ejecutar `gh release upload` |
| Se duplicó una publicación en YouTube | La red `youtube` está activa en una marca cuyo canal es el origen de otro pipeline | `ativa: false` para youtube en `config.yaml` |
| Analytics vacío en la retro | Ventana de 30 días del Free o conector incorrecto | consulta `pipeline/publicar.md` § Analytics |
| Push rechazado por autor | `user.email` incorrecto | `git config user.email inematds@gmail.com` y commit vacío |
