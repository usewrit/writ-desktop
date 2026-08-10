# Assets

`banner.svg` is the README banner. It shares its lockup and palette with the self-hosted
repo's banner (`selfhost/assets/banner.svg`) — same tile, glyph, Signal bar, and embedded
wordmark — and differs only in the tagline, the capability chips, and the footer line.
Keep the two in sync when the brand changes; they are the two public front doors.

The wordmark is embedded as a raster data URI rather than vector text on purpose. GitHub
renders README SVGs as `<img>`, which drops both `foreignObject` and embedded
`@font-face`, so a vector wordmark simply vanishes on the page it exists for.

## Product screenshots — not yet captured

`README.md` deliberately ships **no product screenshots**. The obvious shortcut — reusing
`selfhost/assets/media/*.png` — would be misleading: those are the self-hosted web UI, a
different application with a different shell. And screenshots taken from a desktop build
with no engine running show empty states and error toasts, which is worse than none.

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
