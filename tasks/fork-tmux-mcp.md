# Task: Fork and Enhance tmux MCP

## Objective
Fork the tmux MCP server to add a comprehensive status resource that provides a complete view of all sessions, windows, and panes.

## Motivation
Currently, discovering the tmux state requires multiple API calls. A single resource that shows the full hierarchy would be invaluable for AI agents.

## Proposed Resource: `tmux://status`

Should return a JSON structure like:
```json
{
  "sessions": [
    {
      "id": "$0",
      "name": "main",
      "attached": true,
      "windows": [
        {
          "id": "@1",
          "name": "cora",
          "active": true,
          "panes": [
            {
              "id": "%1",
              "title": "editor",
              "active": false,
              "current_command": "nvim",
              "content_preview": "first 5 lines..."
            },
            {
              "id": "%2",
              "title": "shell",
              "active": true,
              "current_command": "bash",
              "content_preview": "$ |"
            }
          ]
        }
      ]
    }
  ]
}
```

## Additional Enhancements

1. **Add tmux control tools:**
   - `split-pane` - Create splits without manual shortcuts
   - `new-window` - Create windows programmatically
   - `resize-pane` - Adjust pane sizes
   - `kill-pane` - Close panes safely

2. **Safety features:**
   - Warn when executing in editor panes
   - Provide pane type detection
   - Add confirmation for destructive operations

3. **Better documentation:**
   - Include limitations in tool descriptions
   - Add examples for common workflows
   - Provide AI-specific guidance

## Steps

1. Fork https://github.com/zcaceres/tmux-mcp
2. Add the status resource
3. Add control command tools
4. Update documentation
5. Test with Claude Code
6. Submit PR or maintain fork

## Repository
- Original: https://github.com/zcaceres/tmux-mcp
- Fork: TBD