# Sportgramme — CLOUD

The **cloud landscape** of the Sportgramme platform includes 
- AWS resources
- AZURE resources
- GOOGLE cloud
- SG cloud

enabling sophisticated functions adn data processing, browser surfaces depend on but never call directly.

| Folder | Capability | Was |
|---|---|---|
| [media-pipeline/](media-pipeline/) | Batch upload and cloud delivery of stills, video and audio for accredited contributors — staging, pre-flight, just-in-time authorisation, direct device-to-cloud transfer, trust-tiered routing, moderation gate | `sg-media-pipeline` |
| [aws-integration/](aws-integration/) | The shared AWS landscape every surface leans on — storage targets, CDN / image delivery, the proofing and quarantine stores, feed-assembly and moderation functions, the token broker and the managed secret store | `AWS-Integration` |



## The four surfaces

| Surface | Repository | What it is |
|---|---|---|
| AWS | **this repo** | Cloud landscape — media pipeline, storage, CDN, delivery functions |
| GOOGLE | **this repo** | Cloud landscape — Validations, Geospatial processing and calculations |
| AZURE | **this repo** | Cloud landscape — Databricks|
| SG | **this repo** | Private Cloud landscape - LLM Refinement, language translations, Video analyses, Audio  transcription, NLP, tokenisation, vectorisastions, RDBMA and GRAPH database|
| Web | [sportgramme-web](https://github.com/sportgramme/sportgramme-web) | The browser surface — public site, widgets, contributor tools |
| API | [sportgramme-api](https://github.com/sportgramme/sportgramme-api) | Internal API landscape and the syndication / distribution surface |
| On-Prem | [sportgramme-on-prem](https://github.com/sportgramme/sportgramme-on-prem) | Restricted back office — the access-control model and its operator console |

See
[ARCHITECTURE.md](https://github.com/sportgramme/sportgramme/blob/main/ARCHITECTURE.md)
for how the surfaces fit together and the consolidation rationale, and the shared
[glossary](https://github.com/sportgramme/sportgramme/tree/main/Glossary).
