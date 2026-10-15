# StudySync (AI110 Tinker Starter)

Starter code for the AI110 **Tinker 1B / 2B / 3B** in-class activities. All three tinkers
build on this one project, each working in a different module.

| Tinker | Focus | Files you'll work in |
|---|---|---|
| 1B, Split the Logic | Writing your first `pytest` test, then a cross-file refactor | `scoring.py`, `scoring_helpers.py`, `test_scoring.py` |
| 2B, Wire It Up | Streamlit `session_state`, input validation, dataclasses, recurring dates | `sessions.py`, `app.py` |
| 3B, Rank & Explain | CSV loading, weighted scoring that returns reasons, ranking, a data-flow diagram | `ranking.py`, `data/study_spots.csv`, `diagram.mmd` |

## Setup

Fork this repo, then clone **your fork**:

```bash
git clone https://github.com/<YOUR-GITHUB-USERNAME>/ai110-tinker-studysync-starter.git
cd ai110-tinker-studysync-starter
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
```

macOS, Linux, Git Bash or WSL:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies and run:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m streamlit run app.py
```

Run the tests:

```bash
pytest
```

## Files

- `app.py` — Streamlit entry point, wires the three tabs together
- `scoring.py` — `session_rating()` and the Session Scorer tab
- `scoring_helpers.py` — helpers extracted during Tinker 1B
- `sessions.py` — session log, the Tinker 2B surface
- `ranking.py` — study-spot ranking, the Tinker 3B surface
- `data/study_spots.csv` — sample data for ranking
- `diagram.mmd` — Mermaid data-flow diagram, completed in Tinker 3B
- `test_scoring.py` — starting point for your own tests

## Tinker 1B: edge-case notes

The Session Scorer combines two sliders (minutes 0-60 + focus 0-60), so a real
combined score is always 0-120, and the streak is a non-negative integer. I ran
extra inputs through `apply_streak_bonus()` -> `session_rating()` (streak = 3,
so the bonus is +6) via `run_demo()` in `scoring.py`:

| Raw | Boosted | Rating | Verdict |
|---|---|---|---|
| -55 | -49 | Skip | Not reachable from the UI (sliders start at 0). Still lands on the right rating ("Skip"), so no bug. |
| 168 | 100 | Great | Above the real max of 120. The 100 cap in `apply_streak_bonus()` handles it. No bug. |
| 82.5 | 88.5 | Good | Decimals work because the `>=` checks compare numbers. No bug. |
| 91 | 97 | Great | Normal case. |
| 77 | 83 | Good | Normal case. |

Also checked directly on `session_rating()`: `-5` gives "Skip", `150` gives "Great",
`87.5` gives "Good". Non-numeric input (`"90"`, `None`) raises `TypeError`, but the
app never passes those (the sliders and number input only produce numbers).

**Changes made:** none to the logic. Every out-of-range numeric input still gives
a sensible rating, and the crashing inputs can't be produced by the UI. Adding
validation would be guarding against cases that can't happen, so I left the functions
as they are. The edge-case loop stays in `run_demo()` as a record.

**Move refactor:** `apply_streak_bonus()` now lives only in `scoring_helpers.py`;
`scoring.py` imports it, and the duplicate definition was removed.

## Tinker 3B: weighting notes

`score_study_spot()` in `ranking.py` mixes a binary factor with two graduated
ones: noise is a flat +2 (pass) / -1 (fail) switch, while distance (up to +2)
and seats (+1, plus up to +0.5 for surplus) scale smoothly with how well a
spot does. That mismatch matters.

Because noise never partially credits anything, it acts as a hard gate that
the graduated factors can't out-compete. A spot one noise-level over the
limit always eats a flat -1, even if it's the closest option on the list
with seats to spare 

 a moderate-noise spot that's farther away and has
fewer seats can still beat it, because its noise "pass" was worth +2
outright rather than a sliding score.

Concretely, the spot type systematically buried is **closest-but-noisier**:
e.g. Courtyard Cafe (loud, 0.2mi) will never outrank a quieter spot that's
farther away, no matter how close Courtyard Cafe is, because the noise
penalty swamps any amount of distance credit it could otherwise earn.

Tying this to a filter bubble: each individual score is "correct" by the
rubric, but because the noise gate is binary while the other factors are
smooth, the ranking systematically filters out an entire category of real
options close, noisy spots  before the user ever sees them, narrowing
their perceived choices to only the "well-behaved on every axis" spots
rather than surfacing the genuine distance/noise tradeoff they might
actually prefer.

Your instructor will tell you which tinker you're on. Follow the activity on the course portal.
