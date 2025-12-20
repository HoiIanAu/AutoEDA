# AutoEDA MCP Server Setup Guide

Quick setup for Claude Desktop, VS Code Copilot, and Cursor to use the AutoEDA MCP Server. Using the MCP server, you can:
- Run synthesis, placement, clock tree synthesis, and routing tasks directly from your editor
- Request the agent to display and analyze EDA results 

## Prerequisites

- Claude Desktop or VS Code with Copilot or Cursor
- AutoEDA Server running: 
    ```python
    # Open a terminal
    # Activate the virtual environment
    # Navigate to the project root
    # Then run:
    python src/run_server.py --server all
    ```

## Claude Desktop Configuration

To connect Claude Desktop with AutoEDA MCP server:

1. Open Claude Desktop and go to **Settings** → **Developer** → **Local MCP Servers** → **Edit Config**.
2. Add a new MCP server configuration with the following JSON (update the paths as needed):

    ```json
    {
        "mcpServers": {
            "AutoEDA": {
                "command": "\\abs\\path\\to\\venv\\bin\\python.exe",
                "args": ["\\abs\\path\\to\\src\\server\\mcp_server.py"],
                "cwd": "\\abs\\path\\to\\project\\root"
            }
        }
    }
    ```
    > **Note:** Use `/` for Linux/macOS paths and `\\` for Windows paths.
3. Save the configuration and restart Claude Desktop if required.
4. In the chat interface, select the **AutoEDA** MCP server from the available tools.


## VS Code Copilot Configuration

1. In the project root, create a folder called `.vscode`
2. Create a file called `mcp.json` with the below content:

    ```json
    {
        "servers": {
            "AutoEDA": {
                "type": "stdio",
                "command": "/abs/path/to/venv/bin/python3",
                "args": ["/abs/path/to/src/server/mcp_server.py"],
                "cwd": "/abs/path/to/project/root"
            }
        }
    }
    ```    
    > **Note:** Use `/` for Linux/macOS paths and `\\` for Windows paths.

3. Open the chat view `Ctrl+Alt+I` and select **Agent** mode from the drop-down menu.
4. Select the **Tools** button to see whether the custom MCP tools are in the list of available tools.

## Cursor Configuration

1. Go to **File** → **Preferences** → **Cursor Settings** → **MCP&Integrations** → **New MCP Server**
2. Add MCP server configuration:

    ```json
    {
        "mcpServers": {
            "AutoEDA": {
                "type": "stdio",
                "command": "/abs/path/to/venv/bin/python3",
                "args": ["/abs/path/to/src/server/mcp_server.py"],
                "cwd": "/abs/path/to/project/root"
            }
        }
    }
    ```
3. You should see the AutoEDA server in the MCP tools section in the Cursor Settings.

## Testing

In the chat input box enter:
- "What are all the available inputs for the synth server?"
- "What MCP tools are available?"
- "Run synthesis for design des with FreePDK45 using mcp tool"
- "Show me the power consumption report for this design"
- "Run the full flow for design des with FreePDK45 technology"

You should observe the MCP tools being executed in the chat interface.

## Troubleshooting

**Application not recognizing MCP:**
- Make sure AutoEDA servers are running
- Restart the application after updating the configuration (force close from Task Manager if needed)
- Verify that the JSON configuration is valid and free of syntax errors
- Use absolute paths for server configuration
