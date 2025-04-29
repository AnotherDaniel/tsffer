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

This action uploads a file (release asset) to a GitHub release, generates a Trustable Software Framework [TSF](https://codethinklabs.gitlab.io/trustable/trustable/) manifest that contains some metadata for the asset, and uploaeds the metadata alongside the original release asset.
The idea behind this action is to make it possible to create evidence links between TSF statements and corresponding release assets of a project - all automated via the project release workflow.
The generated asset manifest file will have the same name as the release asset, with an added '.tsffer' extension. The asset manifest is using json syntax, and contains some metadata pertaining the the originating git repository and release run, as well as some user-provided input like asset name, description, asset type, and a list of TSF IDs that the asset pertains to.

## Inputs

- `github_token` (required): GitHub token for authentication.
- `file` (required): Path to the file to upload.
- `tag` (required): Tag of the release to upload to.
- `asset_name` (optional): Name of the asset. When not provided will use the file name.
- `asset_description` (optional): More detailed description of the asset (Default: `""`).
- `asset_type` (optional): Type of the asset; free-text field that might be used to e.g. declare an asset to be of type DOCUMENTATION, SOURCE, etc (Default: `""`).
- `asset_tsf_ids` (optional): List of TSF identifiers that this asset pertains to; can be one or more identifiers separated by commas (Default: `""`).

## Outputs

- `release_asset_url`: Download url of the release asset.
- `release_asset_manifest_url`: Download url of the release asset manifest.

## Example Usage

Using tsffer in your workflow looks like this:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Collect README artifact
        uses: anotherdaniel/tsffer
        id: tsffer_README
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          file: README.md
          tag: ${{ github.ref }}
          asset_description: "For tsffer testing purposes, we are providing our README"
          asset_name: "Project README"
          asset_tsf_ids: "TT-TA_01,TT-TA_02"
          asset_type: "DOCUMENTATION"
```

## Example manifest

The above workflow will generate a `README.md.tsffer` manifest file alongside the original `README.md` in the list of release assets, with the following content:

```json
{
  "asset-info": {
    "checksum-sha256": "0b3f3339c15920d7dc6ccdbbc7577b0a33cf15cd412cd04effc16446b2a7f469",
    "description": "For tsffer testing purposes, we are providing our brilliant README",
    "download-url": "https://github.com/AnotherDaniel/tsffer/releases/download/v0.0.11/README.md",
    "name": "Project README",
    "tsf-ids": [
      "TT-TA_01",
      "TT-TA_02"
    ],
    "type": "DOCUMENTATION"
  },
  "context-info": {
    "by-workflow": ".github/workflows/release.yml",
    "commit-sha": "c6c4d32f109884c9ae1b56d23b0b8a2eeb247743",
    "ref": "refs/tags/v0.0.11",
    "repository": "AnotherDaniel/tsffer"
  }
}
````
