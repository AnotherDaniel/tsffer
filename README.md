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

This action uploads a file to a GitHub release and generates a manifest.

### Inputs
- `github_token` (required): GitHub token for authentication.
- `file` (required): Path to the file to upload.
- `tag` (required): Release tag to associate with the file.
- `asset_name` (optional): Name of the asset.

### Outputs
- `release_asset_url`: URL of the uploaded release asset.
- `release_asset_manifest_url`: URL of the uploaded manifest file.

### Example Usage
```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: your-org/tsffer@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          file: ./example.txt
          tag: v1.0.0