# Setting Up Local AI Image Generation

BrandForge Studio ships with a **built-in SVG renderer** that produces every logo, icon, stationery
layout, social template, pattern and design document with no extra software. This guide is for the
*artwork* part — hero images, marketing visuals, posters, mockups — where you want an actual
image-generation model.

You have three options:

| Option | Cost | Hardware | What it gives you |
| --- | --- | --- | --- |
| **Nothing** | Free | Any | SVG logos, icons, stationery, social templates, patterns, PDFs |
| **Automatic1111 / Forge** | Free | GPU recommended | All of the above **plus** AI artwork |
| **ComfyUI** | Free | GPU recommended | Same, with node-level control |

**Recommended: Automatic1111 / Forge.** It's free, runs on your own machine, needs no account or API
key, and its whole API is a single `POST`. If you already run ComfyUI, keep using it — both are
selectable in the app.

---

## Table of contents

- [Part 1 — Automatic1111 (recommended)](#part-1--automatic1111-recommended)
  - [1.1 Install](#11-install)
  - [1.2 Turn on the API](#12-turn-on-the-api--this-is-the-step-people-miss)
  - [1.3 Download a model](#13-download-a-model)
  - [1.4 Verify the API](#14-verify-the-api)
  - [1.5 Connect BrandForge](#15-connect-brandforge)
  - [1.6 Generate your first artwork](#16-generate-your-first-artwork)
- [Part 2 — Forge](#part-2--forge)
- [Part 3 — ComfyUI](#part-3--comfyui)
- [Part 4 — Choosing the provider in the app](#part-4--choosing-the-provider-in-the-app)
- [Part 5 — Troubleshooting](#part-5--troubleshooting)
- [Part 6 — Hardware quality expectations](#part-6--hardware--quality-expectations)

---

## Part 1 — Automatic1111 (recommended)

### 1.1 Install

You need **Python 3.10 or 3.11** and **git**.

**Windows (NVIDIA GPU — easiest path):**

Download the standalone package, which already contains Python and all dependencies:

1. Go to <https://github.com/AUTOMATIC1111/stable-diffusion-webui/releases/latest>
2. Download `sd.webui.zip`
3. Extract it somewhere with a short path and **no spaces**, e.g. `C:\sd`
4. Skip to [1.2](#12-turn-on-the-api--this-is-the-step-people-miss)

**Windows / Linux / macOS (manual install):**

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
python -m venv venv

# Windows
venv\Scripts\activate
# Linux / macOS
source venv/bin/activate

pip install -r requirements.txt
```

> **AMD GPU on Windows** — install a ROCm build of PyTorch before
> `pip install -r requirements.txt`, otherwise it will pull the CUDA build and fail.

> **macOS** — it will run, but slowly (CPU/MPS). Expect minutes per image, not seconds.

### 1.2 Turn on the API (this is the step people miss)

Automatic1111 **does not expose its API by default.** If you skip this, BrandForge will report
`No API on port 7860`.

**Windows easy way** — edit `webui-user.bat` in the folder:

```bat
set COMMANDLINE_ARGS=--api
```

Then **run `webui-user.bat`** — not `launch.py` directly. The batch file is what applies that argument.

**Any OS, or if you want both API and UI:**

```bash
python launch.py --api
```

**Optionally, API only (no web UI), which uses port 7861:**

```bash
python launch.py --nowebui
```

You should see a line in the log like:

```
Running on local URL:  http://127.0.0.1:7860
Uvicorn running on http://127.0.0.1:7861
```

**Optional basic auth.** If you add `--api-auth user:password`, set these in the app's `.env` so
BrandForge can authenticate:

```env
A1111_USER=user
A1111_PASSWORD=password
```

Leave them unset if you are not using `--api-auth`.

### 1.3 Download a model

A model ("checkpoint") must be present before anything can generate.

1. Download a `.safetensors` checkpoint. Good starting points:
   - **SDXL Base** <https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0> — general purpose
   - **DreamShaper** or **Realistic Vision** — from CivitAI, more stylised
   - **Juggernaut XL** — good all-rounder
2. Put the file in:

   | OS | Path |
   | --- | --- |
   | Windows | `stable-diffusion-webui/models/Stable-diffusion/` |
   | Linux/macOS | `models/Stable-diffusion/` |

3. If the WebUI is already open, click **Refresh checkpoints** next to the model dropdown at the top
   left.

SD 1.5 models are ~2 GB; SDXL models are ~7 GB.

### 1.4 Verify the API

Confirm the API answers before touching BrandForge:

```bash
# Should list your checkpoints
curl http://127.0.0.1:7860/sdapi/v1/sd-models

# Should list sampler names
curl http://127.0.0.1:7860/sdapi/v1/samplers
```

Both must return JSON. If they return a 404, the `--api` flag is missing — go back to
[1.2](#12-turn-on-the-api--this-is-the-step-people-miss).

If you are using `--nowebui`, use port **7861** in those URLs.

You can also open <http://127.0.0.1:7860/docs> in a browser for the full interactive API list.

### 1.5 Connect BrandForge

1. Start BrandForge Studio (`start.bat`, or `start-docker.bat` for Docker).
2. Go to **AI Connections** in the sidebar.
3. Under **Image Generation Provider**, choose **Automatic1111 / Forge**.
4. In the **Automatic1111 / Forge** card, set the base URL:
   - `http://localhost:7860` — normal, same machine
   - `http://localhost:7861` — only if you used `--nowebui`
   - `http://192.168.1.50:7860` — if it runs on another machine on your network
5. Press the refresh button next to the URL. The badge should turn **Connected**.
6. Select your **Checkpoint** from the dropdown.
7. Pick a **Sampler** (`DPM++ 2M Karras` is a good default) and set **Steps** (`28` is fine).

That's it. The settings persist in the app's SQLite database, so you do this once.

**Running BrandForge in Docker?** See the note in [Part 5](#part-5--troubleshooting) — `localhost`
inside a container means the container, not your computer.

### 1.6 Generate your first artwork

1. Create or open a project and finish the brand wizard.
2. In the project, generate the brand definition so colours and fonts exist.
3. Open the project and choose asset categories including **Marketing** or **Website**.
   These are the categories the AI image generator handles; logos, icons and stationery always use
   the SVG renderer because they need precise text and geometry.
4. Start generation and watch **Generation Queue**.

The first generation loads the model into VRAM and may take a minute; later ones are much faster.

---

## Part 2 — Forge

**Forge** is a fork of Automatic1111 with better speed and lower VRAM use, and it exposes the
**exact same API**. If you have 6–8 GB of VRAM or find Automatic1111 slow, use Forge instead.

1. Clone and install:
   ```bash
   git clone https://github.com/Haoming02/stable-diffusion-webui-forge
   cd stable-diffusion-webui-forge
   python -m venv venv
   venv\Scripts\activate     # or: source venv/bin/activate
   pip install -r requirements.txt
   ```
2. Run it with the API flag:
   ```bash
   python launch.py --api
   ```
3. Use **the same instructions** as Automatic1111 from [1.3](#13-download-a-model) onward — the URL,
   the ports (7860 / 7861), the checkpoint folder and every API endpoint are identical.

---

## Part 3 — ComfyUI

Use this if you already use ComfyUI and want node-level control.

1. Install:
   ```bash
   git clone https://github.com/comfyanonymous/ComfyUI
   cd ComfyUI
   python -m venv venv
   venv\Scripts\activate     # or: source venv/bin/activate
   pip install -r requirements.txt
   python main.py --listen
   ```
2. Put a checkpoint in `ComfyUI/models/checkpoints/`.
3. In ComfyUI, open the settings (gear icon) and enable **"Save (API Format)"**.
4. Export your workflow as JSON (the API format, not the UI format).
5. In BrandForge go to **Model Settings → Import Workflow**, paste the JSON, name it, and assign it a
   category.
6. Set it as the workflow for the category you want it to handle.
7. Back in **AI Connections**, set the provider to **ComfyUI**.

Unlike Automatic1111, ComfyUI needs a workflow imported before it can generate anything.

---

## Part 4 — Choosing the provider in the app

Open **AI Connections** and pick one of the three provider tiles:

- **Built-in SVG renderer** — the default. Nothing to install. Produces vector logos, icons,
  patterns, stationery and social layouts. Marketing artwork falls back to a simple SVG layout.
- **Automatic1111 / Forge** — free and local. Requires the service running with `--api`.
- **ComfyUI** — free and local. Requires an imported workflow and a checkpoint.

The choice is stored in settings and applies to new generation jobs. Changing it never affects
assets you already generated.

If a selected provider is not reachable at generation time, BrandForge falls back to the SVG renderer
rather than failing the whole job, and records that in the job's step description.

---

## Part 5 — Troubleshooting

### "No API on port 7860"

Automatic1111 is running without the `--api` flag. On Windows make sure you launched
**`webui-user.bat`** with `set COMMANDLINE_ARGS=--api` inside it, or run
`python launch.py --api`.

### "Connection refused"

The service is not running. Check that you can open <http://127.0.0.1:7860> in your browser.

### No checkpoints listed

No `.safetensors` file in `models/Stable-diffusion/`. Download one and press **Refresh checkpoints**.

### Wrong port

| Launch command | Port |
| --- | --- |
| `python launch.py --api` | 7860 |
| `python launch.py --nowebui` | 7861 |

### Running BrandForge in Docker

A container's `localhost` is the container itself, so it cannot see a service running on your
computer. Set the environment variable in `docker-compose.yml`:

```yaml
environment:
  AUTOMATIC1111_URL: "http://host.docker.internal:7860"
```

`host.docker.internal` is mapped to your machine by Docker Desktop on Windows and macOS. On Linux
you may need Docker Engine 20.10+, or replace it with your computer's LAN IP such as
`http://192.168.1.50:7860`. The compose file already adds the `host-gateway` mapping for you.

Then in **AI Connections**, click the refresh button to re-test.

### "Out of memory" / "CUDA out of memory"

The model does not fit in your VRAM. Add these to `COMMANDLINE_ARGS`:

```bat
set COMMANDLINE_ARGS=--api --medvram
```

`--lowvram` is the next step down. Both are slower but work on smaller cards.

### Generation runs but looks wrong

- Increase **Steps** (30–40) for more detail.
- Lower **CFG scale** in the app's generation settings if output looks burnt out (default 7).
- Try a different sampler; `DPM++ 2M Karras` and `Euler a` are reliable defaults.

---

## Part 6 — Hardware & quality expectations

| VRAM | Result | Suggested flags |
| --- | --- | --- |
| 4 GB | Works, SD 1.5 only, slow | `--lowvram` |
| 6–8 GB | SDXL at reduced speed | `--medvram` |
| 12 GB+ | SDXL comfortable, good quality | none |
| 24 GB+ | SDXL and larger models at full detail | none |

Without a GPU everything still works, just slowly — minutes per image on CPU.

### What the image model will *not* do

Diffusion models cannot reliably render exact spelling or accurate small text. BrandForge therefore
never asks them to: **all text in generated assets is drawn afterwards by the app** using your brand
typography and colours. This is why logos, business cards, stationery and social layouts are always
built by the SVG renderer instead of the model — their text has to be perfect.

This is also why the app sends a negative prompt telling the model to avoid `text, letters, words,
watermark` — you get a clean image, then the app puts your real words on top.
