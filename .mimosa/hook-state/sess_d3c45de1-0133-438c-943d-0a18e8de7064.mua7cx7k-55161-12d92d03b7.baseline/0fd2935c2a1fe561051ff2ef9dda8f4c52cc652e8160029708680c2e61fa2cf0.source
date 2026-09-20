#!/usr/bin/env python3
"""
DSA Plan Progress Tracker
=========================
Scans days/*.md, counts checkboxes, and rewrites the stats block in README.md
between the TRACKER:BEGIN / TRACKER:END markers. Stdlib only.

Usage:
    python3 tracker.py          # rewrite README dashboard + print summary
    python3 tracker.py --dry    # print summary only, don't touch README

Counting rules (keep day files in the agreed format or counts drift):
  - Problem checkbox : line starts with "- [ ] N." / "- [x] N."   (N = 1..9)
  - Concept checkbox : line starts with "- [ ] Read:" | "Watch:" | "Do:"
  - Redo flag        : checkbox line (any indent) containing "🔁"
  - Day status       : ✅ all boxes checked · 🟡 some checked · ⬜ untouched · 🌙 no boxes (rest)
"""
import re
import sys
import datetime as dt
from pathlib import Path

ROOT = Path(__file__).resolve().parent
DAYS_DIR = ROOT / "days"
README = ROOT / "README.md"
BEGIN, END = "<!-- TRACKER:BEGIN -->", "<!-- TRACKER:END -->"

PROBLEM_RE = re.compile(r"^\s*- \[([ xX])\] \d+\.", re.M)
CONCEPT_RE = re.compile(r"^\s*- \[([ xX])\] (?:Read|Watch|Do):", re.M)
ANYBOX_RE = re.compile(r"^\s*- \[([ xX])\]", re.M)
REDO_RE = re.compile(r"^\s*- \[([ xX])\][^\n]*🔁", re.M)

START = dt.date(2026, 9, 26)
FINAL = dt.date(2026, 12, 27)

PHASES = [
    ("Phase 1 · Foundations", dt.date(2026, 9, 26), dt.date(2026, 10, 25)),
    ("Phase 2 · Core Data Structures", dt.date(2026, 10, 26), dt.date(2026, 11, 22)),
    ("Phase 3 · Advanced & Interview Mode", dt.date(2026, 11, 23), dt.date(2026, 12, 27)),
]

WEEKS = []  # (label, [dates])
WEEKS.append(("Kickoff", [dt.date(2026, 9, 26), dt.date(2026, 9, 27)]))
for i in range(13):
    mon = dt.date(2026, 9, 28) + dt.timedelta(days=7 * i)
    WEEKS.append((f"Week {i + 1}", [mon + dt.timedelta(days=d) for d in range(7)]))


def scan_file(path: Path):
    text = path.read_text(encoding="utf-8")
    problems = PROBLEM_RE.findall(text)
    concepts = CONCEPT_RE.findall(text)
    boxes = ANYBOX_RE.findall(text)
    redos = [b for b in REDO_RE.findall(text) if b.lower() == "x"]  # only CHECKED struggle flags
    p_done = sum(1 for b in problems if b.lower() == "x")
    c_done = sum(1 for b in concepts if b.lower() == "x")
    total_boxes = len(boxes)
    done_boxes = sum(1 for b in boxes if b.lower() == "x")
    open_redos = len(redos)  # checked 🔁 flags = the Saturday redo queue
    if total_boxes == 0:
        status = "🌙"
    elif done_boxes == total_boxes:
        status = "✅"
    elif done_boxes > 0:
        status = "🟡"
    else:
        status = "⬜"
    return {
        "date": dt.date.fromisoformat(path.stem),
        "problems": len(problems),
        "problems_done": p_done,
        "concepts": len(concepts),
        "concepts_done": c_done,
        "status": status,
        "open_boxes": total_boxes - done_boxes,
        "redo_flags": open_redos,
    }


def bar(done: int, total: int, width: int = 20) -> str:
    if total == 0:
        return "░" * width + " 0%"
    frac = done / total
    filled = round(frac * width)
    return "█" * filled + "░" * (width - filled) + f" {round(frac * 100)}%"


