# Apache Unomi MCP Server

A Model Context Protocol server that provides seamless integration between Claude Desktop and Apache Unomi, enabling profile management and context-aware interactions.

## Overview

This MCP server enables Claude to maintain context about users through Apache Unomi's profile management system. Here's what you can achieve with it:

### Key Capabilities
1. **User Recognition**:
   - Identify users across conversations using email or profile ID
   - Maintain consistent user context between sessions
   - Automatically create and manage user profiles

2. **Context Management**:
   - Store and retrieve user preferences
   - Track user segments and scores
   - Maintain conversation history and user properties

3. **Integration Features**:
   - Seamless Claude Desktop integration
   - Protected event handling
   - Automatic session management
   - Scope-based context isolation

### What You Can Do
- Have Claude remember user preferences across conversations
- Store and retrieve user-specific information
- Track user interactions and behavior
- Maintain consistent user context
- Manage multiple users through email identification

### What You Cannot Do
- Create multiple sessions per day for the same profile
- Modify protected events without proper Unomi key
- Access profiles outside IP whitelist
- Create custom session IDs (they're date-based)
- Bypass Unomi's security mechanisms

### Prerequisites
- Running Apache Unomi server
- Claude Desktop installation
- Network access to Unomi server
- Proper security configuration
- Required environment variables

## Demo

Watch how the MCP server enables Claude to maintain context and manage user profiles:

[![Apache Unomi MCP Server Demo](https://img.youtube.com/vi/UTMbWqimk0M/0.jpg)](https://www.youtube.com/watch?v=UTMbWqimk0M)

This demo showcases:
- Profile management through Claude
- Email-based user recognition
- Context persistence across conversations
- Property updates and retrieval
- Seamless Claude Desktop integration

## Features

### Resources
- List and access profiles via `unomi://profiles/list` URI
- Each profile includes properties, segments, scores, and consents
- JSON format for structured data access

### Tools
- `create_scope` - Create a new Unomi scope
  - Takes scope identifier and optional name/description
  - Required for event tracking and profile updates
  - Example:
    ```json
    {
      "scope": "my-app",
      "name": "My Application",
      "description": "Scope for my application events"
    }
    ```
- `get_my_profile` - Get your profile using environment variables
  - Uses UNOMI_PROFILE_ID from environment
  - Automatically generates a session ID based on the current date
  - Optional parameters:
    - requireSegments: Include segment information
    - requireScores: Include scoring information
- `update_my_profile` - Update properties of your profile
  - Uses UNOMI_PROFILE_ID from environment
  - Takes a properties object with key-value pairs to update
  - Supports string, number, boolean, and null values
  - Example:
    ```json
    {
      "properties": {
        "firstName": "John",
        "age": 30,
        "isSubscribed": true,
        "oldProperty": null
      }
    }
    ```
- `get_profile` - Retrieve a specific profile by ID
  - Takes profileId as required parameter
  - Returns full profile data from Unomi
- `search_profiles` - Search for profiles
  - Takes query string and optional limit/offset parameters
  - Searches across firstName, lastName, and email fields

### Scope Management
The server automatically manages scopes for you:

1. Default Scope:
   - A default scope `claude-desktop` is used for all operations
   - Created automatically when needed
   - Used for profile updates and event tracking

2. Custom Scopes:
   - Can be created using the `create_scope` tool
   - Useful for separating different applications or contexts
   - Must exist before using in profile operations

3. Automatic Scope Creation:
   - The server checks if required scopes exist
   - Creates them automatically if missing
   - Uses meaningful defaults for scope metadata

> **Note**: While scopes are created automatically when needed, you can still create them manually with custom names and descriptions using the `create_scope` tool.

## Configuration

### Environment Variables

The server requires the following environment variables:

```bash
UNOMI_BASE_URL=http://your-unomi-server:8181
UNOMI_USERNAME=your-username
UNOMI_PASSWORD=your-password
UNOMI_PROFILE_ID=your-profile-id
UNOMI_SOURCE_ID=your-source-id
UNOMI_KEY=your-unomi-key
UNOMI_EMAIL=your-email
```

### Profile Resolution

The server uses a two-step process to resolve the profile ID:

1. Email Lookup (if `UNOMI_EMAIL` is set):
   - Searches for a profile with matching email
   - If found, uses that profile's ID
   - Useful for maintaining consistent profile across sessions

2. Fallback Profile ID:
   - If email lookup fails or `UNOMI_EMAIL` is not set
   - Uses the `UNOMI_PROFILE_ID` from environment
   - Ensures a profile is always available

The response will indicate which method was used via the `source` field:
- `"email_lookup"`: Profile found via email
- `"environment"`: Using fallback profile ID

### Unomi Server Configuration

The `update_my_profile` tool uses protected events that require additional Unomi server configuration:

1. Set up the Unomi key in your `etc/org.apache.unomi.cluster.cfg` file:
   ```properties
   org.apache.unomi.cluster.authorization.key=your-unomi-key
   ```

2. Configure allowed IP addresses in `etc/org.apache.unomi.cluster.cfg`:
   ```properties
   org.apache.unomi.ip.ranges=127.0.0.1,::1,your-claude-desktop-ip
   ```
   Replace `your-claude-desktop-ip` with your local machine's IP address where Claude Desktop is running.

3. Restart Unomi server to apply changes

> **Note**: The `update_my_profile` tool will not work unless both the Unomi key and IP address are properly configured.

## Development

Install dependencies:
```bash
npm install
```

Build the server:
```bash
npm run build
```

For development with auto-rebuild:
```bash
npm run watch
```

## Installation

To use with Claude Desktop, add the server config and environment variables:

On MacOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
On Windows: `%APPDATA%/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "unomi-server": {
      "command": "/path/to/unomi-server/build/index.js",
      "env": {
        "UNOMI_BASE_URL": "http://your-unomi-server:8181",
        "UNOMI_USERNAME": "your-username",
        "UNOMI_PASSWORD": "your-password",
        "UNOMI_PROFILE_ID": "your-profile-id",
        "UNOMI_SOURCE_ID": "claude-desktop",
        "UNOMI_KEY": "your-unomi-key",
        "UNOMI_EMAIL": "your-email@example.com"
      }
    }
  }
}
```

The `env` section in the configuration allows you to set the required environment variables for the server. Replace the values with your actual Unomi server details.

### Debugging

Since MCP servers communicate over stdio, debugging can be challenging. We recommend using the [MCP Inspector](https://github.com/modelcontextprotocol/inspector), which is available as a package script:

```bash
npm run inspector
```

The Inspector will provide a URL to access debugging tools in your browser.

You can also tail the Claude Desktop logs to see MCP requests and responses:

```bash
# Follow logs in real-time
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
```

### Session ID Format
When using `get_my_profile`, the session ID is automatically generated using the format:
```
[profileId]-YYYYMMDD
```
For example, if your profile ID is "user123" and today is March 15, 2024, the session ID would be:
```
user123-20240315
```
