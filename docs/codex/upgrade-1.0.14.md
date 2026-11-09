# Codex upgrade manifest — DSH Go 1.0.14

## Baseline and delivery

- Previous Codex revision: `ec81e9189f429672f1c38165339625f43c8406b0` (DSH Go 1.0.4).
- Target: upstream release `v1.0.14`, commit `f36bf91b0736e81ef40c2a7040b2ec337ca1ab40` (annotated tag object `4dce2a65cdbf35bef5c8775e90f8378b89b1e6a3`).
- Delivery branch: `feat/codex-proxy-v1.0.14`; no new PR and no update to the existing PR branch.
- Preserve the separate Codex preview application ID, signing lineage and fork update source.
- Reuse upstream screens; keep protocol adaptation in the Codex packages. Do not claim DSH-only features work with Codex.

## Compatibility checklist

| Area | Required behavior / inspection |
| --- | --- |
| SSH | Host verification, account identity, reconnect, connection ownership and saved host migration |
| Sessions | List, history, new chat, rename, archive/restore, unchanged prompt retention after final events |
| New-chat home | Model and reasoning selection must reach the first Codex turn; unsupported agent/permission controls must not issue DSH RPCs |
| Streaming | Partial/final messages, tool cards, interrupt, steer, approvals and questions, including pet replies |
| Attachments | SFTP upload, Codex image input, file references, history links, preview, download/save/share; bounded limits |
| Search | Current-host isolation, title and rollout content search, bounded/incomplete results |
| Usage | Session tokens/context and account rate limits; unknown cost stays unknown |
| New upstream UI | Image viewer/retry and video/file rendering may be reused when transport supports them |
| DSH-only entry points | Plugins, agent presets, phone control, companion full mode, terminal, session files and destructive session actions must be inspected and gated where unsupported |
| Runtime | Package the matching 1.0.14 runtime/assets rather than silently reusing 1.0.2 payloads |
| Distribution | ARM64 release APK; no personal paths, keys, logs or device fixtures; verify signature and package identity |

## Validation plan

1. Inspect all overlapping changes and resolve merge conflicts against the release tag.
2. Run static analysis, Codex backend/client tests and targeted UI regression tests; add tests for changed behavior.
3. Build with bounded CPU/memory and an available-memory stop threshold; record resource evidence outside Git.
4. Verify APK metadata, signing lineage and privacy markers, then push the delivery branch and provide the APK.
5. Clearly separate automated verification from phone checks that still need to be performed after installation.

## Results

Pre-merge inspection found mandatory adaptations: `attachment` now takes an
`ImageAttachment` reference; model selection uses `setModelResult`; new-chat
preset/permission setup is DSH-specific; the upstream 200 MiB picker must not
overpromise the Codex SFTP transport's 30 MiB limit. Implementation and validation
are pending.
