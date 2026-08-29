# Sportgramme — AWS

The **cloud landscape** of the Sportgramme platform: the AWS resources and
functions that the browser surfaces depend on but never call directly.

| Folder | Capability | Was |
|---|---|---|
| [media-pipeline/](media-pipeline/) | Batch upload and cloud delivery of stills, video and audio for accredited contributors — staging, pre-flight, just-in-time authorisation, direct device-to-cloud transfer, trust-tiered routing, moderation gate | `sg-media-pipeline` |
| [aws-integration/](aws-integration/) | The shared AWS landscape every surface leans on — storage targets, CDN / image delivery, the proofing and quarantine stores, feed-assembly and moderation functions, the token broker and the managed secret store | `AWS-Integration` |

Every *"Depends on the AWS landscape"* reference across the other repositories
resolves here. Resources are **described, not disclosed** — no endpoints, keys, or
hostnames.

## The four surfaces

| Surface | Repository | What it is |
|---|---|---|
| AWS | **this repo** | Cloud landscape — media pipeline, storage, CDN, delivery functions |
| Web | [sportgramme-web](https://github.com/sportgramme/sportgramme-web) | The browser surface — public site, widgets, contributor tools |
| API | [sportgramme-api](https://github.com/sportgramme/sportgramme-api) | Internal API landscape and the syndication / distribution surface |
| On-Prem | [sportgramme-on-prem](https://github.com/sportgramme/sportgramme-on-prem) | Restricted back office — the access-control model and its operator console |

See
[ARCHITECTURE.md](https://github.com/sportgramme/sportgramme/blob/main/ARCHITECTURE.md)
for how the surfaces fit together and the consolidation rationale, and the shared
[glossary](https://github.com/sportgramme/sportgramme/tree/main/Glossary).