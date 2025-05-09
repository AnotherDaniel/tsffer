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
- `file_glob`: If set to true, the file argument can be a glob pattern (Default: false).
- `asset_name` (optional): Name of the asset. When not provided will use the file name.
- `asset_description` (optional): More detailed description of the asset (Default: `""`).
- `asset_type` (optional): Type of the asset; free-text field that might be used to e.g. declare an asset to be of type DOCUMENTATION, SOURCE, etc (Default: `""`).
- `asset_tsf_ids` (optional): List of TSF identifiers that this asset pertains to; can be one or more identifiers separated by commas (Default: `""`).

## Outputs

- `release_url`: URL of the release.

## Target release and other expectations

This action expects to run in the context of a release (tag-initiated) GitHub worflow, using an Ubuntu runner:

- `github.ref_name` and `github.ref` are set to valid/real values, as these are used to determine asset upload target and download URLs
- Ubuntu runner comes with pre-installed gh and jq binaries (this is currently the case on GitHub)

## file_glob

If used with input parameter `file_glob` set to `true`, the `file` parameter will be interpreted as a glob pattern, and all matching files will be treated identically:

- a `.tsffer` manifest is generated for each file, with identical metadata each as provided by the workflow action
- the asset files and the corresponding `.tsffer` manifests are uploaded to the GitHub release

Please note that all uploaded files end up in a flat list of GitHub release assets - take care to not glob-upload multiple files with identical file names!

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
    "checksum-sha256": "db930ec18bfb83cd6db0180faead58d645a9ca71cff7b02e78e1583e3c89c7ec",
    "description": "For tsffer testing purposes, we are providing our README",
    "download-url": "https://github.com/AnotherDaniel/tsffer/releases/download/v0.0.42/README.md",
    "name": "Project README",
    "tsf-ids": [
      "TT-TA_01",
      "TT-TA_02"
    ],
    "type": "DOCUMENTATION"
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
