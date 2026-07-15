# EQ Tagging Pilot — jsPsych experiment

Free stack: **jsPsych** (experiment) + **GitHub Pages** (hosting) + **DataPipe → OSF** (data). No paid platform needed.

## Design: a 3-version comparison pilot

Each participant does all three response paradigms (order randomized), so you get a *paired* comparison of which feels least forced:

- **v1 — Choose:** hear reference + 4 context segments, pick the single best of 4 words (baseline + difference, both models).
- **v2 — Rate:** same segments, rate each of the 4 words 1–5 for how accurately it describes the reference (gives comparable scores, not ranks).
- **v3 — Match:** hear 5 EQ segments, assign each of 5 difference-method words to the segment it best describes (forced one-to-one). Tests whether people recover the contextual model's per-segment tagging.

After **every** trial a continuous **forcedness slider** (honest ↔ didn't fit) captures how forced the answer felt — this is the headline measure. A progress bar runs throughout, and nothing auto-advances: answers can be changed before clicking Continue.

Files:
- `index.html` — the whole experiment (consent, headphone check, instructions, practice, the 3 versions with force-listen + slider, per-version fatigue checks, final preference, data save).
- `experiment_trials.js` — the trials: practice + v1 (3) + v2 (3) + v3 (3), with URLs, words, method mapping, and v3 ground-truth tags. v1/v2 come from your Loop & Merge tables; v3 from the raw model files (tracks with 5 distinct difference tags).
- `lib/` — the jsPsych library, bundled locally (no CDN needed). **Keep this folder next to `index.html` and upload it too when hosting.**

---

## ⚠️ First: make the audio public

Your `survey_segments_7s` repo is **Private**. Participants' browsers can't load `raw.githubusercontent.com` files from a private repo, so the audio would fail for everyone but you.

Fix (simplest): on GitHub → `survey_segments_7s` → **Settings → General → Danger Zone → Change visibility → Public.** This exposes the EQ filenames (which you already accepted) but track *names* stay hidden. Nothing else changes — the URLs stay identical.

---

## Test it locally (30 seconds)

Just open `index.html` in a browser. Left as-is (`DATAPIPE_ID = "YOUR_DATAPIPE_ID"`), it runs the full experiment and **downloads a CSV of your data** at the end — perfect for checking it works.

**Note on audio during local testing:** the page (jsPsych) now works offline, but the *audio* still streams from your GitHub repo, which is **Private**. Until you make it Public (see below), the segments won't play even for you — the experiment will still run, just silent. Make the repo public first if you want to hear it.

Check: audio plays, the response stays locked until all 5 segments finish, you can't skip ahead, and the block order/counterbalancing works.

---

## Host it for participants (GitHub Pages)

1. Add `index.html` and `experiment_trials.js` to a **public** repo — easiest is to drop them into your now-public `survey_segments_7s` repo (audio + experiment in one place).
2. Repo → **Settings → Pages → Source: Deploy from branch → main → /(root) → Save.**
3. After a minute your experiment is live at:
   `https://mariachiaracavagna.github.io/survey_segments_7s/`
4. Share that link. Add `?id=SOMEID` to tag a participant, e.g. `…/?id=p01` (otherwise a random ID is assigned).

---

## Collect data (DataPipe → OSF, free)

1. Make a free account at **https://pipe.jspsych.org** and an OSF account at https://osf.io.
2. In DataPipe: create an experiment, connect it to an OSF project component, and **enable data collection**. Copy the **Experiment ID**.
3. In `index.html`, set `const DATAPIPE_ID = "YOUR_DATAPIPE_ID";` to that ID.
4. Now each participant's CSV is saved automatically to your OSF component. (If a save ever fails, the experiment falls back to a local download.)

**Pavlovia alternative:** if you'd rather use Pavlovia, you host the same jsPsych files on a Pavlovia GitLab repo and add the `@jspsych-contrib/plugin-pavlovia` init/finish trials instead of the DataPipe block. It costs credits per participant, so DataPipe is the free choice.

---

## What you get in the data

One row per screen. `participant_id` and `version_order` are on every row.

**Forcedness** is now recorded *on the same row as each trial* (`forcedness` column: 0 = felt completely honest, 1 = felt completely forced), since the slider sits at the bottom of each trial page and submits with the answer. Compare mean `forcedness` per version (paired across participants) — the headline measure.

**v1 — Choose** (`task="tag"`, `version="v1"`): `chosen_word`, `chosen_method`, `forcedness`, `trial_id`, `track_id`, `ref_eq`, `presented_order`, `rt`.

**v2 — Rate** (`task="tag"`, `version="v2"`): `ratings` — a `{method: 1–5}` object (5 = most accurate), `forcedness`, plus `trial_id`, `track_id`, `rt`.

**v3 — Match** (`task="match"`, `version="v3"`): `assigned` (word per segment in shown order), `seg_true` (correct tags), `n_correct`, `accuracy` (0–1), `exact_match` (all 5 right), `forcedness`, `model` (`clap_general`/`MuQMuLan`), plus `rt`. Chance ≈ 20% per segment, 0.8% for a perfect trial.

**Questionnaires:** per-version `tired` + `clear` (`task="post_version"`); final `most_honest`, `easiest`, `preferred` (`task="final_pref"`); open `feedback`.

`practice:true` marks the practice trial — exclude it. Compare v1's `chosen_method` against v2's highest-rated method for cross-check; `accuracy` is your v3 metric and works as a score comparable across the two models.
