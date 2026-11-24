[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/brianwang1226/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/brianwang1226/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Token Radar

A token launch radar: it reads a sealed snapshot of every protocol DefiLlama tracks, ranks the
protocols that record no token yet by what their launch would be worth, and hands in a web
dashboard with the board, a detail view for each protocol and the steps to farm it.

    mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>

`main.ts` (Node 22, standard library only) reads `/input/snapshot.json`, writes the ranked board
to `/output/board.json`, writes the dashboard's data beside the page, copies `dashboard/` and
`screenshots/` to `/output`, and writes `/output/dossier.json` with `buildDossier` from
`vendor/makefun-board.ts`.

## The model (radar-2)

Every row is scored by one function of the snapshot's own figures. No protocol name is
special-cased, and no row is hand-picked.

    implied = round( tvl_usd x 0.25
      x (1 + clamp(change_7d, -60, 120) / 100 x 0.6)
      x (0.65 + min(age_days, 1095) / 1095 x 0.55)
      x (1 + 0.04 x min(chains - 1, 5)) )

TVL and seven-day growth come from the snapshot. Age is the time since the protocol was first
listed, the best record of how long it has run with no token; an unlisted age uses the snapshot's
median pre-token age. A protocol on more chains earns a small reach bonus. The board covers the
whole 40-protocol watchlist and adds every rankable pre-token protocol below the $50M floor: 226
rows in all, 186 of them early.

## The dashboard

A static page, no framework and no network: `dashboard/index.html`, `dashboard/app.css` and
`dashboard/app.js` read `data.js`. It sorts by rank, TVL, growth, age or implied value; filters
by search, category, chain, watchlist-only and newly listed (last 90 days); shows a live-token
view set apart from the pre-token ranking; lists the five fastest seven-day movers; and opens a
detail view for each protocol with its model breakdown and farm plan. Each farm plan is seven
steps derived from the protocol's own record (category, chains, TVL, age), with what each step
costs and how long it takes, and a last step against fake claim links.

Built for the make.fun "Build the best token launch radar" bounty. Implied value is a model,
not a promise: it does not predict whether or when a token is issued, and a protocol is listed
as pre-token because DefiLlama records no token for it, never because it is known to have none.
