---
name: file-access-vuln
description: >-
  Entry P1 category router for file access and upload workflows. Use when
  testing download endpoints, file paths, local file inclusion, upload flows,
  preview pipelines, archive extraction, or storage and sharing boundaries.
---

# File Access Router

This is the routing entry point for filesystem paths, download endpoints, upload pipelines, and file preview handling.

## When to Use

- Parameters, filenames, download endpoints, or import flows influence file paths
- The target supports upload, preview, transcoding, extraction, sharing, download, or proxied file access
- You need to decide whether this is path traversal/LFI or an upload-validation/processing-chain issue

## Skill Map

- [Path Traversal LFI](../path-traversal-lfi/SKILL.md): path traversal, file read, wrapper abuse, include chains

## Upload Workflow Focus

For upload-centric cases, split the workflow into four stages before choosing a deeper skill:

1. Accept: extension, MIME, magic bytes, rename, and filename normalization
2. Store: final path, tenant isolation, overwrite behavior, and direct object URLs
3. Process: image/document/archive/XML converters, preview jobs, and extraction steps
4. Serve: inline render, forced download, sharing, and cache/CDN exposure

Then pivot as needed:

- [xss-cross-site-scripting](../xss-cross-site-scripting/SKILL.md): inline render, filename reflection, SVG/HTML execution
- [xxe-xml-external-entity](../xxe-xml-external-entity/SKILL.md): SVG, OOXML, XML import, or preview parsing
- [cmdi-command-injection](../cmdi-command-injection/SKILL.md): converters, media pipelines, or helper binaries shell out
- [business-logic-vulnerabilities](../business-logic-vulnerabilities/SKILL.md): quota, approval, replace, and cross-tenant storage rules

## Recommended Flow

1. First identify whether the entry point is a path parameter, download endpoint, or upload workflow
2. Then locate whether the issue appears in accept, store, process, or serve stages
3. For upload workflows, use the stage map above and pivot into the specific exploit category that matches the observed behavior
4. Small path-chain and upload-bypass samples are merged into the main topic skills; no separate payload entry is needed

## Related Categories

- [injection-checking](../injection-checking/SKILL.md)
- [business-logic-vuln](../business-logic-vuln/SKILL.md)
