# Install ArtifactBridge in Cline

ArtifactBridge is a remote MCP server. Nothing runs on the local computer, and
the server needs no API key. The person signs in with browser OAuth.

- Server URL: `https://app.artifactbridge.com/mcp`
- Transport: Streamable HTTP
- Authentication: OAuth 2.1 with PKCE and dynamic client registration
- Requirement: an ArtifactBridge account (sign up at
  <https://www.artifactbridge.com>)

## Steps

1. Add this entry to `cline_mcp_settings.json` under `mcpServers`. Do not add
   an `Authorization` header or a token.

   ```json
   {
     "mcpServers": {
       "artifactbridge": {
         "type": "streamableHttp",
         "url": "https://app.artifactbridge.com/mcp",
         "disabled": false
       }
     }
   }
   ```

2. Save the file. Cline connects to the server, and the server asks for
   authorization.
3. Tell the person to sign in in the browser and choose a workspace. If the
   browser does not open, tell the person to open the MCP Servers view in
   Cline and start the authentication for `artifactbridge` there.
4. Check the connection: call `artifactbridge_get_workspace_info`. A
   successful reply names the workspace. An empty document list is a correct
   result for a new workspace.

## Rules for the agent

- Agents read, comment, and propose changes. People approve changes.
- Do not change external sources (Google Docs, Notion) directly.
- Do not ask the person to paste a credential into the chat.

Full guide: <https://www.artifactbridge.com/docs/connect>
