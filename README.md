# browserkit-models

ONNX models for [BrowserKit](https://github.com/sanqian2022/BrowserKit)'s image tools, which run
entirely in the browser. This repository exists so the model bytes can be fetched from public CDNs
at run time instead of being served by the site itself.

Nothing here is code. It is model weights, a checksum manifest, and the upstream licences.

## Why the files are split

Upstream, `migan-512-places2` is a single 26.78 MB file, `isnet-general-int8` a single 42.18 MB
file, and `resnet50-v1-7-int8` a single 24.93 MB file. They are stored here in parts of about
14-15 MB because CDNs apply per-file size limits. The consumer downloads the parts in parallel and
joins them in order.

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
| `isnet-general-int8` | 3 | 42.18 MiB | [xrds/isnet-general-onnx-int8](https://huggingface.co/xrds/isnet-general-onnx-int8) | MIT |
| `resnet50-v1-7-int8` | 2 | 24.93 MiB | [onnx/models ResNet50 v1](https://github.com/onnx/models/tree/main/validated/vision/classification/resnet) | Apache-2.0 |

Full licence texts, with attribution and paper references, are in `LICENSES/`.

`isnet-general-int8` replaced RMBG-1.4 for background matting. RMBG-1.4 is released under the
`bria-rmbg-1.4` licence, which is source-available for non-commercial use only and does not permit
redistribution, so it could never be served from here; that made the feature unusable on networks
where huggingface.co is blocked. IS-Net is the architecture RMBG-1.4 was fine-tuned from, is the same
size once quantized, runs at the same speed, and is MIT licensed.

## How the files are addressed

The layout is chosen so the same relative path works whether the files are served from this
repository or from the npm package built from it. Only the base URL changes:

```
https://cdn.jsdelivr.net/gh/sanqian2022/browserkit-models@models-v3/models/<file>
https://registry.npmmirror.com/browserkit-models/1.2.0/files/models/<file>
https://unpkg.com/browserkit-models@1.2.0/models/<file>
https://cdn.jsdelivr.net/npm/browserkit-models@1.2.0/models/<file>
```

Always address an immutable reference: a git tag for the jsDelivr `/gh/` form, an exact version for
the npm forms. A moving reference such as `@main` is cached for hours and can hand out parts from
different revisions, which the manifest would then reject.

## Updating a model

1. Replace the files under `models/`, keeping parts at or below 15 MB.
2. Regenerate `manifest.json` so the byte counts and hashes match.
3. Bump `version` in `package.json` and `manifest.json`.
4. Commit, push, then create and push a new tag (`models-v3`, and so on).
5. Point the consuming site at the new tag or version. Old tags keep working, so a deployed site is
   never broken by an update here.

Do not enable Git LFS for this repository. jsDelivr does not resolve LFS pointers; it would serve the
small pointer file instead of the model, and the failure looks like a corrupt model rather than a
configuration mistake.
