# Animated coding heatmaps

This package generates two self-hosted SVGs with a one-time reveal animation using real public activity:
- GitHub contributions from the public contribution calendar.
- LeetCode submissions from the public profile calendar API.

## Install
1. Open https://github.com/AadityaRaj59/AadityaRaj59.
2. Extract this ZIP and upload `scripts/`, `.github/workflows/update-heatmaps.yml`, and `.gitignore` into the same paths in your profile repository.
3. Merge `README_SECTION.md` into your existing `README.md`; do not overwrite your bio unless intended.
4. Commit and push.
5. In repository Settings → Actions → General → Workflow permissions, enable read/write access to repository contents if restricted.
6. Open Actions → Refresh animated coding heatmaps → Run workflow.
7. On success, the workflow commits `assets/github-heatmap.svg` and `assets/leetcode-heatmap.svg`. Refresh your profile.

## Local run
Requires Python 3.10+:
```bash
python -m pip install -r scripts/requirements.txt
python scripts/generate_heatmaps.py
```

The generator intentionally fails if a source cannot be fetched or parsed. It does not invent activity or replace failures with empty charts. GitHub's public HTML and LeetCode's public GraphQL response can change or rate-limit; check the Actions logs if it fails. SVG animation support varies by renderer; if disabled, the heatmaps still render statically. GitHub contributions and LeetCode submissions are separate metrics.
