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
The tsffer action has three operation modes:

- mode `file`: upload a file (release asset) to a GitHub release, generates a tsffer manifest that contains some metadata about the asset, and optionally upload manifest to release file set.
- mode `reference`: create tsffer evidence manifest file based on an URL reference, and optionally upload to release file set.
- mode `package`: typically run at the end of a release workflow, collect and package all generated tsffer manifest files into a single archive file, and optionally upload to release file set.

The generated asset manifest file will have the same name as the release asset, with an added '.tsffer' extension. For URL reference manifest files, the name is provided via tsffer action configuration. The asset manifest is using json syntax, and contains some metadata pertaining the the originating git repository and release run, as well as some user-provided input like asset name, description, asset type, and a list of TSF IDs that the asset pertains to.

## Inputs

- `github_token` (required): GitHub token for authentication.
- `mode` (required): Operation mode: "file" (default), "reference" or "package".
- `file` (required if mode is "file"): Path to the file to upload.
- `file_glob`: If set to true, the file argument can be a glob pattern (Default: false).
- `release_upload`: Boolean (true/false) switch determining whether generated tsffer file/archive should be uploaded to release file set (default: false).
- `reference_urls` (required if mode is "reference"): URL(s) referencing evidence. Multiple URLs can be provided by using `|` as a separator.
- `asset_name` (required if mode is "reference"): Name of the asset. When not provided and mode is "file", will use the file name.
- `asset_description` (optional): More detailed description of the asset (Default: `""`).
- `asset_type` (optional): Type of the asset; free-text field that might be used to e.g. declare an asset to be of type DOCUMENTATION, SOURCE, etc (Default: `""`).
- `asset_tsf_ids` (optional): List of TSF identifiers that this asset pertains to; can be one or more identifiers separated by commas (Default: `""`).

## Outputs

- `tsffer_file`: Name of generated json file or tar archive containing tsffer metadata.

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
          mode: file
          file: README.md
          asset_description: "For tsffer testing purposes, we are providing our README"
          asset_name: "Project README"
          asset_tsf_ids: "TA-BEHAVIOURS"
          asset_type: "DOCUMENTATION"
      - name: Point to action in release workflow
        uses: anotherdaniel/tsffer
        id: tsffer_ReleaseCI
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: reference
          reference_urls: "https://github.com/AnotherDaniel/tsffer/blob/369c487287fb1e2fbe15d580501445c5d2b062ed/.github/workflows/release.yml#L11|https://www.example.org"
          asset_description: "Link to specific line in tsffer release automation"
          asset_name: "ReleaseCI"
          asset_tsf_ids: "TA-RELEASES,TA-ITERATIONS"
          asset_type: "SOURCE"
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
    "description": "For tsffer testing purposes, we are providing our README",
    "download-url": "https://github.com/AnotherDaniel/tsffer/releases/download/v0.0.42/README.md",
    "name": "Project README",
    "tsf-ids": [
      "TA-BEHAVIOURS"
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

The second tsffer step will generate a `ReleaseCI.tsffer` reference manifest, with the following content:

```json
{
  "asset-info": {
    "checksum-sha256": "db930ec18bfb83cd6db0180faead58d645a9ca71cff7b02e78e1583e3c89c7ec",
    "description": "Link to specific line in tsffer release automation",
    "evidence_links": [
      "https://github.com/AnotherDaniel/tsffer/blob/369c487287fb1e2fbe15d580501445c5d2b062ed/.github/workflows/release.yml#L11",
      "https://www.example.org",
    ],
    "name": "ReleaseCI",
    "tsf-ids": [
      "TA-RELEASES",
      "TA-ITERATIONS"
    ],
    "type": "SOURCE"
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
