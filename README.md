# Build, Inspect, and Run an MCP Server

A short hands-on tutorial: use Claude Code to **create** a small
[Model Context Protocol](https://modelcontextprotocol.io) (MCP) server,
**inspect** it to confirm it works, then **run** it so Claude can use it.

The server, `ned`, exposes one tool: `resolve_object(name)`. It looks up an
extragalactic object (e.g. "M87") in [NED](https://ned.ipac.caltech.edu) and
returns its J2000 coordinates, object type, and redshift.

```
  Claude  ──MCP (stdio)──▶  server.py  ──HTTPS──▶  NED ObjectLookup
          ◀── result ─────            ◀── JSON ──
```

## What's here

| Path                 | Purpose                                                  |
| -------------------- | -------------------------------------------------------- |
| `prompt.txt`         | The prompt you give Claude Code to build the server       |
| `workspace/`         | Empty folder: build your server here                      |
| `solution/`          | A finished, working server, if you get stuck               |
| `PRESENTER_NOTES.md` | Run-of-show for presenting this as a live demo            |

## Prerequisites

- [Claude Code](https://docs.claude.com/en/docs/claude-code)
- [uv](https://docs.astral.sh/uv/) (Python package and project manager)
- [Node.js](https://nodejs.org) (only needed for the MCP Inspector web UI)
- The `mcp-server-dev` plugin. Install it inside Claude Code:

  ```
  /plugin install mcp-server-dev@claude-plugins-official
  ```

## Step 1: Create the server

```bash
cd workspace
claude
```

Paste the contents of [`prompt.txt`](prompt.txt) into Claude Code. Claude
uses the `mcp-server-dev` plugin to set up a uv project and write `server.py`.

Read the generated `server.py` before you go on. Things to notice:

- `FastMCP("ned")` creates the server.
- `@mcp.tool` turns a normal Python function into an MCP tool.
- The **docstring and type hints** are what Claude sees. They tell the model
  what the tool does and when to use it.

## Step 2: Inspect the server

Test the server before you connect it to Claude. All commands run from
`workspace/`.

**From the terminal:**

```bash
uv run fastmcp list server.py                              # what tools exist?
uv run fastmcp call server.py resolve_object name=M87      # call one
```

**In the browser (MCP Inspector):**

```bash
uv run fastmcp dev inspector server.py
```

In the Inspector, click **Connect**, open the **Tools** tab, click
**List Tools**, choose `resolve_object`, enter `M87`, and click **Run Tool**.
You should see something like:

```json
{
  "name": "Messier 087",
  "ra_deg": 187.7059308,
  "dec_deg": 12.39112326,
  "object_type": "G",
  "redshift": 0.004282962982
}
```

## Step 3: Run it with Claude

Register the server with Claude Code (run this from `workspace/`):

```bash
claude mcp add ned -- uv run --directory "$(pwd)" server.py
claude mcp list        # ned should show as connected
```

Start a **new** Claude Code session in the same folder, then ask:

> What are the coordinates of M87?

Claude should call `resolve_object` and answer with the NED result. Type
`/mcp` inside Claude Code to see the server and its tools.

## Try next

- Ask about a star, such as "Where is Vega?". Does Claude still use the tool?
  How does the docstring affect that choice?
- Add a second tool, such as one that searches for objects near a position.
- Connect the same server to Claude Desktop or another MCP client.

## Start over

```bash
claude mcp remove ned -s local        # run from workspace/
cd workspace && rm -rf ./* .venv
```

**Stuck?** Copy the finished version with `cp solution/* workspace/`, then
continue from Step 2.
