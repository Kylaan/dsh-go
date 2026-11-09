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

- Adopted the 1.0.14 chat/composer and attachment gallery. On phones the model picker follows upstream into the expanded composer.
- Adapted image reads to `ImageAttachment` references, including images from history, tools and image generation.
- Implemented `setModelResult` via Codex model selection. A new conversation's model and reasoning effort reach its first turn; no global server defaults are written.
- Hidden DSH preset/permission/plan/plugin/phone-control controls in Codex composer entry points. Restored DSH-only new-chat picks do not block a Codex first message. Permanent deletion stays unavailable through the upstream transport capability check.
- Preserved Codex's 30 MiB file cap in picker metadata checks, streamed reads and backend upload validation; DSH retains 200 MiB. Images retain 20 MiB; history previews/save/share use upstream components.
- Retained streaming completion, user/tool history, interrupt/steer, approvals/questions, host-scoped search and token/quota views. No protocol changes to DSH Remote or the runtime bridge were introduced by the Codex adaptation.
- Runtime reused from the official 1.0.14 ARM64 APK after SHA-256 verification: `0a2e2496db46ce4956eaaa313a80edfa226846b040bac2bbfc56cb3081381c84`. Runtime version: `r15-dsh0.2.2-rc.1-node24.22.1-ubuntu24.04.5-2e998e3bf28f`; its rootfs hash was also verified. This is reuse of the released runtime, not a local runtime rebuild.

### Automated validation

| Check | Result |
| --- | --- |
| `cd app && flutter analyze --no-pub` | Passed |
| Focused Flutter regressions: chat/new-chat, attachments/gallery, model selection, search/archive, usage, notifications, links and session ownership | 409 passed |
| `flutter test --no-pub test/remote/codex_integration_test.dart` | 7 passed, including preserved messages after completion, first-send model/effort and restored unsupported picks |
| `cd packages/codex_backend && dart analyze && dart test` | 27 passed |
| `cd packages/codex_client && dart analyze && dart test` | 12 passed |
| `cd packages/dsh_client && dart analyze && dart test` | 816 passed; 31 skipped (explicit live/disposable host prerequisites) |
| `python3 -m unittest discover -s tooling -p 'test_build_number.py'` | 12 passed |
| `python3 -m unittest discover -s tooling -p 'test_release_assets.py'` | 15 passed |
| `python3 app/tool/l10n_merge.py --check` | Passed: 60 parts, 6120 keys |
| `python3 app/tool/guide_sync.py --check` | Passed: 26 files |
| Negative controls in a disposable backend copy | Both new model-result and upload-cap tests fail when their corresponding safeguards are removed |

The initial analysis ran before dependency installation finished and failed on
missing packages; analysis after dependency setup passed. The previous chat
screenshot differed because of the upstream layout. Its actual render was
inspected and the synthetic golden regenerated; the integration test then passed
without updating goldens. No assertions were removed to accommodate failures.

### Delivery and remaining phone checks

- Built ARM64 release `1.0.14-codex-preview.1`, version code `11092159` (204,335,813 bytes).
- APK SHA-256: `209fb8f74ce751dfd52526a7f8cd59726199422c27f4afc3c93ef547b4020988`.
- Package identity remains `io.github.ccch1mneyyy.dsh_app.codexpreview`. Existing signing lineage verified for Android 8 and Android 9+; APK alignment verification passed.
- Known personal-path/address/account markers and credential patterns were absent from the fork source additions and 638 scanned APK entries. The bundled rootfs matched the official release hash. This bounded scan is not an exhaustive secret audit; private build logs, symbols and signing material remain outside Git.
- Build completed in 18 minutes 27 seconds under a 6 GiB hard memory cap and a two-core CPU quota. Peak cgroup memory was 5070 MiB, cgroup swap peak was zero, and system available memory stayed at or above 6895 MiB. The soft memory threshold was raised from 4 to 5 GiB during the build; one confirmed-idle Kotlin daemon belonging to this build was stopped to release memory.
- The build logged an AAPT2 idle-daemon shutdown timeout; Gradle cleaned it up and completed successfully. Android release lint reported no issues.

No Android device is attached. Installation over the earlier preview, real
SSH/SFTP transfers/reconnect, gallery save/share and video playback must still be
checked on the phone. New DSH-only companion/phone-control/terminal functionality
is not claimed as Codex support. The complete upstream test suite and live DSH
hosts were not exercised. This branch is an independent preview, not an upstream
PR submission.
