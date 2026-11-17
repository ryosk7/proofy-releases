# Proofy runbook for Lares

This file is written for **Lares** (the Olares AI assistant) to install, update and check **Proofy** with `olares-cli`.
A person can read it too; the illustrated manual for people is separate.

> 日本語の利用者へ：Lares にこのファイルの URL を渡し、「この手順書に沿って Proofy の状態を確認して」と頼んでください。
> インストールや更新は、Lares が確認結果と実行内容を示したあとに、あなたが承認してから行われます。

## 0. Rules (read first)

1. **Report in the user's language** (Japanese for this office). Show the exact command before running anything that changes
   the device, and wait for the user's approval. Read-only commands (`list`, `get`, `curl` to GitHub) need no approval.
2. **Never** uninstall Proofy or any other app, never delete uploaded charts, never stop apps you were not asked to stop,
   never change GPU / accelerator settings, never edit app env values. If a step seems to need one of these, stop and ask.
3. **Stop at the first failure.** Do not retry with different flags or another source. Report the command, its output
   and the state from `olares-cli market list --mine`.
4. **Idempotent**: if a step's goal is already met (app running, version already installed), skip it and say so.
5. Use only the package source in section 2. Do not download Proofy from anywhere else.
6. Installing apps and uploading custom packages needs an **Olares administrator** and Lares' **write** (or full)
   permission. If a command is denied, stop and tell the user which permission is missing.

## 1. What Proofy needs

| Olares app id | Title in Market | Required | Role |
|---|---|---|---|
| `router` | Router | yes | AI gateway; Proofy calls every model through it. Proofy cannot install without it. |
| `llamacppqwen3827bggufv3` | Qwen3.8-27B (llama.cpp) | yes | Chat model with **vision** and **reasoning**: proofreading, drafting and reading drawings. Needs an NVIDIA GPU with about 23 GiB VRAM. |
| `llamacppllmbasev3` cloned as "Qwen3 Embedding 8B (llama.cpp)" (section 4) | llama.cpp Engine Base, Qwen3 Embedding 8B Q4_K_M on the CPU | recommended | Embedding model that picks manual items for long manuals; best multilingual retrieval. Router lists it as `Olares/Qwen/Qwen3-Embedding-8B-GGUF:Q4_K_M`. Runs on the CPU (~6 GiB RAM), leaving the GPU to Qwen3.8. |
| `embeddinggemmav3` | EmbeddingGemma | alternative | Lighter embedding model; use it instead when the GPU has no room for Qwen3 Embedding. One embedding model is enough. |
| `proofy` | Proofy | — | The app itself, installed from an uploaded package (source `upload`). |

Olares must be **1.12.7 or later** (`olares-cli --version` prints the CLI version; the OS version is shown in Settings).

## 2. Package source

- Releases: `https://github.com/ryosk7/proofy-releases/releases`
- Latest release (JSON): `https://api.github.com/repos/ryosk7/proofy-releases/releases/latest`
  - `tag_name` is `v<version>` (for example `v2.6.16`).
  - The asset `proofy-<version>.tgz` is the package; its `browser_download_url` is the download link.
- Ignore assets named `patentproof-*.tgz` (legacy app, not upgradeable to Proofy 2.x).

## 3. Check (read-only) — always run this first

```sh
olares-cli market list --mine -o json
curl -s https://api.github.com/repos/ryosk7/proofy-releases/releases/latest
```

From the first output, read `state` and `version` of `router`, `llamacppqwen3827bggufv3`, `proofy`, and of the embedding
model: a row whose `title` contains `Qwen3 Embedding 8B (llama.cpp)` (its `name` is a clone such as
`llamacppllmbasev314c33e`), or else `embeddinggemmav3`. An older Ollama `Qwen3 Embedding 8B` row is no longer used. Either embedding model counts as present. From the second, read the latest version (`tag_name` without the leading `v`).

Report a table like this, then propose the next section:

| App | Installed | State | Note |
|---|---|---|---|
| Router | 1.1.3 | running | ok |
| Qwen3.8-27B | 1.0.20 | running | ok |
| Qwen3 Embedding 8B (llama.cpp) | 1.2.40 | running | ok |
| Proofy | 2.6.16 (latest 2.6.16) | running | up to date |

Decide:
- A required app is missing → section 4.
- A required app is `stopped` → section 5.
- Proofy is missing → section 6.
- Proofy is older than the latest release → section 7.
- Everything is running and Proofy is the latest → report "準備完了" and stop.
- Any state ending in `Failed` (for example `installFailed`, `upgradeFailed`) → stop and report; do not try to fix it.

