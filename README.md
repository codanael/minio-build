# MinIO Release Automation

This repository automatically builds MinIO binaries whenever a new version is released by the official [MinIO project](https://github.com/minio/minio).

## Features

- **Automated Release Monitoring**: Checks daily for new MinIO releases
- **Multi-Platform Builds**: Builds binaries for:
  - Linux (AMD64, ARM64)
  - macOS (AMD64, ARM64)
  - Windows (AMD64)
- **Docker Images**: Publishes multi-arch Docker images to GitHub Container Registry
- **Checksums**: Generates SHA256 checksums for all binaries
- **GitHub Releases**: Automatically creates releases with binaries

## How It Works

The GitHub Actions workflow runs daily and:

1. Checks for new MinIO releases via GitHub API
2. Verifies if the version has already been built
3. Clones the MinIO source code at the specific release tag
4. Builds binaries for multiple platforms using MinIO's official build process
5. Generates checksums for verification
6. Builds and publishes multi-arch Docker images to GitHub Container Registry
7. Creates a GitHub release with all binaries and Docker image links

## Setup

### Required Permissions

The workflow requires the following GitHub Actions permissions:

- `contents: write` - To create releases
- `packages: write` - To push Docker images to GitHub Container Registry

These are configured in the workflow file and are automatically available via `GITHUB_TOKEN`.

### GitHub Container Registry Setup

Docker images are automatically published to GitHub Container Registry (GHCR). No additional secrets or configuration is required.

By default, GHCR packages are private. To make your MinIO images publicly accessible:

1. Go to your repository's main page
2. Click on "Packages" in the right sidebar (after the first build)
3. Click on the `minio-build` package
4. Click "Package settings"
5. Scroll down to "Danger Zone"
6. Click "Change visibility" and select "Public"

## Usage

### Automatic Builds

The workflow runs automatically every day at 00:00 UTC. No manual intervention is required.

### Manual Builds

You can also trigger builds manually:

1. Go to Actions → MinIO Release Build
2. Click "Run workflow"
3. Optionally specify a MinIO version (e.g., `RELEASE.2024-10-29T16-01-48Z`)
4. Click "Run workflow"

### Downloading Binaries

Binaries are available in the [Releases](../../releases) section:

```bash
# Example: Download Linux AMD64 binary
wget https://github.com/YOUR_USERNAME/YOUR_REPO/releases/download/RELEASE.2024-10-29T16-01-48Z/minio-linux-amd64.tar.gz

# Verify checksum
wget https://github.com/YOUR_USERNAME/YOUR_REPO/releases/download/RELEASE.2024-10-29T16-01-48Z/minio-linux-amd64.tar.gz.sha256
sha256sum -c minio-linux-amd64.tar.gz.sha256

# Extract and run
tar xzf minio-linux-amd64.tar.gz
chmod +x minio-linux-amd64
./minio-linux-amd64 server /data
```

### Using Docker Images

Docker images are published to GitHub Container Registry:

```bash
# Pull latest image
docker pull ghcr.io/YOUR_USERNAME/YOUR_REPO:latest

# Or specific version
docker pull ghcr.io/YOUR_USERNAME/YOUR_REPO:RELEASE.2024-10-29T16-01-48Z

# Run MinIO
docker run -p 9000:9000 -p 9001:9001 \
  -v /data:/data \
  ghcr.io/YOUR_USERNAME/YOUR_REPO:latest \
  server /data --console-address ":9001"
```

**Note**: Replace `YOUR_USERNAME/YOUR_REPO` with your actual GitHub repository path (e.g., `octocat/minio-build`).

If the package is private, you'll need to authenticate first:

```bash
# Login to GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u YOUR_USERNAME --password-stdin

# Then pull the image
docker pull ghcr.io/YOUR_USERNAME/YOUR_REPO:latest
```

## Build Process

The builds follow MinIO's official build process:

- **Go Version**: 1.24 or later
- **Build Command**: `CGO_ENABLED=0 go build -tags kqueue -trimpath`
- **Build Flags**: Generated using MinIO's `buildscripts/gen-ldflags.go`
- **Static Binaries**: All binaries are statically compiled (CGO disabled)

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── minio-release-build.yml    # Main workflow
└── README.md                           # This file
```

## Workflow Details

### Jobs

1. **check-release**: Determines the latest MinIO version and checks if it's already built
2. **build-binaries**: Builds binaries for all platforms in parallel
3. **build-docker**: Builds and publishes multi-arch Docker images
4. **create-release**: Creates a GitHub release with all binaries

### Matrix Strategy

The workflow uses a matrix strategy to build for multiple platforms simultaneously:

```yaml
matrix:
  include:
    - goos: linux, goarch: amd64
    - goos: linux, goarch: arm64
    - goos: darwin, goarch: amd64
    - goos: darwin, goarch: arm64
    - goos: windows, goarch: amd64
```

## Troubleshooting

### Workflow not running

- Ensure GitHub Actions is enabled in repository settings
- Check that the workflow file is in the correct location: `.github/workflows/`
- Verify that the default branch has the workflow file

### Build failures

- Check the Actions tab for detailed logs
- Verify Go version compatibility with MinIO
- Ensure MinIO's build requirements haven't changed

### Docker push failures

- Ensure the workflow has `packages: write` permission
- Check that `GITHUB_TOKEN` has the necessary permissions
- Verify that GitHub Container Registry is accessible
- Check the Actions logs for specific error messages

## License

This repository contains automation scripts only. The MinIO binaries are subject to MinIO's AGPLv3 license. See the [official MinIO repository](https://github.com/minio/minio) for details.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Disclaimer

This is an unofficial build repository. Production environments using these compiled binaries do so at their own risk. For official MinIO releases, visit [https://github.com/minio/minio/releases](https://github.com/minio/minio/releases).
