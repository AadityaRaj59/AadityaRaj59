#!/usr/bin/env python3
import json, os, re, sys
from datetime import date, datetime, timedelta, timezone
from html import escape
from pathlib import Path
import requests
from bs4 import BeautifulSoup

GH_USER = os.getenv("GITHUB_USERNAME", "AadityaRaj59")
LC_USER = os.getenv("LEETCODE_USERNAME", "Aaditya_Raj1101")
OUT = Path("assets")
OUT.mkdir(exist_ok=True)
BG, TEXT, MUTED, BORDER = "#0d1117", "#c9d1d9", "#8b949e", "#30363d"
PALETTE = ["#161b22", "#0e4429", "#006d32", "#26a641", "#39d353"]
S = requests.Session()
S.headers.update({"User-Agent": "AadityaRaj59-profile-heatmaps/1.0", "Accept": "text/html,application/json"})

def count_from_tooltip(text):
    m = re.search(r"([\d,]+)\s+(?:contributions?|submissions?)", text or "", re.I)
    return int(m.group(1).replace(",", "")) if m else 0

def github_calendar():
    r = S.get(f"https://github.com/users/{GH_USER}/contributions", timeout=30)
    r.raise_for_status()
    soup = BeautifulSoup(r.text, "html.parser")
    nodes = soup.select("[data-date][data-level]")
    if not nodes:
        raise RuntimeError("GitHub contribution cells not found; GitHub may have changed its HTML.")
    result = {}
    for node in nodes:
        try:
            day = date.fromisoformat(node.get("data-date", ""))
        except ValueError:
            continue
        tooltip = node.get("aria-label", "")
        title = node.find("title")
        if title:
            tooltip = title.get_text(" ", strip=True)
        result[day] = count_from_tooltip(tooltip)
    if not result:
        raise RuntimeError("GitHub returned no usable contribution dates.")
    return result

def leetcode_calendar():
    query = '''query userProfileCalendar($username: String!, $year: Int) {
      matchedUser(username: $username) {
        userCalendar(year: $year) { totalActiveDays submissionCalendar }
      }
    }'''
    url = "https://leetcode.com/graphql/"
    headers = {"Content-Type":"application/json", "Referer":f"https://leetcode.com/u/{LC_USER}/", "Origin":"https://leetcode.com"}
    combined = {}
    current_year = datetime.now(timezone.utc).year
    for year in (current_year - 1, current_year):
        r = S.post(url, headers=headers, json={"query":query, "variables":{"username":LC_USER, "year":year}}, timeout=30)
        r.raise_for_status()
        payload = r.json()
        if payload.get("errors"):
            raise RuntimeError("LeetCode GraphQL error: " + json.dumps(payload["errors"]))
        matched = (payload.get("data") or {}).get("matchedUser")
        if not matched:
            raise RuntimeError(f"LeetCode profile '{LC_USER}' was not returned; check username/profile visibility.")
        calendar = (matched.get("userCalendar") or {}).get("submissionCalendar")
        if not calendar:
            raise RuntimeError("LeetCode returned no submission calendar; it may be rate-limiting or its API may have changed.")
        calendar = json.loads(calendar) if isinstance(calendar, str) else calendar
        for ts, count in calendar.items():
            try:
                combined[datetime.fromtimestamp(int(ts), tz=timezone.utc).date()] = int(count)
            except (ValueError, TypeError, OverflowError):
                pass
    if not combined:
        raise RuntimeError("LeetCode calendar had no usable dates.")
    return combined

def intensity(count, positives):
    if count <= 0: return 0
    vals = sorted(positives)
    if not vals: return 1
    n = len(vals)
    if count <= vals[max(0, int(n*.25)-1)]: return 1
    if count <= vals[max(0, int(n*.50)-1)]: return 2
    if count <= vals[max(0, int(n*.75)-1)]: return 3
    return 4

def render(title, username, data, filename, noun, accent):
    end = datetime.now(timezone.utc).date()
    start = end - timedelta(days=364)
    days = [start + timedelta(days=i) for i in range(365)]
    counts = [data.get(d, 0) for d in days]
    positives = [v for v in counts if v > 0]
    total, active = sum(counts), sum(v > 0 for v in counts)
    padded = [None] * start.weekday() + days
    cols = (len(padded)+6)//7
    cell, gap, left, top = 11, 3, 24, 65
    width, height = left + cols*(cell+gap)+22, 153
    rects = []
    for idx, day in enumerate(padded):
        if day is None: continue
        col, row = idx//7, idx%7
        val = data.get(day, 0)
        x, y = left + col*(cell+gap), top + row*(cell+gap)
        delay = min(idx*.008, 3.0)
        tip = escape(f"{val} {noun} on {day.isoformat()}")
        rects.append(
          f'<rect x="{x}" y="{y}" width="{cell}" height="{cell}" rx="2" fill="{PALETTE[intensity(val, positives)]}" stroke="{BORDER}" stroke-width=".35">'
          f'<title>{tip}</title><animate attributeName="opacity" values="0;1" dur=".32s" begin="{delay:.3f}s" fill="freeze"/>'
          f'<animateTransform attributeName="transform" type="translate" values="0 3;0 0" dur=".32s" begin="{delay:.3f}s" fill="freeze"/></rect>'
        )
    fy = top + 7*(cell+gap)+19
    lx = width-126
    legend = "".join(f'<rect x="{lx+i*15}" y="{fy-8}" width="10" height="10" rx="2" fill="{c}"/>' for i,c in enumerate(PALETTE))
    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{width}" height="{height}" viewBox="0 0 {width} {height}" role="img">
<title>{escape(title)}</title><desc>Real public activity for {escape(username)}: {total} {noun} and {active} active days in the last 365 days.</desc>
<rect width="100%" height="100%" rx="10" fill="{BG}" stroke="{BORDER}"/>
<text x="18" y="23" fill="{TEXT}" font-family="Segoe UI,Arial,sans-serif" font-size="14" font-weight="600">{escape(title)}</text>
<text x="18" y="41" fill="{MUTED}" font-family="Segoe UI,Arial,sans-serif" font-size="10">@{escape(username)}</text>
<text x="18" y="55" fill="{accent}" font-family="Segoe UI,Arial,sans-serif" font-size="9.5" font-weight="600">{total:,} {noun} · {active} active days · last 365 days</text>
{''.join(rects)}
<text x="18" y="{fy}" fill="{MUTED}" font-family="Segoe UI,Arial,sans-serif" font-size="9">Less</text>{legend}
<text x="{lx+78}" y="{fy}" fill="{MUTED}" font-family="Segoe UI,Arial,sans-serif" font-size="9">More</text></svg>'''
    (OUT/filename).write_text(svg, encoding="utf-8")
    print(f"Generated {filename}: {total} {noun}, {active} active days")

def main():
    try:
        print(f"Fetching GitHub activity for {GH_USER}")
        gh = github_calendar()
        print(f"Fetching LeetCode activity for {LC_USER}")
        lc = leetcode_calendar()
        render("GitHub Contributions", GH_USER, gh, "github-heatmap.svg", "contributions", "#39d353")
        render("LeetCode Submissions", LC_USER, lc, "leetcode-heatmap.svg", "submissions", "#58a6ff")
    except Exception as e:
        print(f"ERROR: {e}", file=sys.stderr)
        print("No fake activity or empty fallback chart was written.", file=sys.stderr)
        return 1
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
