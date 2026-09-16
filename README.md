# mcp-forge — MCP Server

Generate production-ready MCP servers from any source: OpenAPI/Swagger specs, GraphQL APIs, codebases (23 languages incl. COBOL & Fortran), CLI tools and websites.

**SSE endpoint**: `https://mcp.mcp-forge.vulcai.io/sse`  
**Auth**: `Authorization: Bearer <YOUR_LICENSE_KEY>`  
**Get a license**: https://mcp-forge.vulcai.io/register

## Tools

### generate_from_openapi

Generate a complete MCP server from an OpenAPI/Swagger spec URL. Pass the URL of the spec file (e.g. `/openapi.json`, `/swagger.json`) — not the API base URL. Server-side parsing, no pre-processing required.

### generate_from_graphql

Generate a complete MCP server from a GraphQL endpoint. Uses server-side introspection. Works with endpoints that have introspection enabled (dev/staging environments).

### generate_from_website

Generate an MCP server by analyzing a public website's structure and patterns. Works best with server-rendered HTML pages.

### generate_async

Submit a long-running generation job and get a `job_id` immediately (< 100 ms). Use `get_job_status` to poll for completion. Recommended for large specs or slow endpoints.

### get_job_status

Poll the status of an async generation job. Returns `pending`, `running`, `done`, or `failed`. When `done`, includes the full generation result with `bundle_url`, `eval_score`, and `preview_tools`.

### get_health

Check the health status of the mcp-forge API. Returns `status`, `version`, `license_mode`, `region`, and `degraded_features`.

### validate_license

Validate a license key and retrieve plan details: `valid`, `plan`, `tenant_id`, `valid_until`, `license_mode`. Always returns HTTP 200 (never 401).

### get_quota

Retrieve quota usage for the current license: `sources_used`, `sources_limit` (or `"unlimited"`), `is_unlimited`, `valid_until`.

## Configuration

```json
{
  "mcpServers": {
    "mcp-forge": {
      "url": "https://mcp.mcp-forge.vulcai.io/sse",
      "headers": {
        "Authorization": "Bearer <YOUR_LICENSE_KEY>"
      }
    }
  }
}
```

## Links

- Homepage: https://mcp-forge.vulcai.io
- CLI on PyPI: https://pypi.org/project/vulcai-mcp-forge-cli/
- Trust & Security: https://mcp-forge.vulcai.io/trust
- Troubleshooting: https://mcp-forge.vulcai.io/docs/troubleshooting/no-tools
