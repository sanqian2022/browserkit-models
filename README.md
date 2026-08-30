# browserkit-models

ONNX models for [BrowserKit](https://github.com/sanqian2022/BrowserKit)'s image tools, which run
entirely in the browser. This repository exists so the model bytes can be fetched from public CDNs
at run time instead of being served by the site itself.

Nothing here is code. It is model weights, a checksum manifest, and the upstream licences.

## Why the files are split

`migan-512-places2` is a single 26.78 MB ONNX file upstream. It is stored here as two ~13.4 MB parts
because CDNs apply per-file size limits, and because the deploy target of the consuming site rejects
any single file over 25 MiB. The consumer downloads the parts in parallel and joins them in order.

`manifest.json` records, for every model, each part's path and byte count plus the SHA-256 of both the
individual parts and the joined result. The consumer verifies these after downloading, so a CDN that
serves truncated, stale, or wrong bytes fails loudly instead of producing a confusing error deep
inside the inference runtime. `manifest.json` is small (about 1.5 KB) and doubles as the probe file
used to find a reachable CDN quickly.

## Contents

| Model | Files | Joined size | Upstream | Licence |
| --- | --- | --- | --- | --- |
| `slimsam-77-vision-encoder` | 1 | 8.47 MiB | [Xenova/slimsam-77-uniform](https://huggingface.co/Xenova/slimsam-77-uniform) | Apache-2.0 |
| `slimsam-77-decoder` | 1 | 4.68 MiB | same | Apache-2.0 |
| `migan-512-places2` | 2 | 26.78 MiB | [andraniksargsyan/migan](https://huggingface.co/andraniksargsyan/migan) | MIT |

Full licence texts, with attribution and paper references, are in `LICENSES/`.

RMBG-1.4 is deliberately **not** here. It is released under the `bria-rmbg-1.4` licence, which is
source-available for non-commercial use only, so redistributing its weights is not permitted.
Consumers must fetch it from BRIA's own repository.

## How the files are addressed

The layout is chosen so the same relative path works whether the files are served from this
repository or from the npm package built from it. Only the base URL changes:

```
https://cdn.jsdelivr.net/gh/sanqian2022/browserkit-models@models-v1/models/<file>
https://registry.npmmirror.com/browserkit-models/1.0.0/files/models/<file>
https://unpkg.com/browserkit-models@1.0.0/models/<file>
https://cdn.jsdelivr.net/npm/browserkit-models@1.0.0/models/<file>
```

Always address an immutable reference: a git tag for the jsDelivr `/gh/` form, an exact version for
the npm forms. A moving reference such as `@main` is cached for hours and can hand out parts from
different revisions, which the manifest would then reject.

## Updating a model

1. Replace the files under `models/`, keeping parts at or below 15 MB.
2. Regenerate `manifest.json` so the byte counts and hashes match.
3. Bump `version` in `package.json` and `manifest.json`.
4. Commit, push, then create and push a new tag (`models-v2`, and so on).
5. Point the consuming site at the new tag or version. Old tags keep working, so a deployed site is
   never broken by an update here.

Do not enable Git LFS for this repository. jsDelivr does not resolve LFS pointers; it would serve the
small pointer file instead of the model, and the failure looks like a corrupt model rather than a
configuration mistake.
