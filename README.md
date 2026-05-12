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

Visit the root URL to see the 10 most recent traces by pipeline ID
(`trace-<id>.json`). Click any row to open the trace in Perfetto UI.

```
https://eic.github.io/perfetto-launcher/
```

### Open a specific trace

Append `?trace=<path>` where `<path>` is relative to this site's root:

```
https://eic.github.io/perfetto-launcher/?trace=traces/trace-143108.json
```

## Trace index

`traces/index.json` is **not stored in this repository**. It is generated
automatically by the [Deploy to GitHub Pages](.github/workflows/deploy.yml)
workflow whenever a new trace is pushed to `main`. The workflow scans
`traces/trace-<id>.json`, sorts by numeric `<id>` descending, and writes the
top entries into `traces/index.json` in the deployment artifact.

The format is intentionally extensible:

```json
{
  "traces": [
    {
      "path": "traces/trace-143108.json",
      "pipeline_id": "143108"
    }
  ]
}
```

### Retention and archive policy

The active repository keeps only the newest 1000 trace files
(`traces/trace-<id>.json` by numeric ID). Older traces are archived to GitHub
Releases by the [Archive and compact trace history](.github/workflows/archive-and-compact.yml)
workflow (scheduled monthly, and runnable manually).

When archival runs:

1. Older traces are bundled as `traces-<minID>-<maxID>.tar.gz`.
2. The tarball is published as a release asset under a tag like
   `archive/<timestamp>`.
3. Archived traces are removed from the active tree.
4. `main` is compacted to a fresh single-commit history and force-pushed.

Compaction is aborted if release upload fails, and also aborted if `origin/main`
changes while the workflow is running.

### Recovery

To recover archived traces, download the desired release asset and extract:

```sh
tar -xzf traces-<minID>-<maxID>.tar.gz
```

## Uploading traces

Traces are uploaded by [`waterfall-eicweb.sh`](https://github.com/eic/snippets/blob/main/waterfall-eicweb.sh)
from [eic/snippets](https://github.com/eic/snippets) using the GitHub
Contents API.  Uploading a trace commits it to `main`, which automatically
triggers the deploy workflow to regenerate the index and redeploy the site.
A `GITHUB_PAGES_TOKEN` with `contents:write` permission on this repository
is required.

```sh
# Upload the trace for EIC pipeline 143108 in project 290
GITHUB_PAGES_TOKEN=<token> ./waterfall-eicweb.sh --pages 290 143108
```

The CI integration in [eic/containers](https://github.com/eic/containers)
runs this automatically after every pipeline via the `waterfall:upload` job,
using the `EIC_PERFETTO_LAUNCHER_GITHUB_PAGES_TOKEN` CI/CD variable.

## Repository layout

```
.github/workflows/deploy.yml   # Builds site + generates traces/index.json
.github/workflows/archive-and-compact.yml  # Archives old traces and compacts main history
index.html                     # Launcher / listing page (GitHub Pages entry point)
traces/
  trace-<id>.json              # Perfetto JSON trace files
```
