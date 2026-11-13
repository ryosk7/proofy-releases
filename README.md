# Proofy releases

<img src="icon.png" width="96" alt="">

Installable packages of **Proofy**, the proofreading app for Olares. The source repository is private; this repository
only hosts the Olares app packages (`proofy-<version>.tgz`) and the app icon the packages point to.

## Install or update

1. Open [Releases](https://github.com/ryosk7/proofy-releases/releases) and download `proofy-<version>.tgz` from the latest release.
2. On Olares (as the administrator): Market → My Olares → **Upload custom app package** → choose the `.tgz`.
3. First time: install it. Later versions: upload the new `.tgz`, then press **Update** on the Proofy card.

Requirements: Olares 1.12.7 or later, the Router app, and a vision- and reasoning-capable chat model
(Qwen3.8-27B (llama.cpp)); EmbeddingGemma is recommended.

## Let Lares do it

[LARES.md](LARES.md) is a runbook for the Olares assistant **Lares**: give it the link
`https://github.com/ryosk7/proofy-releases/blob/main/LARES.md` and ask it to check, install or update Proofy following it.
Lares shows each change and waits for your approval before running it.

## Versions

| Package | App name | Notes |
|---|---|---|
| `proofy-2.x.tgz` | `proofy` | Shared app (one install for all users). Current line. |
| `patentproof-1.x.tgz` | `patentproof` | Legacy per-user app; kept for reference only. Not upgradeable to 2.x. |

`icon.png` is referenced by the packages' manifests; do not move or rename it.
