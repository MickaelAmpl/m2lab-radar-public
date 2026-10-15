# M2 Lab Radar

A niche radar for an Instagram account about AI in business. Twice a week it collects public
reel metrics across a set of accounts in the niche, scores every reel against its own author's
norm, takes the top reels apart frame by frame, transcribes the speech, tags topics and proposes
shooting cards with angles and hooks.

**The machine counts and narrows, people read and decide.**

This is the public copy of the code. The working repository also holds the collected data,
frames and transcripts of other people's reels, and our own positioning and content plan.
Those stay private, so this copy ships the code only, with a fresh history.

## How a run works

```
roster -> collect -> score -> deep pass on the top -> topics -> cards -> delta -> pages
```

1. **Roster** (`roster.py`, `judge_accounts.py`, `harvest.py`): the set of accounts in the niche,
   with liveness, dropout and top-up from neighbouring accounts.
2. **Collect** (`collect_snapshot.py`, `lib/hiker.py`): pulls reels into a dated snapshot through
   HikerAPI. The balance is checked before the run and every paid call is logged.
3. **Score** (`score.py`, `baseline.py`): every reel is compared to its own author's median,
   inside an age band, so a small account's hit counts as a hit and a big account's average
   post does not.
4. **Deep pass** (`deep.py`, `blocks.py`): frames, contact sheet, cut count and local
   transcription for the top of the freshness window. It runs in the same pass as the
   collection, because Instagram media links expire within hours.
5. **Topics and cards** (`tag_topics.py`, `topics.py`, `cards.py`, `content_blocks.py`):
   topic tags and a weekly shortlist; an agent (Claude Code on the server) writes angles and
   hooks from the frames and transcripts, using the prompts in `prompts/`.
6. **Delta and pages** (`delta.py`, `pages.py`): what moved since the last snapshot, and three
   weekly HTML pages: what to shoot, the niche over 14 days, the numbers.
7. **Notion** (`notion.py`, `notion_db.py`, `engine/notion_sync.py`): optional feedback loop;
   status and comments come back and change the next selection.

`engine/` is the content engine built on top: corpus statistics, insights, the card schema,
storyboards and an edit decision list for video.

## Run it

```bash
cp .env.example .env        # fill in HIKER_KEY; Notion is optional
python3 run.py              # estimate: what happens and what it costs
python3 run.py --yes        # the run itself
```

On the server it runs from cron, Monday and Thursday at 07:00 (`cron.sh`, `install-cron.sh`).

## Checks

```bash
python3 -m pytest -q -m "not slow"
for t in test_*.py; do python3 "$t"; done
```

307 pytest tests plus 12 standalone check scripts pass on this copy. A few tests that need
the private folders are deselected in `pytest.ini`.

## Money

HikerAPI, about $0.02 per request; a run over ~130 accounts costs a few dollars. Frames,
transcription, tagging and pages are local and free. Keys live in `.env`, which is never
committed.

## Credits

Built with Claude Code by [Mykhailo Amplieiev](https://www.linkedin.com/in/mykhailoamplieiev/),
together with [Max](https://github.com/mcharniuk1-spec) on parts of the pipeline.
