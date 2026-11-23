# Dependency-Track MCP Server

A small Model Context Protocol (MCP) server for [OWASP Dependency-Track](https://dependencytrack.org/). It exposes a practical subset of the official Dependency-Track REST API over stdio so tools like Codex can query projects, fetch findings, trigger analysis, upload CycloneDX BOMs, and check async token status.

The implementation is based on the official Dependency-Track API surface:

- `GET /api/v1/project`
- `GET /api/v1/project/{uuid}`
- `GET /api/v1/project/lookup`
- `GET /api/v1/project/latest/{name}`
- `GET /api/v1/finding/project/{uuid}`
- `POST /api/v1/finding/project/{uuid}/analyze`
- `PUT /api/v1/bom`
- `GET /api/v1/event/token/{uuid}`
- `GET /api/v1/component/identity`

Official references:

- [Dependency-Track REST API docs](https://docs.dependencytrack.org/integrations/rest-api/)
- [ProjectResource.java](https://github.com/DependencyTrack/dependency-track/blob/master/src/main/java/org/dependencytrack/resources/v1/ProjectResource.java)
- [FindingResource.java](https://github.com/DependencyTrack/dependency-track/blob/master/src/main/java/org/dependencytrack/resources/v1/FindingResource.java)
- [BomResource.java](https://github.com/DependencyTrack/dependency-track/blob/master/src/main/java/org/dependencytrack/resources/v1/BomResource.java)
- [EventResource.java](https://github.com/DependencyTrack/dependency-track/blob/master/src/main/java/org/dependencytrack/resources/v1/EventResource.java)

## Features

- `list_projects`
- `search_projects_by_name`
- `get_project`
- `lookup_project`
- `get_latest_project`
- `get_project_findings`
- `trigger_project_analysis`
- `upload_bom`
- `get_event_token_status`
- `lookup_component`

## Requirements

- Node.js 18+ (tested with Node 25)
- A reachable Dependency-Track instance
- Either an API key or bearer token with the necessary Dependency-Track permissions

## Configuration

Set these environment variables before starting the server:

```powershell
$env:DEPENDENCY_TRACK_BASE_URL="https://dependency-track.example.com"
$env:DEPENDENCY_TRACK_API_KEY="your-api-key"
```

Or use a bearer token instead:

```powershell
$env:DEPENDENCY_TRACK_BASE_URL="https://dependency-track.example.com"
$env:DEPENDENCY_TRACK_BEARER_TOKEN="your-bearer-token"
```

## Run

```powershell
node src/index.js
```

Inside a docker container : 
```powershell
docker build -t dependency-track-mcp-server:0.2.0 .
```

## Codex MCP configuration

Example stdio entry:

```json
{
  "mcpServers": {
    "dependency-track": {
      "command": "node",
      "args": [
        "C:/absolute/path/to/dependency-track-mcp-server/src/index.js"
      ],
      "env": {
        "DEPENDENCY_TRACK_BASE_URL": "https://dependency-track.example.com",
        "DEPENDENCY_TRACK_API_KEY": "your-api-key"
      }
    }
  }
}
```

## OpenCode configuration

opencode.jsonc :

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "dependency-track": {
      "type": "local",
      "command": ["docker", "run", "-i", "--rm", "--read-only", "--cap-drop=ALL",
                  "--security-opt=no-new-privileges",
                  "-e", "DEPENDENCY_TRACK_BASE_URL", "-e", "DEPENDENCY_TRACK_API_KEY",
                  "dependency-track-mcp-server:0.2.0"],
      "enabled": true
    }
  }
}
```

> The variables here are passed to the docker so they should be exported in the opencode shell.

## Notes on permissions

The server only wraps official Dependency-Track endpoints. Actual access still depends on the permissions of the API key or bearer token:

- project listing and lookup: `VIEW_PORTFOLIO`
- component lookup: `VIEW_PORTFOLIO`
- findings and analysis: `VIEW_VULNERABILITY`
- BOM upload: `BOM_UPLOAD`
- auto-create during BOM upload: `PORTFOLIO_MANAGEMENT` or `PROJECT_CREATION_UPLOAD`

> Security Notice : 
>
> * VIEW_PORTFOLIO and VIEW_VULNERABILITY are enough for read-only use.
> * upload_bom and trigger_project_analysis change data on the server. An agent following injected instructions could call them, so leave them out of the key's permissions unless you need them.

## Query behavior

- `list_projects` now supports optional client-side `offset` and `limit` parameters.
- `search_projects_by_name` is intended for normal interactive use and defaults to returning up to 25 matches.
- Both tools still use the official `GET /api/v1/project` endpoint underneath.

## License

MIT
