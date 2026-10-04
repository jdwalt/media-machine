# Ingestion workflow

Owned optical media enters ARM for identification, MakeMKV extraction, Quick Sync transcoding, completed-library placement and ejection. Raw and transcode paths stay on the scratch disk; completed media enters the primary storage pool through `/srv/media`. Both physical optical drives have been detected and concurrent jobs were demonstrated in project records.

The collected settings select OMDb metadata, a 60-second manual identification window, duplicate-disc processing, MakeMKV `mkv` mode and HandBrake `qsv_h264`. DVD and Blu-ray presets are recorded in `config/arm/arm.yaml.example`; output uses MKV with English subtitle selection. The current ARM UI requires login.

The Rev2 patch preserves CRC-derived identification in the label-search stage and blocks generic disc labels from metadata retries. Identification remains a workflow that includes checking titles and accepted placement. The patch, Dockerfile and upstream attribution live in `image/arm-rev2/` and `third-party/`.

Portable-drive, photo/document and permitted online imports are manual workflows described in the records. Copy into a staging area, compare bytes, inspect representative media streams, establish the destination identity, resolve filename collisions, place into the correct library, and verify playback or opening. Keep original source data until its copy and backup are verified. Collection-specific rename scripts and private file inventories are excluded from this repository.

This project is intended to preserve and enjoy media obtained with permission, rather than encourage unauthorized acquisition. Public-domain works, openly licensed media and creator-authorized downloads offer useful starting points. [Project Gutenberg](https://www.gutenberg.org/) provides ebooks with [permission and licensing guidance](https://www.gutenberg.org/policy/permission); copyright status can differ outside the United States. [Internet Archive](https://archive.org/) offers many downloadable collections, but its [rights guidance](https://archivesupport.zendesk.com/hc/en-us/articles/360014759692-Rights) explains that availability alone does not establish permission. Check the rights for each item and use purchased media only where personal copying is permitted under the applicable law and terms.

Mini-media manual transfers are separate from optical ingestion and are currently on hold. No automated synchronization is active, and Jellyfin metadata and users remain separate.
