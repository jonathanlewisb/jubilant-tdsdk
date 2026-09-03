# SDK Tests

A minimal single-page test harness for the [Treasure Data JavaScript SDK](https://github.com/treasure-data/td-js-sdk) (`td.min.js` v4.4). It loads the SDK on a bare Bootstrap page and fires a single pageview event so SDK behaviour — signed mode, cross-domain tracking, global ID — can be verified end to end against a live TD database.

## What it does

`index.html` is the entire application:

1. Loads Bootstrap 4 CSS/JS, jQuery slim and Popper from CDN (with SRI hashes).
2. Injects the TD JS SDK loader snippet, which async-loads `//cdn.treasuredata.com/sdk/4.4/td.min.js`.
3. Instantiates a `Treasure` client (`index.html:25`) pointed at the EU region.
4. Inside `td.ready()` (`index.html:36`) — required, because `getTrackValues()` does not exist until `td.min.js` has finished loading — it:
   - calls `td.setSignedMode()`,
   - sets the `td_global_id` global value for cross-domain tracking,
   - calls `td.trackPageview('pageview_test')`.

## Configuration

The client config lives inline in `index.html`:

| Option | Value |
| --- | --- |
| `host` | `eu01.in.treasuredata.com` |
| `database` | `nestle_jlb_test` |
| `writeKey` | **placeholder — must be replaced** |
| `startInSignedMode` | `true` |
| `trackCrossDomain` | `true` |
| Target table | `pageview_test` |

> **Heads up:** `writeKey` is currently the literal string `'process.env.TREASUREDATA_WRITE_KEY'`. There is no build step here, so nothing substitutes that value — the page will not deliver events until you paste a real write-only key in its place. Because this is client-side code, use a **write-only** key, never a master key.

## Running it

No build, no dependencies to install. Serve the directory over HTTP (the SDK and CDN assets misbehave over `file://`):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Or use any static server you prefer (`npx serve`, `php -S`, etc.).

### Verifying events

1. Open the page with devtools on the Network tab and look for a request to `eu01.in.treasuredata.com`.
2. Confirm the row lands in TD:

```sql
SELECT * FROM nestle_jlb_test.pageview_test
WHERE TD_INTERVAL(time, '-1h', 'UTC')
ORDER BY time DESC
```

## Deployment

`.github/workflows/github-actions-demo.yml` publishes the repository root to GitHub Pages on every push to `main`. Since the whole project directory is uploaded as the Pages artifact, don't commit anything to the repo root that shouldn't be publicly served.

## Project layout

```
.
├── index.html                              # the whole test page + SDK setup
├── .github/workflows/github-actions-demo.yml  # GitHub Pages deploy
└── .claude/                                # local Claude Code settings
```
