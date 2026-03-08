<h1 align="center">MyBuddy-cfg</h1>

<p align="center">
  <strong>Remote model catalog definitions for MyBuddy LLM and STT downloads.</strong>
</p>

<p align="center">
  <a href="#overview">Overview</a> &bull;
  <a href="#related-repositories">Related Repositories</a> &bull;
  <a href="#repository-contents">Repository Contents</a> &bull;
  <a href="#how-the-app-uses-this-repository">How the App Uses This Repository</a> &bull;
  <a href="#schema-reference">Schema Reference</a> &bull;
  <a href="#editing-the-catalogs">Editing the Catalogs</a> &bull;
  <a href="#validation-and-release-checklist">Validation and Release Checklist</a> &bull;
  <a href="#breaking-change-policy">Breaking Change Policy</a>
</p>

---

## Overview

MyBuddy-cfg is the configuration repository for model discovery and download metadata used by the MyBuddy application.

This repository does not contain runtime application code. It contains machine-readable JSON catalogs that the Flutter app fetches remotely in order to:

- display available LLM and Whisper models
- download model files from remote sources
- verify downloaded files with minimum byte thresholds
- apply runtime loading configuration for supported model families

Because the MyBuddy app consumes these files directly from GitHub raw URLs, changes in this repository are effectively API changes.

---

## Related Repositories

This config repository is one part of the full MyBuddy product surface:

- **Flutter app**: https://github.com/newnonsick/MyBuddy
- **Unity avatar runtime**: https://github.com/newnonsick/MyBuddy-Unity
- **Model catalogs**: https://github.com/newnonsick/MyBuddy-cfg

---

## Repository Contents

```text
llm_models.json   Catalog of downloadable LLM models
stt_models.json   Catalog of downloadable Whisper STT models
```

---

## How the App Uses This Repository

The MyBuddy app fetches these files at runtime:

- `llm_models.json`
- `stt_models.json`

Current fetch locations used by the app:

- `https://raw.githubusercontent.com/newnonsick/MyBuddy-cfg/refs/heads/main/llm_models.json`
- `https://raw.githubusercontent.com/newnonsick/MyBuddy-cfg/main/stt_models.json`

The app then parses these files into typed descriptors, surfaces them in Settings, downloads the selected assets, and verifies basic integrity with `expectedMinBytes` before registering them as installed.

---

## Schema Reference

### LLM Catalog: `llm_models.json`

Each entry represents one downloadable LLM artifact.

Top-level fields:

- `id` - stable unique identifier used by the app
- `fileName` - filename stored locally after download
- `downloadUrl` - direct download URL
- `approximateSize` - human-readable size string for UI display
- `expectedMinBytes` - minimum accepted file size for validation
- `config` - runtime model configuration object

`config` fields:

- `type` - model family, such as `qwen`, `deepseek`, `gemmait`, `llama`, `hammer`, `functiongemma`, or `general`
- `maxTokens`
- `tokenBuffer`
- `randomSeed`
- `temperature`
- `topK`
- `topP`
- `isThinking`
- `supportsFunctionCalls`
- `fileType` - currently interpreted as `task` or `binary`

Example:

```json
{
  "id": "Qwen3-0.6B",
  "fileName": "Qwen3-0.6B.litertlm",
  "downloadUrl": "https://example.invalid/model",
  "approximateSize": "0.6 GB",
  "expectedMinBytes": 500000000,
  "config": {
    "type": "qwen",
    "maxTokens": 1280,
    "tokenBuffer": 1280,
    "randomSeed": 1,
    "temperature": 0.8,
    "topK": 1,
    "topP": null,
    "isThinking": false,
    "supportsFunctionCalls": true,
    "fileType": "task"
  }
}
```

### STT Catalog: `stt_models.json`

Each entry represents one Whisper model and optional CoreML encoder metadata.

Top-level fields:

- `id`
- `fileName`
- `downloadUrl`
- `approximateSize`
- `expectedMinBytes`
- `modelType`
- `config`
- `display`

