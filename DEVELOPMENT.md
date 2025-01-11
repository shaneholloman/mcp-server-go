# Development Guide for prime and mcp-server-go

## Repository Relationship

- `mcp-server-go`: Core MCP server implementation
- `prime`: CLI tool that depends on `mcp-server-go`

These apps are tightly coupled - `prime` uses `mcp-server-go` as a dependency. They maintain version parity for clarity (when mcp-server-go is v0.1.0, prime should also be v0.1.0).

## Local Development Setup

For developing both apps simultaneously, use Go workspaces:

```bash
# In your go-apps directory containing both repositories
go work init
go work use ./prime
go work use ./mcp-server-go
```

This lets you modify both codebases without needing to push new versions.

## Versioning and Tagging

When ready to release a new version:

1. First tag mcp-server-go:

    ```bash
    cd mcp-server-go
    git tag v0.x.x  # e.g., v0.1.0
    git push origin v0.x.x
    ```

2. Then tag prime with the same version:

    ```bash
    cd ../prime
    git tag v0.x.x  # Same version as mcp-server-go
    git push origin v0.x.x
    ```

## Troubleshooting

If you encounter dependency or version issues:

1. Clear Go's module cache:

    ```bash
    go clean -modcache
    ```

2. Rebuild module dependencies:

    ```bash
    # In prime directory
    go mod tidy
    ```

3. If you see "unknown revision" or "invalid version" errors:

    - Check if both repositories have matching version tags
    - Verify tags are pushed to GitHub
    - Clear module cache and retry

4. To remove local installations:

    ```bash
    # Remove prime binary
    go clean -i github.com/shaneholloman/prime
    ```

5. To check current tags:

    ```bash
    # In each repository
    git tag -l
    ```

6. To delete a tag if needed:

    ```bash
    # Locally
    git tag -d v0.x.x
    # Remote
    git push --delete origin v0.x.x
    ```

## Common Issues

1. **Pseudo-versions appearing** (e.g., v0.0.0-20250111103953-9892921d5e0f):

    - This means Go can't find a proper version tag
    - Solution: Ensure tags are properly set and pushed

2. **Cache inconsistencies**:

    ```bash
    # Full cleanup
    go clean -modcache
    go clean -cache
    go mod tidy
    ```

3. **Wrong version being used**:

    ```bash
    # Check what version is actually being used
    go list -m github.com/shaneholloman/mcp-server-go
    go list -m github.com/shaneholloman/prime
    ```

## Development Workflow

1. Make changes in either/both repositories
2. Test changes locally using Go workspace
3. When ready to release:
   - Tag mcp-server-go first
   - Tag prime with same version
   - Push both tags
   - Run cleanup commands if needed
   - Verify installation: `go install github.com/shaneholloman/prime@latest`

## Git Commands Reference

```bash
# View all tags
git tag -l

# Create new tag
git tag v0.x.x

# Push new tag
git push origin v0.x.x

# Delete tag locally
git tag -d v0.x.x

# Delete tag from remote
git push --delete origin v0.x.x

# View tag details
git show v0.x.x
```

## Go Commands Reference

```bash
# Clear module cache
go clean -modcache

# Update dependencies
go mod tidy

# Check current module versions
go list -m all

# Install specific version
go install github.com/shaneholloman/prime@v0.x.x

# Setup workspace
go work init
go work use ./prime ./mcp-server-go
```

## Making Changes

When making changes to either repository, follow these steps:

1. **Changes to mcp-server-go**:
   - If you modify any interfaces in `mcp/types.go`
   - If you change any client methods in `client/*.go`
   - If you update server handling in `server/*.go`
   Then you MUST:
   1. Update prime's code to match the changes
   2. Test prime thoroughly with the changes
   3. Update both repositories' versions together

2. **Changes to prime**:
   - If you add new MCP client usage
   - If you modify how prime interacts with mcp-server-go
   Then you MUST:
   1. Verify compatibility with mcp-server-go
   2. Test with the current mcp-server-go version
   3. Consider if mcp-server-go needs updates

3. **Critical Files to Watch**:
   - In mcp-server-go:
     - `mcp/types.go`: Core types and interfaces
     - `client/*.go`: Client implementation
     - `server/*.go`: Server implementation
   - In prime:
     - `cmd/mcp.go`: MCP client usage
     - Any files that import mcp-server-go packages

4. **Testing Changes**:

   ```bash
   # In mcp-server-go directory
   go test ./...

   # In prime directory
   go test ./...

   # Test prime with local mcp-server-go changes
   go work use ./prime ./mcp-server-go
   cd prime
   go run . # Test your changes
   ```

## Important Notes

1. **Version Parity**:
   - Always keep `prime` and `mcp-server-go` versions in sync
   - This makes it clear which versions are compatible
   - Helps with debugging and support

2. **Local Development**:
   - Use Go workspaces for development
   - Only create tags when ready to release
   - Test thoroughly before pushing tags

3. **Cache Issues**:
   - Go's module cache can get out of sync
   - When in doubt, run `go clean -modcache`
   - Always run `go mod tidy` after cache cleanup

4. **Version Checking**:
   - Regularly verify you're using the intended versions
   - Check both repositories have matching tags
   - Use `go list -m all | grep shaneholloman` to see all your module versions

## Verifying Installation

After releasing new versions, verify the installation:

1. **Clean Install Test**:

    ```bash
    # Remove existing installation
    go clean -i github.com/shaneholloman/prime

    # Clear module cache
    go clean -modcache

    # Install latest version
    go install github.com/shaneholloman/prime@latest

    # Verify version
    prime version
    ```

2. **Specific Version Test**:

    ```bash
    # Install specific version
    go install github.com/shaneholloman/prime@v0.1.0

    # Verify correct version is installed
    prime version
    ```

If you see any issues during verification, refer to the Troubleshooting section above.
