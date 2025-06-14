# JobSpy MCP Server - Windows Setup Guide

A complete step-by-step guide to set up the JobSpy MCP Server on Windows for use with Claude Desktop.

## Prerequisites

Before starting, install the following software:

1. **Node.js** (v16 or higher)
   - Download from: https://nodejs.org/
   - Choose the LTS version
   - Install with default settings

2. **Docker Desktop for Windows**
   - Download from: https://www.docker.com/products/docker-desktop/
   - Install and start Docker Desktop
   - Ensure it's running (Docker icon in system tray)

3. **Git** (for cloning the repository)
   - Download from: https://git-scm.com/downloads
   - Install with default settings

4. **Claude Desktop**
   - Download from: https://claude.ai/download
   - Install and set up your account

## Step 1: Clone and Navigate to Repository

Open PowerShell as Administrator and run:

```powershell
# Navigate to your desired directory (e.g., your user folder)
cd C:\Users\YourUsername

# Clone the repository (replace with actual repository URL)
git clone https://github.com/yourusername/jobspy-mcp-server.git

# Navigate to the project directory
cd jobspy-mcp-server
```

## Step 2: Build the JobSpy Docker Image

The Node.js server executes job searches via a Docker container, so we need to build the image first:

```powershell
# Navigate to the jobspy directory
cd jobspy

# Build the Docker image with the name 'jobspy'
docker build -t jobspy .

# Verify the image was created
docker images jobspy

# Test the image works
docker run jobspy --help
```

You should see the help output showing all available job search parameters.

## Step 3: Install Node.js Dependencies

```powershell
# Navigate back to the main project directory
cd ..

# Install Node.js dependencies
npm install
```

Note: You may see some warnings about deprecated packages - these are normal and don't affect functionality.

## Step 4: Fix Windows Compatibility

The original code uses `sudo docker run` which doesn't work on Windows. We need to modify the source code:

**Edit the file:** `src/tools/search-jobs.js`

**Find this line (around line 158):**
```javascript
const cmd = `sudo docker run jobspy ${args.join(' ')}`;
```

**Replace it with:**
```javascript
const cmd = `docker run jobspy ${args.join(' ')}`;
```

**Save the file.**

## Step 5: Test the MCP Server

Before configuring Claude Desktop, test that the server works:

```powershell
# Set environment variable and start the server
$env:ENABLE_SSE=0; node src/index.js
```

You should see output like:
```
info: Starting JobSpy MCP server...
info: Stdio transport connected
info: Server successfully connected with transports: stdio
```

Press `Ctrl+C` to stop the server.

## Step 6: Configure Claude Desktop

1. **Create/Edit the Claude Desktop config file:**

   The config file is located at: `%APPDATA%\Claude\claude_desktop_config.json`

   Open it with:
   ```powershell
   notepad "$env:APPDATA\Claude\claude_desktop_config.json"
   ```

2. **Add the following configuration** (replace entire file content):

   ```json
   {
     "mcpServers": {
       "jobspy": {
         "command": "node",
         "args": ["C:\\Users\\YourUsername\\jobspy-mcp-server\\src\\index.js"],
         "env": {
           "ENABLE_SSE": "0"
         }
       }
     }
   }
   ```

   **Important:** 
   - Replace `YourUsername` with your actual Windows username
   - Use double backslashes (`\\`) in the Windows path
   - Ensure the path points to your exact installation location

3. **Save and close** the file.

## Step 7: Restart Claude Desktop

1. **Close Claude Desktop** completely if it's running
2. **Restart Claude Desktop** - it will automatically load your MCP server

## Step 8: Test the Integration

Once Claude Desktop is restarted, test the integration by asking Claude:

- *"Can you search for software engineer jobs in Seattle?"*
- *"Find remote Python developer positions posted in the last 24 hours"*
- *"Search for data scientist jobs on LinkedIn and Indeed in San Francisco"*

## Supported Job Sites

The server can search across these job platforms:
- Indeed
- LinkedIn
- Glassdoor
- ZipRecruiter  
- Google Jobs
- Bayt
- Naukri

## Available Search Parameters

When asking Claude to search for jobs, you can specify:

- **Job titles/keywords:** "software engineer", "data scientist", etc.
- **Location:** "San Francisco, CA", "New York", "remote"
- **Job sites:** "LinkedIn and Indeed", "all sites"  
- **Time filters:** "last 24 hours", "past week"
- **Job types:** "full-time", "part-time", "contract", "internship"
- **Results count:** "find 50 jobs", "get 10 results"

## Troubleshooting

### JSON Parsing Errors (FIXED)
**Problem**: Errors like "Unexpected non-whitespace character after JSON" in log files.
**Solution**: The updated version has reduced logging output in stdio mode to prevent interference with JSON communication.

### Prompts Errors (FIXED)  
**Problem**: "field.isOptional is not a function" errors.
**Solution**: Prompts have been temporarily disabled to ensure core job search functionality works properly.

### Server Won't Start
- Ensure Docker Desktop is running
- Check that Node.js is properly installed: `node --version`
- Verify the path in Claude config file is correct

### Claude Desktop Not Connecting
- Check the config file syntax (valid JSON)
- Ensure the file path uses double backslashes
- Restart Claude Desktop after config changes
- Check Claude Desktop logs for error messages

### Docker Issues
- Ensure Docker Desktop is running
- Verify the jobspy image exists: `docker images jobspy`
- Test Docker works: `docker run hello-world`

### Job Search Errors
- Some job sites may have rate limiting
- Try reducing the number of results requested
- Check your internet connection
- Some sites may block automated requests temporarily

### Log File Analysis
If you see a `mcp-server-jobspy.log` file, look for these indicators:
- ✅ `"Server successfully connected"` - Connection is working
- ✅ `"tools":[{"name":"search_jobs"` - Tool is properly registered
- ✅ Tool responses without JSON errors - Everything working properly
- ❌ Repeated JSON parsing errors - Fixed in updated version
- ❌ `field.isOptional` errors - Fixed by disabling prompts

## Optional Configuration

You can set additional environment variables before starting Claude Desktop:

```powershell
# Set these in your PowerShell profile or before starting Claude
$env:JOBSPY_PORT="9423"
$env:LOG_LEVEL="info"
```

## File Structure

After setup, your directory should look like:

```
jobspy-mcp-server/
├── src/
│   ├── index.js          # Main MCP server
│   ├── tools/
│   │   └── search-jobs.js # Modified for Windows
│   ├── schemas/
│   └── prompts/
├── jobspy/
│   ├── main.py           # Python job search script
│   ├── Dockerfile        # For building jobspy image
│   └── requirements.txt
├── tests/
├── package.json
└── README.md
```

## Security Notes

- The server runs Docker containers to perform job searches
- No personal data is stored permanently
- Job search results are returned directly to Claude
- The server only runs when Claude Desktop is active

## Success Indicators

You'll know everything is working when:

1. ✅ Docker image `jobspy` exists and shows help when tested
2. ✅ Node.js server starts with "Stdio transport connected" message
3. ✅ Claude Desktop starts without MCP-related errors
4. ✅ Claude can successfully respond to job search requests
5. ✅ Job search results are returned in a structured format

## Support

If you encounter issues:

1. Check all prerequisites are installed and running
2. Verify file paths in the Claude config
3. Test each component individually
4. Check Windows firewall/antivirus isn't blocking Docker or Node.js
5. Ensure you have sufficient permissions to run Docker

---

*This guide was created for Windows 10/11 with PowerShell. Adjust paths and commands as needed for your specific setup.* 