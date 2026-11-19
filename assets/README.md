# Assets

`cover-light.png` and `cover-dark.png` are the README cover (2560x800, shown through a
`<picture>` so GitHub's dark mode gets the dark one). They share the usewrit.app tape look
with the covers of every other public usewrit repository: the writ wordmark and red bar,
mono annotations, one red headline line. `banner.svg` is the previous banner, kept only
because older release pages still point at it.

## Product screenshots — not yet captured

`README.md` deliberately ships **no product screenshots** yet. Screenshots taken from a
desktop build with no engine running show empty states and error toasts, which is worse
than none.

Capture these from a real desktop build with a running engine and some real data, save
them here under `media/`, and add them to the README:

| File | Shot |
|---|---|
| `media/record-steps.gif` | the recorder: a live browser with each action landing as an editable step |
| `media/workflows.png` | the workflows list with several real workflows |
| `media/call-this.png` | the "Call this workflow" panel showing the REST / OpenAI / MCP surfaces |
| `media/monitors.png` | a monitor with change history |
| `media/connect-mcp.png` | Settings → MCP & APIs with a client installed |

Keep them under 2 MB each — `scripts/publish-desktop.sh` fails the publish above that,
because oversized blobs in git history are permanent.
