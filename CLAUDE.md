# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Terraform provider for Ansible Automation Platform (AAP) that uses HashiCorp's Terraform Plugin Framework (protocol version 6.0).

## Key Commands

### Build and Development
- `make build` - Compile the provider binary
- `make test` - Run unit tests
- `make testacc` - Run acceptance tests (requires AAP instance)
- `make lint` - Run golangci-lint static analysis
- `make gofmt` - Format code
- `make generatedocs` - Regenerate documentation from templates

### Testing a Single Test
```bash
# Unit test
go test -v -run TestInventoryResource_Create ./internal/provider/

# Acceptance test
TF_ACC=1 go test -v -run TestAccInventoryResource_basic ./internal/provider/
```

### Environment Setup for Acceptance Tests
```bash
export AAP_USERNAME=admin
export AAP_PASSWORD=password
export AAP_INSECURE_SKIP_VERIFY=true
export AAP_HOST=https://aap.example.com
export AAP_TEST_JOB_TEMPLATE_ID=1
export AAP_TEST_WORKFLOW_JOB_TEMPLATE_ID=2
export AAP_TEST_ORGANIZATION_ID=2
```

## Architecture

### Provider Structure
- **Entry point**: `main.go`
- **Provider implementation**: `internal/provider/provider.go`
- **HTTP client**: `internal/provider/client.go` (implements `ProviderHTTPClient` interface)
- **Base classes**: `base_datasource.go` provides reusable logic for data sources
- **Custom types**: `internal/provider/customtypes/` for JSON/YAML variable handling

### Key Patterns

1. **Resources** follow this structure:
   - Model struct (e.g., `InventoryResourceModel`) - maps Terraform schema to Go
   - Resource struct (e.g., `InventoryResource`) - implements CRUD operations
   - Must implement `resource.Resource` and `resource.ResourceWithConfigure`

2. **Data Sources** inherit from base classes:
   - `BaseDataSource` for simple data sources
   - `BaseDataSourceWithOrg` for organization-scoped data sources

3. **API Models** use inheritance:
   - `BaseDetailAPIModel` for common fields
   - `BaseDetailAPIModelWithOrg` adds organization support

### Adding New Resources

1. Create `internal/provider/<resource_name>_resource.go`
2. Define model and resource structs
3. Implement Create, Read, Update, Delete methods
4. Add to `Resources()` in `provider.go`
5. Create unit and acceptance tests
6. Add documentation template in `templates/resources/`
7. Add example in `examples/resources/`
8. Run `make generatedocs`

### Adding New Data Sources

1. Create `internal/provider/<name>_data_source.go`
2. Extend `BaseDataSource` or `BaseDataSourceWithOrg`
3. Define minimal schema and Read method
4. Add to `DataSources()` in `provider.go`
5. Add tests, documentation, and examples

## Testing Approach

- **Unit tests**: Mock HTTP client using `ProviderHTTPClient` interface
- **Acceptance tests**: Require real AAP instance with proper environment variables
- **Fixtures**: Use `fixtures.go` for test data
- Always run `make lint` before committing

## Release Process

1. Run `make generatedocs` to format examples
2. Run `antsibull-changelog release --version <version>`
3. Commit and push changes
4. Tag with `v<version>` format (e.g., `v1.2.3`)
5. GitHub Actions handles registry publication