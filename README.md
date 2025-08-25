# ChainSafe Infrastructure Docker Images

This repository contains Dockerfiles for ChainSafe Infrastructure services.

## Available Images

### [`fil-snapshots-archive`](./images/fil-snapshots-archive)

Docker image for storing FIL snapshots.

**Usage:**

```bash
docker pull ghcr.io/chainsafe/fil-snapshots-archive:latest
docker run -d \
  --name fil-snapshots-archive \
  -v /path/to/snapshots:/snapshots \
  ghcr.io/chainsafe/fil-snapshots-archive:latest
```

## Development

This repository uses:

- [pre-commit](https://pre-commit.com/) hooks for commit quality and linting.
- [release-please](https://github.com/googleapis/release-please) for automated releases.
- GitHub Actions workflows to build and push Docker images to GitHub Container Registry (GHCR).

### Building Locally

```bash
docker build -t fil-snapshots-archive ./images/fil-snapshots-archive
```

### Running Locally

```bash
docker run -it --rm fil-snapshots-archive
```

## Contributing

Contributions are welcome! Please ensure:

1. Code is formatted and linted using pre-commit hooks.
2. PRs follow the [repository workflow](.github/workflows).

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.