## 4. Install a missing model app (needs approval)

Install only the apps from section 1 that are missing, in this order: `router`, then `llamacppqwen3827bggufv3`, then
the embedding model (Qwen3 Embedding 8B; EmbeddingGemma only if the user asks for the lighter one).

```sh
olares-cli market install router --watch
olares-cli market install llamacppqwen3827bggufv3 --compute-mode nvidia --watch --watch-timeout 1h
olares-cli market clone llamacppllmbasev3 --title "Qwen3 Embedding 8B (llama.cpp)" --compute-mode nvidia \
  --env "MODEL_SOURCE=hf://Qwen/Qwen3-Embedding-8B-GGUF --include Qwen3-Embedding-8B-Q4_K_M.gguf" \
  --env "MODEL_NAME=Qwen/Qwen3-Embedding-8B-GGUF:Q4_K_M" \
  --env MODEL_MODE=embedding --env MODEL_SUPPORTS=none \
  --env "ENGINE_ARGS=-c 2048 -ub 2048 -b 2048 --pooling last -ngl 0 -dev none" \
  --watch --watch-timeout 1h
# The engine flags live on the model card, which wins over ENGINE_ARGS after the first start; set them there too:
olares-cli router model spec edit 'Olares/Qwen/Qwen3-Embedding-8B-GGUF:Q4_K_M' \
  --engine-args "-c 2048 -ub 2048 -b 2048 --pooling last -ngl 0 -dev none" --yes
# lighter alternative, only if asked:
# olares-cli market install embeddinggemmav3 --watch --watch-timeout 30m
```

- The Qwen model downloads tens of GB; a long wait is normal.
- If the install asks for a GPU binding or compute mode you cannot answer from this file, stop and ask the user to
  install that app from Market in the browser instead.
- Success: the app shows `state: running` in `olares-cli market list --mine`.

## 5. Resume a stopped app (needs approval)

```sh
olares-cli market resume <app id> --watch
```

Only resume apps listed in section 1. If the resume needs a `--compute-binding`, stop and ask the user to resume it from
Market → My Olares in the browser.

## 6. Install Proofy (needs approval)

Preconditions: `router` and `llamacppqwen3827bggufv3` are `running` (section 3).

```sh
# 1. Download the latest package (replace <version>, e.g. 2.6.16)
curl -fL -o proofy-<version>.tgz \
  https://github.com/ryosk7/proofy-releases/releases/download/v<version>/proofy-<version>.tgz

# 2. Upload it to the Market's "Upload" source
olares-cli market upload proofy-<version>.tgz

# 3. Install from that source
olares-cli market install proofy -s upload --version <version> --watch
```

Success: `proofy` shows `state: running` and `version: <version>` with `source: upload`.
Then tell the user: open Proofy from the Launchpad, and (administrator) check **設定 → AI の設定**:
文章を扱うモデル = `Olares/unsloth/Qwen3.8-27B-GGUF:UD-Q4_K_XL`, 検索用の埋め込みモデル = `Olares/Qwen/Qwen3-Embedding-8B-GGUF:Q4_K_M`
(or `Olares/embeddinggemma-300m`), then 保存. Proofy re-embeds the registered manuals by itself after the embedding model
changes (a few minutes).

## 7. Update Proofy (needs approval)

Preconditions: `proofy` is `running` or `stopped`, and its version is older than the latest release.

```sh
curl -fL -o proofy-<version>.tgz \
  https://github.com/ryosk7/proofy-releases/releases/download/v<version>/proofy-<version>.tgz
olares-cli market upload proofy-<version>.tgz
olares-cli market upgrade proofy -s upload --version <version> --watch
```

- Tell the user before starting that Proofy may be unavailable for a few minutes. Data (manuals, history, settings) is kept.
- App settings entered in Olares (for example the optional Langfuse host and keys) are kept by an update; never change them.
- Do not start an update while a proofreading run is in progress (Proofy's history shows it as running); wait for it to finish.
- Never downgrade. If the installed version is newer than the latest release, report it and stop.
- Success: `proofy` shows `state: running` and the new `version`.

## 8. After any change

Run section 3 again and report the final table. Mention anything that is not `running`.

## Notes

- There may be a second, old entry for Proofy (app `patentproof`, or a `proofy` row stuck at version 2.0.0 in a failed
  state). Leave it alone; do not uninstall or upgrade it. Report it only as "旧版が残っています".
- Proofy's own screen shows **AI 接続中** when Router and the chat model are reachable. Lares cannot open Proofy's
  screen; ask the user to confirm it after an install or update.
