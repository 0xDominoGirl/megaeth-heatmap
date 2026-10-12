# MegaETH Contract Deployment Heatmap

A browser-based heatmap of MegaETH contract deployments, built with HTML, CSS,
JavaScript, and PixiJS. Hover over a cell for deployment counts, then click a
non-empty cell to inspect contracts and open their addresses in Blockscout.

## Run locally

Install Git and Python 3, then run:

```sh
git clone https://github.com/0xDominoGirl/megaeth-heatmap.git
cd megaeth-heatmap
python3 -m http.server 8000 --bind 127.0.0.1
```

Open <http://127.0.0.1:8000>. Stop the server with `Ctrl+C`.
There is no package installation or build step. Use an HTTP server rather than
opening `index.html` as a file: the page fetches its adjacent JSON data file.
An internet connection is needed for PixiJS and Google Fonts.

## Views and data

- **24H** displays hourly buckets across the last 14 days.
- **30D** groups daily totals into a month-by-day grid.
- The page reads `contract-deployments.json` on load. Reload after updating it.
- Deployment counts and unique deployers are calculated in the browser.
- The displayed update time comes from the JSON file, not a live RPC connection.

The JSON object contains `updated_at`, `total_contracts`, `query_id`, and a `data`
array of deployment rows. The frontend supports these column names:

| Value | Supported fields, in priority order |
| --- | --- |
| Timestamp | `block_time`, `timestamp`, `created_at` |
| Deployer | `deployer`, `from`, `tx_from`, `creator` |
| Contract address | `contract_address`, `address`, `created_contract` |
| Transaction hash | `tx_hash`, `hash`, `transaction_hash` |

Use valid UTC timestamps and actual deployment records. An empty `data` array
contains no deployment history; it should not be interpreted as evidence that
no contracts were deployed on the network.

## Refreshing the dataset

[Update Contract Deployments](.github/workflows/update-contract-deployments.yml)
fetches the saved results of Dune query `6883877`, writes the JSON snapshot, and
commits changes. It is scheduled for midnight UTC and supports manual runs from
the repository's **Actions** tab. The workflow fetches existing query results;
it does not execute the Dune query itself.

To use the workflow in a fork, configure an Actions repository secret named
`DUNE_API_KEY` with access to the query and allow the workflow to write repository
contents. Keep that key in Actions secrets, never in frontend code or commits.
Serving an existing snapshot locally does not require a Dune key.

## Troubleshooting

- **Blank canvas:** check the browser console and whether the PixiJS CDN loads.
- **Data load error:** serve the repository over HTTP and check that
  `/contract-deployments.json` returns valid JSON.
- **Empty or stale data:** inspect `data` and `updated_at`, then check the latest
  Actions run and the query's saved results. The current workflow can write an
  empty snapshot when a response contains no result rows.
- **No recent cells:** check the timestamps; the hourly view filters to the last
  14 days, and the frontend filters records before February 1, 2026.
