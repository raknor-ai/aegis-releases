# AEGIS Platform Binaries

Pre-built binaries for the [AEGIS](https://aegis.raknor.ai) security scanner.

## Quick Start

Download the binary for your platform from [Releases](https://github.com/raknor-ai/aegis-releases/releases/latest), extract, and scan:

```bash
# macOS Apple Silicon
tar xzf aegis-cli-darwin-arm64.tar.gz
chmod +x aegis
./aegis scan ./your-project

# Linux x64
tar xzf aegis-cli-linux-x64.tar.gz
chmod +x aegis
./aegis scan ./your-project
```

## Available Platforms

| Platform | Asset |
|---|---|
| macOS Apple Silicon | `aegis-cli-darwin-arm64.tar.gz` |
| macOS Intel | `aegis-cli-darwin-x64.tar.gz` |
| Linux x64 | `aegis-cli-linux-x64.tar.gz` |
| Linux ARM64 | `aegis-cli-linux-arm64.tar.gz` |
| Windows x64 | `aegis-cli-win32-x64.zip` |

## CI/CD

```yaml
# GitHub Actions
- name: Download AEGIS
  run: |
    curl -sL https://github.com/raknor-ai/aegis-releases/releases/latest/download/aegis-cli-linux-x64.tar.gz | tar xz
    chmod +x aegis
- name: AEGIS Security Scan
  run: ./aegis scan . --sarif --fail-on critical
```

## Verify Downloads

Each release includes a `SHA256SUMS` file:

```bash
sha256sum -c SHA256SUMS
```

## License Activation

```bash
# Activate (writes to ~/.raknor/license.json)
aegis activate --key AEGIS-...

# Or environment variable
export AEGIS_LICENSE_KEY="AEGIS-..."
```

Without a key, AEGIS runs in **Community mode**: full Rust engine, 50-finding cap, SARIF + HTML + JSON output, compliance readiness preview.

## Documentation

See the [User Guide](USER-GUIDE.md) for the complete flag reference, tier comparison, and feature documentation.

---
*Pareidolia LLC (d/b/a Raknor AI / Equilateral AI)*
