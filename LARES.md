# Proofy runbook for Lares

This file is written for **Lares** (the Olares AI assistant) to install, update and check **Proofy** with `olares-cli`,
and to bring the device to the **recommended setup** (section 1). A person can read it too; the illustrated manual for
people is separate.

> 日本語の利用者へ：Lares に「https://github.com/ryosk7/proofy-releases を見てアップデートして」と頼んでください。
> Lares がこの手順書に沿って状態を確認し、足りないモデルの導入と Proofy の更新を提案します。
> 端末を変える操作は、Lares が内容を示したあとに、あなたが承認してから行われます。

## 0. Rules (read first)

1. **Report in the user's language** (Japanese for this office). Show the exact command before running anything that changes
   the device, and wait for the user's approval. One approval may cover a whole section's commands if the user says so.
   Read-only commands (`list`, `get`, `curl` to GitHub) need no approval.
2. **Never** uninstall Proofy or any other app, never delete uploaded charts, never stop apps you were not asked to stop,
   never change GPU / accelerator settings, never edit app env values. Never stop or uninstall a running embedding app
   (EmbeddingGemma or another): other apps may still use it. If a step seems to need one of these, stop and ask.
3. **Stop at the first failure.** Do not retry with different flags or another source. Report the command, its output
   and the state from `olares-cli market list --mine`.
4. **Idempotent**: if a step's goal is already met (app running, version already installed), skip it and say so.
5. Use only the package source in section 2. Do not download Proofy from anywhere else.
6. Installing apps and uploading custom packages needs an **Olares administrator** and Lares' **write** (or full)
   permission. If a command is denied, stop and tell the user which permission is missing.
7. **No downgrades.** Proofy is never installed at a version older than the installed one (Olares refuses it as well).
   If an update misbehaves, report it and wait for a newer release.

## 1. The recommended setup

Bring the device to this setup. "Required" and "recommended" apps are both installed by default; "keep" apps are never
installed for a new setup and are left as they are.

| Olares app id | Title in Market | Level | Role |
|---|---|---|---|
| `router` | Router | required | AI gateway; Proofy calls every model through it. Proofy cannot install without it. |
| `llamacppqwen3827bggufv3` | Qwen3.8-27B (llama.cpp) | required | Chat model with **vision** and **reasoning**: proofreading, drafting and reading drawings. Needs an NVIDIA GPU with about 23 GiB VRAM. |
| `llamacppllmbasev3` cloned as "Qwen3 Embedding 8B (llama.cpp)" (section 4) | llama.cpp Engine Base, Qwen3 Embedding 8B Q4_K_M on the CPU | recommended | Embedding model that picks manual items for long manuals; best multilingual retrieval. Router lists it as `Olares/Qwen/Qwen3-Embedding-8B-GGUF:Q4_K_M`. Runs on the CPU (~6 GiB RAM), leaving the GPU to Qwen3.8. |
| `bgererankerv2m3` | BGE-Reranker-v2-m3 | recommended | Reranks manual items before proofreading (Intel iGPU, compute mode `intel`). Without it Proofy uses a simpler selection. |
| `embeddinggemmav3` | EmbeddingGemma | keep if installed | Lighter embedding model of older setups. Do not install it for a new setup; leave it as it is when installed. |
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

From the first output, read `state` and `version` of every app in section 1. The Qwen3 Embedding app is a clone: its
`title` contains `Qwen3 Embedding 8B (llama.cpp)` and its `name` looks like `llamacppllmbasev314c33e`. An older Ollama
`Qwen3 Embedding 8B` row is no longer used and may stay stopped. From the second, read the latest version (`tag_name` without the leading `v`).

Report a table like this, then propose the next section:

| App | Level | Installed | State | Note |
|---|---|---|---|---|
| Router | required | 1.1.3 | running | ok |
| Qwen3.8-27B | required | 1.0.21 | running | ok |
| Qwen3 Embedding 8B (llama.cpp) | recommended | — | — | missing → section 4 |
| BGE-Reranker-v2-m3 | recommended | 1.0.8 | running | ok |
| EmbeddingGemma | keep | 1.1.5 | stopped | left as it is |
| Proofy | — | 2.6.17 (latest 2.6.18) | running | update → section 7 |

Decide, in this order:
1. Any state ending in `Failed` (for example `installFailed`, `upgradeFailed`) → stop and report; do not try to fix it.
2. A required or recommended app is missing → section 4.
3. A required or recommended app is `stopped` → section 5. A stopped "keep" app stays stopped; do not ask about it.
4. Proofy is missing → section 6.
5. Proofy is older than the latest release → section 7.
6. Everything required and recommended is running and Proofy is the latest → report "準備完了", then section 9.

## 4. Install a missing model app (needs approval)

Install the missing apps of section 1 in this order: `router`, `llamacppqwen3827bggufv3`, the Qwen3 Embedding app,
then `bgererankerv2m3`.

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
olares-cli market install bgererankerv2m3 --compute-mode intel --watch --watch-timeout 30m
```

- The Qwen model downloads tens of GB; a long wait is normal.
- If a compute mode above is refused (the device has no Intel iGPU, for example), stop and report the modes the error
  lists; do not pick another mode yourself. The user can install that app from Market in the browser instead.
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
Then tell the user to open Proofy from the Launchpad. Proofy picks its models by itself from what Router lists;
nothing is configured by hand (do not tell the user to save 設定 → AI の設定).

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
- Never downgrade (rule 7). If the installed version is newer than the latest release, report it and stop.
- Success: `proofy` shows `state: running` and the new `version`.

## 8. After any change

Run section 3 again and report the final table. Mention anything that is not `running`. Then section 9.

## 9. What Proofy finishes by itself (tell the user)

Nothing here needs Lares or a command.

- While nobody has saved 設定 → AI の設定, Proofy uses the best models Router lists (Qwen3 Embedding 8B over
  EmbeddingGemma, BGE-Reranker when present). After a model app is added it switches by itself and re-embeds the
  registered manuals in the background within about 15 minutes; meanwhile manual items are picked by keywords.
- If an administrator saved 設定 → AI の設定, that choice is kept. Tell the user that the administrator can choose the
  Qwen3 Embedding model there if they want the recommended setup.

## Notes

- There may be a second, old entry for Proofy (app `patentproof`, or a `proofy` row stuck at version 2.0.0 in a failed
  state). Leave it alone; do not uninstall or upgrade it. Report it only as "旧版が残っています".
- Proofy's own screen shows **AI 接続中** when Router and the chat model are reachable. Lares cannot open Proofy's
  screen; ask the user to confirm it after an install or update.