`config` fields:

- `variant`
- `quantization`
- `coreML` - optional nested object for iOS and macOS accelerator assets

`coreML` fields:

- `downloadUrl`
- `archiveFileName`
- `extractedFolderName`
- `approximateSize`
- `expectedMinBytes`

`display` fields:

- `name`
- `description`

Example:

```json
{
  "id": "whisper-tiny",
  "fileName": "ggml-tiny.bin",
  "downloadUrl": "https://example.invalid/whisper",
  "approximateSize": "0.08 GB",
  "expectedMinBytes": 60000000,
  "modelType": "whisper",
  "config": {
    "variant": "tiny",
    "quantization": null,
    "coreML": {
      "downloadUrl": "https://example.invalid/coreml.zip",
      "archiveFileName": "ggml-tiny-encoder.mlmodelc.zip",
      "extractedFolderName": "ggml-tiny-encoder.mlmodelc",
      "approximateSize": "0.02 GB",
      "expectedMinBytes": 10000000
    }
  },
  "display": {
    "name": "Whisper Tiny",
    "description": "A balance between speed and accuracy, ideal for general use."
  }
}
```

---

## Editing the Catalogs

### When Adding a New LLM Model

1. Choose a stable `id` that will not need to be renamed later.
2. Use a direct, durable `downloadUrl`.
3. Set a realistic `expectedMinBytes` threshold.
4. Match `config.type` to a model family supported by the app.
5. Verify `fileType` matches how the app runtime loads the artifact.

### When Adding a New STT Model

1. Keep `modelType` consistent with app expectations.
2. Provide a clear `display.name` and `display.description`.
3. Set `expectedMinBytes` for the main model file.
4. If a CoreML encoder exists, provide all nested `coreML` fields.
5. Keep the non-CoreML flow valid for Android and other non-Apple platforms.

### Editing Rules

- prefer additive changes over renames or removals
- do not silently repurpose an existing `id`
- keep field names stable unless the app has been updated first
- treat download host changes as production-impacting changes

---

## Validation and Release Checklist

Before merging catalog changes:

1. Validate JSON syntax.
2. Confirm download URLs are reachable.
3. Confirm `expectedMinBytes` is realistic for the hosted file.
4. Confirm the app can parse the changed entries.
5. Confirm the new model family or field is supported by the Flutter app.

### Simple JSON Validation

Using Python:

```bash
python -m json.tool llm_models.json
python -m json.tool stt_models.json
```

### Recommended Manual Validation

- test new URLs in a browser or with `curl`
- check file size on the hosting side
- run the MyBuddy app and refresh the Settings model list
- download at least one changed entry end to end before release

---

## Release Process

This repository has no binary build step. Release is the act of publishing validated JSON to the branch consumed by the app.

Recommended process:

1. make catalog edits in a branch
2. validate syntax and download metadata
3. verify the Flutter app can consume the new entries
4. merge to the branch referenced by the app's raw GitHub URLs

Because the app reads these files remotely, broken JSON or incorrect URLs can break model management without requiring a new mobile app release.

---

## Breaking Change Policy

Avoid breaking changes whenever possible.

Breaking examples:

- renaming existing IDs already stored by users
- removing fields the app currently reads
- changing field meaning without updating the app
- replacing a model URL with a different artifact format under the same ID

Preferred approach:

- add new fields instead of renaming old ones
- add new IDs rather than mutating established ones
- coordinate app updates first when schema evolution is necessary

---

## Troubleshooting

### The app no longer shows the catalog

- check for invalid JSON
- verify GitHub raw URLs are still correct
- verify CORS, host reachability, and file availability

### Downloads fail after a catalog change

- verify the download URL points directly to the intended binary
- confirm `expectedMinBytes` is not larger than the actual file
- confirm the artifact format still matches the runtime loader

### Apple CoreML assets do not install

- verify the `coreML` object is complete
- confirm the archive expands to the expected `extractedFolderName`
- ensure the CoreML asset remains optional for non-Apple platforms

---

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
