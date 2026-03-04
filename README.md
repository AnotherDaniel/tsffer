<!--
 * Copyright (C) 2025 Eclipse Foundation and others. 
 * 
 * This program and the accompanying materials are made available under the
 * terms of the Eclipse Public License v. 2.0 which is available at
 * http://www.eclipse.org/legal/epl-2.0.
 * 
 * SPDX-FileType: DOCUMENTATION
 * SPDX-FileCopyrightText: 2025 Eclipse Foundation
 * SPDX-License-Identifier: EPL-2.0
-->

# tsffer Action

The tsffer action collects metadata about evidence that can support statements about quality and process adherence of a project - automated via the project release workflow. It is designed to support adoption of the Trustable Software Framework [TSF](https://codethinklabs.gitlab.io/trustable/trustable/).
The tsffer action has two operation modes:

- mode `reference` (default): create tsffer evidence manifest file based on evidence reference properties as consumed by the TSF trudag tool, and optionally upload manifest to release file set.
- mode `package`: typically run at the end of a release workflow, collect and package all generated tsffer manifest files into a single archive file, and optionally upload to release file set.

The asset manifest is using json syntax, and contains some metadata pertaining the the originating git repository and release run, as well as user-provided input like asset name, type of TSF evidence that is being referenced, an optional description, and a list of TSF IDs that the asset pertains to.

## Inputs

- `github_token` (required): GitHub token for authentication.
- `mode`: Operation mode: "reference" (default) or "package".
- `release_upload`: Boolean (true/false) switch determining whether generated tsffer file/archive should be uploaded to release file set (default: false).
- `reference_properties` (required if mode is "reference"): json snippet describing the referenced evidence (refer to examples below).
- `asset_name` (required if mode is "reference"): Name of the asset.
- `asset_description` (optional): More detailed description of the asset (Default: `""`).
- `asset_tsf_ids` (required if mode is "reference"): list of TSF identifiers that the evidence pertains to; can be one or more identifiers separated by commas (Default: `""`).

## Outputs

- `tsffer_file`: Name of generated json file or tar archive containing tsffer metadata.

## Target release and other expectations

This action expects to run in the context of a release (tag-initiated) GitHub worflow, using an Ubuntu runner:

- `github.ref_name` and `github.ref` are set to valid/real values, as these are used to determine asset upload target
- Ubuntu runner comes with pre-installed gh and jq binaries (this is currently the case on GitHub)

## Example Usage

Using tsffer in your workflow looks like this:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Upload README to release
        uses: svenstaro/upload-release-action@v2
        id: upload_readme
        with:
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          file: README.md
          tag: ${{ github.ref }}

      - name: Collect README artifact
        uses: anotherdaniel/tsffer
        id: tsffer_README
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: reference
          reference_properties: |
            {
              "reference_type": "download_url",
              "url": "${{ steps.upload_readme.outputs.browser_download_url }}"
            }
          asset_description: "For illustration purposes, we are providing a link to our README in the release artifacts"
          asset_name: "Project README"
          asset_tsf_ids: "TA-BEHAVIOURS"

      - name: Point to action in release workflow
        uses: anotherdaniel/tsffer
        id: tsffer_ReleaseCI
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: reference
          reference_properties: |
            [
              {
                "reference_type": "github",
                "repository": "${{ github.repository }}",
                "path": ".github/workflows/release.yml#L11",
                "public": true,
                "ref": "${{ github.ref_name }}"
              },
              {
                "reference_type": "webpage",
                "url": "https://www.example.org",
              }
            ]
          asset_description: "Link to specific line in tsffer release automation"
          asset_name: "ReleaseCI"
          asset_tsf_ids: "TA-RELEASES,TA-ITERATIONS"

      - name: Package quality artifacts, but leave assets in place
        uses: anotherdaniel/tsffer
        id: tsffer_package
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: package
          release_upload: true
```

## Example manifest

The above workflow will generate a `README.md.tsffer` manifest file pipeline run artifact, with the following content:

```json
{
  "asset-info": {
    "checksum-sha256": "db930ec18bfb83cd6db0180faead58d645a9ca71cff7b02e78e1583e3c89c7ec",
    "description": "For illustration purposes, we are providing a link to our README in the release artifacts",
    "reference-properties": 
      {
        "reference_type": "download_url",
        "url": "https://github.com/AnotherDaniel/tsffer/releases/download/v0.0.42/README.md"
      }
    "name": "Project README",
    "tsf-ids": [
      "TA-BEHAVIOURS"
    ],
  },
  "context-info": {
    "by-workflow": ".github/workflows/release.yml",
    "commit-sha": "bc100425602aed8f55005f37e689b8603be94637",
    "ref": "refs/tags/v0.0.42",
    "release-url": "https://github.com/AnotherDaniel/tsffer/releases/tag/v0.0.42",
    "repository": "AnotherDaniel/tsffer"
  }
}
````

The second tsffer step will generate a `ReleaseCI.tsffer` reference manifest, with the following content:

```json
{
  "asset-info": {
    "checksum-sha256": "db930ec18bfb83cd6db0180faead58d645a9ca71cff7b02e78e1583e3c89c7ec",
    "description": "Link to specific line in tsffer release automation",
    "reference-properties": 
    [
      {
        "reference_type": "github",
        "repository": "AnotherDaniel/tsffer",
        "path": ".github/workflows/release.yml#L11",
        "public": true,
        "ref": "v0.0.42"
      },
      {
        "reference_type": "webpage",
        "url": "https://www.example.org",
      }
    ],
    "name": "ReleaseCI",
    "tsf-ids": [
      "TA-RELEASES",
      "TA-ITERATIONS"
    ],
  },
  "context-info": {
    "by-workflow": ".github/workflows/release.yml",
    "commit-sha": "bc100425602aed8f55005f37e689b8603be94637",
    "ref": "refs/tags/v0.0.42",
    "release-url": "https://github.com/AnotherDaniel/tsffer/releases/tag/v0.0.42",
    "repository": "AnotherDaniel/tsffer"
  }
}
````

The third tsffer step will package up the two previous files into an archive (`tsffer_assets.tar.bz2`), and upload this archive to the release file set (`release_upload` parameter set to `true`).
