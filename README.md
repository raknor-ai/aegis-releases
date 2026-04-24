# AEGIS Platform Binaries

Pre-built binaries for the [AEGIS](https://aegis.raknor.ai) security scanner.

## Install via npm (recommended)

```bash
npx @raknor/aegis scan ./your-project
```

The npm package auto-downloads the correct binary for your platform on first run.

## Manual download

Download the tarball for your platform from [Releases](https://github.com/raknor-ai/aegis-releases/releases), extract, and add to PATH:

```bash
tar xzf aegis-cli-darwin-arm64.tar.gz
./aegis scan-local ./your-project --all
```

## Available platforms

| Platform | Asset |
|---|---|
| macOS Apple Silicon | `aegis-cli-darwin-arm64.tar.gz` |
| macOS Intel | `aegis-cli-darwin-x64.tar.gz` |
| Linux x64 | `aegis-cli-linux-x64.tar.gz` |
| Linux ARM64 | `aegis-cli-linux-arm64.tar.gz` |
| Windows x64 | `aegis-cli-win32-x64.tar.gz` |

Not all platforms are available in every release. Check the release assets.

## Documentation

See the [User Guide](USER-GUIDE.md) for the complete flag reference, tier comparison, and feature documentation.

## License

The AEGIS binary runs in Community mode (free, 50-finding cap) by default.
Set `AEGIS_PRODUCT_KEY` for Pro/Premium/Enterprise tiers.

See [aegis.raknor.ai](https://aegis.raknor.ai) for pricing.

---
*Pareidolia LLC (d/b/a Raknor AI)*
