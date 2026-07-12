# Azure DevOps Integration MCP Extension

Sync user stories and tasks from Spec Kit to Azure DevOps work items using OAuth authentication (no PAT tokens required).

## Features

- Azure DevOps MCP Integration
- User Story Sync: Create Azure DevOps User Story work items from spec.md
- Task Sync: Create Azure DevOps Task work items from tasks.md, automatically linked to parent User Stories
- Interactive Selection: Choose which user stories or tasks to sync
- Configuration Persistence: Saves organization, project, and area path for reuse
- Work Item Mapping: Tracks synced items to prevent duplicates
- Priority Mapping: Automatically maps spec-kit priorities (P1-P4) to Azure DevOps priorities
- Auto-Hook: Optional automatic sync after task generation

## Installation

```bash
# Install Directly by URL
specify extension add azure-devops --from https://github.com/rmanju1981/spec-kit-azure-devops-mcp/archive/refs/tags/v1.0.0.zip
```

## Prerequisites

- **Azure DevOps MCP Server**: Required for Azure DevOps API access
  - Link: https://github.com/microsoft/azure-devops-mcp
  
- **Spec Kit**: Version 0.1.0 or higher

## Configuration

### Option 1: Interactive Configuration (Recommended)

The extension will prompt you for configuration the first time you use it:

1. **Organization**: Your Azure DevOps organization name (e.g., "MSFTDEVICES" from `https://dev.azure.com/MSFTDEVICES`)
2. **Project**: Your Azure DevOps project name (e.g., "Devices")
3. **Area Path**: Work item area path (e.g., "Devices\\SW\\ASPX\\CE\\Portals and Services")

Configuration is saved to `~/.speckit/ado-config.json` and reused for future syncs.

### Option 2: Manual Configuration

Create `~/.speckit/ado-config.json`:

```json
{
  "Organization": "your-org-name",
  "Project": "your-project-name",
  "AreaPath": "your-area-path",
  "LastUpdated": "2026-03-03 10:30:00"
}
```

## Usage

### Sync User Stories

```bash
# In your AI agent (Claude, Copilot, etc.)
> /speckit.azure-devops-mcp.sync
```

The command will:

1. Ask for Azure DevOps configuration (if not already saved)
2. Parse the feature folder present in the root folder and ask user to select the feature
3. Ask the user either to select Story or Tasks to sync
4a. If user selects story to sync, then it parse user stories from `spec.md`
   a. Display found stories and ask which ones to sync
   b. Create Azure DevOps User Story work items
   c. Display results with work item IDs and URLs
4b. If user selects tasks to sync, then it parse the tasks from `tasks.md`
   a. Display found tasks and ask which ones to sync
   b. Create Azure DevOps tasks work items and links it to user stories
   c. Display results with work item IDs and URLs

5. Display overall results

### Automatic Hook After Task Generation

After running `/speckit.tasks`, you'll be prompted:

```text
## Extension Hooks

**Optional Hook**: azure-devops
Command: `/speckit.azure-devops-mcp.sync`
Description: Automatically create Azure DevOps work items after task generation

Sync tasks to Azure DevOps? (yes/no)
```

## Configuration Reference

### Saved Configuration (`~/.speckit/ado-config.json`)

| Setting       | Type   | Required | Description                                               |
| ------------- | ------ | -------- | --------------------------------------------------------- |
| Organization  | string | Yes      | Azure DevOps organization name                            |
| Project       | string | Yes      | Azure DevOps project name                                 |
| AreaPath      | string | Yes      | Work item area path (e.g., "Project\\Team\\Component")      |
| LastUpdated   | string | No       | Timestamp of last configuration update (auto-maintained)  |

## Examples

### Example 1: First-Time Setup and Sync

```bash
# Step 1: Create specification
> /speckit.spec Create photo album management feature

# Step 2: Generate tasks
> /speckit.tasks

# Step 3: Sync to Azure DevOps (will prompt for configuration)
> /speckit.azure-devops-mcp.sync
```

## Troubleshooting

### Issue: Command not available in AI agent

**Solutions**:

1. Check extension is installed: `specify extension list`
2. Restart your AI agent
3. Reinstall extension: `specify extension remove azure-devops-mcp && specify extension add azure-devops-mcp`

## License

MIT License - see [LICENSE](LICENSE) file

## Support

- **Issues**: https://github.com/github/spec-kit-azure-devops-mcp/issues
- **Spec Kit Docs**: https://github.com/github/spec-kit

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

*Extension Version: 1.0.0*  
*Spec Kit: >=0.1.0*  

