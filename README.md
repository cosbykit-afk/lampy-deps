# lampy-deps

Build dependencies for the [Lampy](https://github.com/cosbykit-afk/lampy-single)
forum stack: an offline Python wheelhouse and a vendored Ollama distribution.
These are the two large build inputs the `lampy-single` Dockerfile expects in
its build context (as `wheelhouse/` and `ollama-donor/`). They live here
instead of in that repo because GitHub caps release files at 2 GB each, so
both archives are split into parts.

## Release: `build-deps-v1`

All files are on the
[build-deps-v1 release](https://github.com/cosbykit-afk/lampy-deps/releases/tag/build-deps-v1)
(7 assets, ~8.4 GB total).

| Archive | Parts | Size | MD5 |
|---|---|---|---|
| `wheelhouse.tar` | `.part00`–`.part02` | 3,356,774,400 bytes (~3.4 GB) | `a5244e6d6861c4d1f59aae6d4ec449c6` |
| `ollama-donor.tar` | `.part00`–`.part03` | 5,049,344,000 bytes (~5.0 GB) | `6838234504bcef0c1a734cf5f8fccb4d` |

Contents:

- **wheelhouse.tar** — 209 offline Python wheels (torch, pgai, flask,
  gunicorn, psycopg, etc.) used by the Dockerfile to install packages
  without network access.
- **ollama-donor.tar** — `/usr/bin/ollama`, `/usr/lib/ollama/`, and
  `/usr/share/ollama/` extracted from the pinned
  `ollama/ollama:latest@sha256:da6e0dc5651df159e45686fd663c4dbe1624a52c44d7280eeac1551d8f865532`
  image, so the build doesn't re-download Ollama.

## Assembling the pieces

Download **all** parts of an archive, concatenate them **in order**, then
verify the MD5 before extracting:

```bash
# Wheelhouse (3 parts)
cat wheelhouse.tar.part00 wheelhouse.tar.part01 wheelhouse.tar.part02 > wheelhouse.tar
md5sum wheelhouse.tar   # expect a5244e6d6861c4d1f59aae6d4ec449c6
tar -xf wheelhouse.tar  # produces wheelhouse/

# Ollama donor (4 parts)
cat ollama-donor.tar.part00 ollama-donor.tar.part01 \
    ollama-donor.tar.part02 ollama-donor.tar.part03 > ollama-donor.tar
md5sum ollama-donor.tar   # expect 6838234504bcef0c1a734cf5f8fccb4d
tar -xf ollama-donor.tar  # produces ollama-donor/
```

On Windows (PowerShell), replace `cat` with:

```powershell
Get-Content wheelhouse.tar.part* -Raw -AsByteStream |
    Set-Content wheelhouse.tar -AsByteStream
```

Place the extracted `wheelhouse/` and `ollama-donor/` directories next to
the `lampy-single` Dockerfile, then build:

```bash
docker build -t lampy-single .
```

## Checksums

| Item | Algorithm | Digest |
|---|---|---|
| `wheelhouse.tar` (reassembled) | MD5 | `a5244e6d6861c4d1f59aae6d4ec449c6` |
| `ollama-donor.tar` (reassembled) | MD5 | `6838234504bcef0c1a734cf5f8fccb4d` |
| Ollama executable (inside donor) | SHA-256 | `ca9f4d3b7538196fab8bad3842bff3aa1352a80bfaaf95391ef7dda28880928b` |

## Build status (2026-09-28)

- **Release `build-deps-v1`: COMPLETE** — all 7 parts uploaded and
  checksums verified against the published digests above.
- The `lampy-single` Docker image (`kitcosby/lampy-single:windows-1.0.0`)
  was built from exactly these dependencies.

## Known issues

- **No per-part checksums** — only the reassembled archives have published
  digests. If a download is corrupt you'll find out at the `md5sum` step;
  re-download the parts and try again.
- **An empty `build-deps-v1` release also exists on
  [lampy-single](https://github.com/cosbykit-afk/lampy-single/releases/tag/build-deps-v1)**
  — that's a leftover, ignore it; the real files are here.

## License

Public domain ([The Unlicense](https://unlicense.org)).
