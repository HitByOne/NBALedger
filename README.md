# NBA Ledger

The Gridiron Ledger, rebuilt for basketball: a GitHub Pages dashboard whose data refreshes itself from ESPN every day.

Same setup as the NFL repo: Python scripts pull box scores into `site/data/*.csv`, `site/index.html` reads those CSVs on every page load, and a GitHub Actions workflow runs the scripts and redeploys the site. No build step, no API key for the stats.

## Set it up (about 10 minutes)

1. **Create a new, empty GitHub repo** (no README), e.g. `nbadashboard`.
2. **Push these files to it** from the command line (the web uploader skips the hidden `.github` folder):
   ```
   cd nba_ledger
   git init && git add . && git commit -m "NBA Ledger"
   git branch -M main
   git remote add origin https://github.com/<you>/nbadashboard.git
   git push -u origin main
   ```
3. **Turn on Pages:** repo Settings -> Pages -> Build and deployment -> Source = **GitHub Actions**.
4. **Prop prices (optional):** Settings -> Secrets and variables -> Actions -> New repository secret `RAPIDAPI_KEY`. The same key the Gridiron Ledger uses works.
5. **Run it once:** Actions tab -> "Daily NBA Dashboard Refresh" -> Run workflow (mode: full). The first run pulls the whole season so far, then deploys. The site is at `https://<you>.github.io/nbadashboard/`.
6. **Friends / synced picks (optional):** in the same Supabase project the Gridiron Ledger uses, open the SQL Editor and run `supabase_nba.sql` once. Your sign-in and friend groups are shared with the NFL page; NBA picks and tickets live in their own tables. `site/config.js` already holds the project URL and public key.

## Schedule

| When (UTC) | Central | What runs |
|---|---|---|
| 11:00 | 6am | Full refresh: last night's box scores, team tables, ratings, backtest, milestones, injuries, tonight's slate |
| 19:00 | 2pm | Quick refresh: injury report, tonight's lines, player props |

A date is only saved once every game on it is final, so a late West Coast game never leaves a half-finished night in the data.

## How it differs from the NFL version

- **Game days instead of weeks.** Each date with games gets a number (1 = opening night); picks, records and history are keyed by it, and the page shows the date.
- **Positions are ESPN's G / F / C**, and **starters are real**: the box score marks who started. For an upcoming game the projected five are the players with the most starts in the team's last 5 games, with Out players replaced by the next man up.
- **Efficiency replaces EPA:** points per 100 possessions, pace and the four factors (`09_team_efficiency.py`). Play Style replaces Coaching Tendencies.
- **Schedule Load is about back-to-backs** and 3-in-4 / 4-in-6 stretches.
- **Double-Double Props replaces Anytime TD.**
- **A player who doesn't play is VOID**, not a miss, as at the sportsbook.
- Dropped because they're football-only: Line Play, Defense Field, Film Room, Mike's Template.

## Run it by hand

```
pip install pandas requests
python3 run_daily.py                    # full refresh
python3 run_daily.py --quick            # injuries + tonight's slate
python3 run_daily.py --date 2026-11-03  # re-pull one date, rebuild everything after it
RAPIDAPI_KEY=xxxx ODDS_FORCE=1 python3 17_prop_odds.py
cd site && python3 -m http.server       # then open http://localhost:8000
```

## Files

| Script | Writes |
|---|---|
| `01_initial_pull.py` / `02_daily_update.py` | box scores: `player_game_stats`, `team_box`, `games` |
| `03_build_team_tables.py` | `team_game_stats` (pace, ratings, rest), `team_summary` |
| `04_strength_of_schedule.py` | `strength_of_schedule` |
| `05_defense_vs_position.py` | `defense_vs_position` (G / F / C, all and starters) |
| `06_matchup_preview.py` | `upcoming_matchups` (tonight + next date, lines, rest) |
| `07_milestone_watch.py` | `milestones`, `team_milestones` |
| `08_injury_report.py` | `injury_report` |
| `09_team_efficiency.py` | `team_efficiency`, `team_style` |
| `16_backtest.py` | `backtest` (Best Picks replayed on past nights) |
| `17_prop_odds.py` | `prop_odds`, `prop_odds_status` |

Everything comes from ESPN's public site API, the same source the Gridiron Ledger uses. This dashboard is not affiliated with the NBA.
