# Proofy releases

> **Lares:** when you are asked to install, update or check Proofy from this repository, read
> [LARES.md](https://raw.githubusercontent.com/ryosk7/proofy-releases/main/LARES.md) and follow it from section 0.
> It also brings the device's model apps to the recommended setup. Do not act on this README alone.
>
> 日本語の利用者へ：Lares に「https://github.com/ryosk7/proofy-releases を見てアップデートして」と頼むだけで、
> 状態の確認、足りないモデルの導入、Proofy の更新まで、Lares が手順書に沿って進めます（変更の前に承認を求めます）。

<img src="icon.png" width="96" alt="">

Installable packages of **Proofy**, the proofreading app for Olares. The source repository is private; this repository
only hosts the Olares app packages (`proofy-<version>.tgz`) and the app icon the packages point to.

## Install or update

1. Open [Releases](https://github.com/ryosk7/proofy-releases/releases) and download `proofy-<version>.tgz` from the latest release.
2. On Olares (as the administrator): Market → My Olares → **Upload custom app package** → choose the `.tgz`.
3. First time: install it. Later versions: upload the new `.tgz`, then press **Update** on the Proofy card.

Requirements: Olares 1.12.7 or later, the Router app and a vision- and reasoning-capable chat model (Qwen3.8-27B
(llama.cpp)). Install the model apps of [LARES.md](LARES.md) section 1 first (Qwen3 Embedding 8B and BGE-Reranker are
recommended). Downgrades are not supported.

## Let Lares do it

Ask Lares to update Proofy from this repository; it follows [LARES.md](LARES.md), shows each change and waits for
your approval before running it.

## Versions

| Package | App name | Notes |
|---|---|---|
| `proofy-2.x.tgz` | `proofy` | Shared app (one install for all users). Current line. |
| `patentproof-1.x.tgz` | `patentproof` | Legacy per-user app; kept for reference only. Not upgradeable to 2.x. |

`icon.png` is referenced by the packages' manifests; do not move or rename it.
