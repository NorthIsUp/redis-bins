# redis-bins

Pre-compiled [Redis](https://github.com/redis/redis) binaries for Linux and macOS (x86_64 and arm64). OpenSSL is statically linked, so the binaries are self-contained — no Homebrew, no `libssl` version skew.

Releases track upstream `redis/redis` stable tags. A nightly cron checks for new releases and kicks off a build automatically.

## Install

### mise

Use [mise](https://mise.jdx.dev)'s [`ubi`](https://mise.jdx.dev/dev-tools/backends/ubi.html) backend. One entry per binary:

```sh
mise use -g "ubi:NorthIsUp/redis-bins[exe=redis-server]@8.6.2"
mise use -g "ubi:NorthIsUp/redis-bins[exe=redis-cli]@8.6.2"
```

Or in `.mise.toml`:

```toml
["ubi:NorthIsUp/redis-bins"]
version = "8.6.2"
exe = "redis-server"
```

Add additional entries for `redis-benchmark`, `redis-check-aof`, `redis-check-rdb` as needed. Platform/arch are auto-detected from the tarball asset names.

### curl

```sh
VERSION=8.6.2
OS=$(uname -s | tr '[:upper:]' '[:lower:]' | sed 's/darwin/macos/')
ARCH=$(uname -m)

curl -fsSL "https://github.com/NorthIsUp/redis-bins/releases/download/${VERSION}/redis-${VERSION}-${OS}-${ARCH}.tar.gz" \
  | tar xz
```

Binaries land in `redis-${VERSION}-${OS}-${ARCH}/bin/`.

## Platforms

| OS    | Arch   | Runtime requirement          |
|-------|--------|------------------------------|
| linux | x86_64 | glibc ≥ 2.35 (Ubuntu 22.04+) |
| linux | arm64  | glibc ≥ 2.35                 |
| macos | x86_64 | macOS 13+                    |
| macos | arm64  | macOS 14+                    |

TLS is included via OpenSSL 3.5 (statically linked).

## Verifying portability

Linux:

```sh
ldd redis-*/bin/redis-server   # should not list libssl or libcrypto
```

macOS:

```sh
otool -L redis-*/bin/redis-server   # should only list /usr/lib/* and /System/*
```

The build workflow runs these checks and fails if a non-system library is referenced.

## Building manually

```sh
gh workflow run build-release.yml --repo NorthIsUp/redis-bins --ref main -f redis_version=8.6.2
```

Workflow source: [`.github/workflows/build-release.yml`](.github/workflows/build-release.yml). Upstream release watcher: [`.github/workflows/nightly-check.yml`](.github/workflows/nightly-check.yml).