def main(dry=False):
    files = sorted(DAYS_DIR.glob("*.md"))
    days = [scan_file(f) for f in files]
    by_date = {d["date"]: d for d in days}

    total_p = sum(d["problems"] for d in days)
    done_p = sum(d["problems_done"] for d in days)
    total_c = sum(d["concepts"] for d in days)
    done_c = sum(d["concepts_done"] for d in days)
    done_days = sum(1 for d in days if d["status"] == "✅")
    active_days = sum(1 for d in days if d["problems"] > 0)
    redos = sum(d["redo_flags"] for d in days)

    today = dt.date.today()
    remaining = (FINAL - today).days if today <= FINAL else 0

    lines = []
    lines.append("### 📊 Current progress")
    lines.append("")
    lines.append(f"- **Overall:** {bar(done_p, total_p)} — **{done_p} / {total_p} problems solved** ({total_p - done_p} to go)")
    lines.append(f"- **Concepts studied:** {bar(done_c, total_c)} — {done_c} / {total_c}")
    lines.append(f"- **Days fully completed:** {done_days} / {len(days)} · Days with problems: {active_days}")
    lines.append(f"- **🔁 Redo queue (flagged problems):** {redos}")
    if today < START:
        lines.append(f"- **Starts in** {(START - today).days} days — first file: [days/{START.isoformat()}.md](days/{START.isoformat()}.md)")
    elif today <= FINAL:
        cur = by_date.get(today)
        cur_txt = f"today is a {'rest' if cur and cur['status'] == '🌙' else 'work'} day" if cur else ""
        lines.append(f"- **Today:** [{today.isoformat()}](days/{today.isoformat()}.md) — {cur_txt} · **{remaining} days remain**")
    else:
        lines.append(f"- **Program window ended** {FINAL.isoformat()} — keep the streak alive with impromptu days (see templates/)")
    lines.append("")

    lines.append("### 🗺️ Phase status")
    lines.append("")
    lines.append("| Phase | Problems | Progress |")
    lines.append("|---|---|---|")
    for name, lo, hi in PHASES:
        pdays = [d for d in days if lo <= d["date"] <= hi]
        p_tot = sum(d["problems"] for d in pdays)
        p_done = sum(d["problems_done"] for d in pdays)
        lines.append(f"| {name}<br>{lo} → {hi} | {p_done}/{p_tot} | `{bar(p_done, p_tot)}` |")
    lines.append("")

    lines.append("### 📅 Week-by-week")
    lines.append("")
    lines.append("| Week | Mon | Tue | Wed | Thu | Fri | Sat | Sun |")
    lines.append("|---|---|---|---|---|---|---|---|")
    for label, dates in WEEKS:
        cells = []
        week_has_today = False
        for dte in dates:
            cell = "—"
            if dte in by_date:
                d = by_date[dte]
                cell = f"[{d['status']}](days/{dte.isoformat()}.md)"
                if dte == today:
                    week_has_today = True
                    cell = f"**{cell}**"
            cells.append(cell)
        prefix = f"**{label}**" if week_has_today else label
        if label == "Kickoff":
            cells = ["—", "—", "—", "—", "—"] + cells  # pad to Mon–Fri columns; Sat/Sun carry the links
        lines.append(f"| {prefix} | " + " | ".join(cells) + " |")
    lines.append("")
    lines.append("_Legend:_ ✅ done · 🟡 in progress · ⬜ untouched · 🌙 rest · **bold** = current week. _Re-run `python3 tracker.py` after checking boxes._")

    block = "\n".join(lines)
    if not dry:
        text = README.read_text(encoding="utf-8")
        if BEGIN not in text or END not in text:
            sys.exit("README.md is missing TRACKER markers — aborting instead of corrupting it.")
        pre, rest = text.split(BEGIN, 1)
        _, post = rest.split(END, 1)
        README.write_text(pre + BEGIN + "\n" + block + "\n" + END + post, encoding="utf-8")

    print(f"{done_p}/{total_p} problems · {done_c}/{total_c} concepts · {done_days}/{len(days)} days complete · {redos} redos flagged"
          + ("" if dry else " → README.md updated"))


if __name__ == "__main__":
    main(dry="--dry" in sys.argv)
