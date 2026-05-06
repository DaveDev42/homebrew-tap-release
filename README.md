# homebrew-tap-release

GitHub composite action that bumps a Homebrew formula in a tap repository
when a new release is published.

Designed for the case where one user/org maintains both the source repos
and the tap repo, and wants a fine-grained PAT scoped to the tap repo only
(not the classic `repo`-wide PAT that
[mislav/bump-homebrew-formula-action][mislav] requires).

The action only ever talks to the tap repo. Source-repo metadata reads are
unnecessary because the caller passes the `download-url` directly.

## Usage

```yaml
jobs:
  bump-tap:
    needs: release
    runs-on: ubuntu-latest
    steps:
      - uses: DaveDev42/homebrew-tap-release@v1
        with:
          formula-name: macstate
          version: ${{ github.ref_name }}
          download-url: >-
            https://github.com/DaveDev42/macstate/releases/download/${{ github.ref_name }}/macstate-${{ github.ref_name }}-aarch64-apple-darwin.tar.gz
        env:
          TAP_TOKEN: ${{ secrets.HOMEBREW_TAP_TOKEN }}
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `formula-name` | yes | — | Formula name, e.g. `macstate`. |
| `version` | yes | — | New version. Accepts `0.1.2` or `v0.1.2`. |
| `download-url` | yes | — | Tarball URL; sha256 is computed by downloading it. |
| `formula-path` | no | `Formula/{formula-name}.rb` | Path inside the tap repo. |
| `tap-repo` | no | `DaveDev42/homebrew-tap` | `owner/name` of the tap. |
| `tap-branch` | no | `main` | Branch to commit to. |
| `commit-message` | no | `{name} {version}` | Supports `{name}` and `{version}` placeholders. |

## Required env

- `TAP_TOKEN` — fine-grained PAT with `Contents: Write` on the tap repo.

## Formula contract

The action edits the first line that matches each of:

- `^\s*version\s+"…"`
- `^\s*url\s+"…"`
- `^\s*sha256\s+"…"`

So formulas must be flat (no `if Hardware::CPU.arm? … else …` branches with
multiple `url`/`sha256` lines). Single-target macOS arm64 formulas only.

## License

BSD-3-Clause.

[mislav]: https://github.com/mislav/bump-homebrew-formula-action
