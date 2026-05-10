# EIC Perfetto Trace Launcher

A lightweight [GitHub Pages](https://pages.github.com/) site that opens
EIC CI pipeline traces in [Perfetto UI](https://ui.perfetto.dev) with a
single click.

Live at: **https://eic.github.io/perfetto-launcher/**

## How it works

GitHub Pages cannot set `Access-Control-Allow-Origin` headers, so traces
hosted here cannot be loaded by Perfetto UI directly via its `?url=` deep
link.  Instead, this site acts as an intermediary:

1. `index.html` fetches the trace file (same-origin, no CORS needed).
2. It opens `https://ui.perfetto.dev` in a new tab.
3. It pushes the trace data via the
   [Perfetto `postMessage` API](https://perfetto.dev/docs/visualization/deep-linking-to-perfetto-ui)
   (`PING` / `PONG` handshake, then `{perfetto: {buffer, title, fileName}}`).

## Usage

### Browse recent traces

Visit the root URL to see the 10 most recent uploaded traces with their
upload timestamps.  Click any row to open the trace in Perfetto UI.

```
https://eic.github.io/perfetto-launcher/
```

### Open a specific trace

Append `?trace=<path>` where `<path>` is relative to this site's root:

```
https://eic.github.io/perfetto-launcher/?trace=traces/trace-143108.json
```

## Trace index

`traces/index.json` is updated automatically on every upload and drives
the default listing page.  Its format is intentionally extensible:

```json
{
  "traces": [
    {
      "path": "traces/trace-143108.json",
      "uploaded_at": "2026-05-10T17:45:00Z",
      "pipeline_id": "143108"
    }
  ]
}
```

Additional metadata fields (e.g. a backlink to the eicweb GitLab pipeline)
can be added to each entry without breaking existing consumers.

## Uploading traces

Traces are uploaded by [`waterfall-eicweb.sh`](https://github.com/eic/snippets/blob/main/waterfall-eicweb.sh)
from [eic/snippets](https://github.com/eic/snippets) using the GitHub
Contents API.  A `GITHUB_PAGES_TOKEN` with `contents:write` permission on
this repository is required.

```sh
# Upload the trace for EIC pipeline 143108 in project 290
GITHUB_PAGES_TOKEN=<token> ./waterfall-eicweb.sh --pages 290 143108
```

The CI integration in [eic/containers](https://github.com/eic/containers)
runs this automatically after every pipeline via the `waterfall:upload` job,
using the `EIC_PERFETTO_LAUNCHER_GITHUB_PAGES_TOKEN` CI/CD variable.

## Repository layout

```
index.html          # Launcher / listing page (GitHub Pages entry point)
traces/
  index.json        # Metadata index of uploaded traces (latest first)
  trace-<id>.json   # Perfetto JSON trace files
```
