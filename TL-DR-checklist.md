Quarto → GitHub Actions → Artifacts (and How to Debug It Fast)

A compact, copy-pastable playbook you can drop into an LLM session or follow yourself. It assumes you’re competent, prefers verify-first checks, and uses single, reliable commands (no here-docs).

⸻

TL;DR Checklist
	1.	Local render first

quarto render quarto_llm_cheatsheet.qmd --to html --no-execute

	2.	Fix shortcode traps (show literally, don’t execute):

	•	Replace {{< include _file.qmd >}} with {{</* include _file.qmd */>}} (same for embed, video, etc.).

	3.	Satisfy cross-refs without executing code (e.g., @fig-scatter):

![Scatter of X vs Y](images/placeholder.svg){#fig-scatter fig-cap="Scatter of X vs Y"}

(Add a tiny SVG placeholder file if needed.)
	4.	Minimal CI workflow (YAML below) + push.
	5.	Use gh to inspect runs & download artifacts (commands below).
	6.	If gh can’t write cache: set XDG_CACHE_HOME="$HOME/.gh-cache" or fix ~/.cache ownership.

⸻

1) Local Render (fail fast)

quarto --version
quarto render quarto_llm_cheatsheet.qmd --to html --no-execute
open quarto_llm_cheatsheet.html   # macOS (optional)

	•	--no-execute avoids kernel/engine issues while you’re getting the doc structure right.
	•	If it fails, fix here before touching CI.

Common local failure & fix

Problem: ERROR: Include directive failed … could not find _file.qmd
Why: Shortcodes like {{< include … >}} are expanded before code fences; your sample tries to run.
Fix: Comment the shortcode so it renders as text:

{{</* include _file.qmd */>}}
{{</* embed notebook.ipynb#cell-id */>}}
{{</* video video.mp4 */>}}


⸻

2) Cross-refs Without Execution

If your text references @fig-scatter but you’re not executing code, add a literal figure:
	1.	Make a tiny placeholder:

mkdir -p images
cat > images/placeholder.svg <<'EOF'
<svg xmlns="http://www.w3.org/2000/svg" width="320" height="200">
  <rect width="100%" height="100%" fill="#ffffff"/>
  <text x="12" y="24" font-size="14" fill="#333">Placeholder figure</text>
  <circle cx="60" cy="150" r="4" fill="#333"/>
  <circle cx="150" cy="110" r="4" fill="#333"/>
  <circle cx="260" cy="70" r="4" fill="#333"/>
  <line x1="40" y1="160" x2="280" y2="60" stroke="#999"/>
</svg>
EOF

	2.	Insert the figure in your QMD near the figures section:

![Scatter of X vs Y](images/placeholder.svg){#fig-scatter fig-cap="Scatter of X vs Y"}

Re-render locally to confirm the warning disappears.

⸻

3) Minimal GitHub Actions Workflow (HTML artifact only)

Create/replace .github/workflows/render.yml with:

name: Render Quarto (HTML + artifacts)

on:
  push:
    branches: [ "main" ]   # change to [ "**" ] to run on all branches
  workflow_dispatch:

jobs:
  render:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: quarto-dev/quarto-actions/setup@v2     # use v2 (v3 tag not available)
      - name: Render HTML (no code execution)
        run: |
          set -euo pipefail
          quarto --version
          quarto render "quarto_llm_cheatsheet.qmd" --to html --no-execute
          test -f "quarto_llm_cheatsheet.html"
      - name: Upload HTML artifact
        uses: actions/upload-artifact@v4
        with:
          name: html
          if-no-files-found: error
          path: quarto_llm_cheatsheet.html

Commit/push:

git add .github/workflows/render.yml
git commit -m "ci: minimal Quarto render workflow"
git push

Trigger:

git commit --allow-empty -m "ci: trigger render"
git push

If you had older failing workflows, disable them by renaming to *.disabled (e.g., mv .github/workflows/quarto.yml .github/workflows/quarto.yml.disabled).

⸻

4) Inspect CI Runs and Download Artifacts (with gh)

If gh cache is blocked by a root-owned ~/.cache:
	•	Quick bypass for this shell session:

export XDG_CACHE_HOME="$HOME/.gh-cache"
mkdir -p "$XDG_CACHE_HOME"

	•	Permanent fix (recommended):

sudo chown -R "$USER":"$(id -gn)" "$HOME/.cache"
sudo chmod -R u+rwX "$HOME/.cache"

Check latest run on current branch:

RID=$(gh run list --branch "$(git branch --show-current)" --limit 1 --json databaseId -q '.[0].databaseId')
gh run view "$RID" --json name,displayTitle,headBranch,status,conclusion,event,workflowName -q \
  '"workflow=\(.workflowName)  title=\(.displayTitle)\nbranch=\(.headBranch)  status=\(.status)  conclusion=\(.conclusion)  event=\(.event)"'

Tail the job log:

gh run view "$RID" --log | tail -n 200

Download the html artifact and open the page:

mkdir -p artifacts
gh run download "$RID" -n html -D artifacts
find artifacts -maxdepth 2 -type f -name 'quarto_llm_cheatsheet.html' -print
open "$(find artifacts -maxdepth 2 -type f -name 'quarto_llm_cheatsheet.html' -print -quit)"   # macOS


⸻

5) Common CI Failures → Fast Fix
	•	“Unable to resolve action quarto-dev/quarto-actions@v3”
Use quarto-dev/quarto-actions/setup@v2 in your workflow.
	•	YAML parse error (“Invalid workflow file … line 15”)
Use spaces (no tabs), plain ASCII quotes. Replace with the minimal workflow above.
	•	HTML not found at upload step
Ensure the path matches the output (quarto_llm_cheatsheet.html). If you render to a different path (e.g., _site/ or docs/), adjust the path: accordingly.
	•	Include directive failed
Comment shortcodes as shown above.
	•	Cross-ref unresolved
Add a placeholder figure/table with the correct {#id} or render without --no-execute.
	•	PDF failures
Skip PDF on CI until needed (no TinyTeX). Later, flip to:

- uses: quarto-dev/quarto-actions/setup@v2
  with:
    tinytex: true

…and add a separate Render PDF step. Keep it continue-on-error: true if desired.

⸻

6) Optional: Run on All Branches

Switch:

sed -i '' 's/branches: \[ "main" \]/branches: [ "**" ]/' .github/workflows/render.yml   # macOS
git commit -am "ci: run on all branches"
git push


⸻

7) Optional: Publish as a Public Page (GitHub Pages)

If you want a URL (not just artifacts):

A) Commit a minimal Quarto website config and render to docs/:

cat > _quarto.yml <<'YAML'
project:
  type: website
  output-dir: docs
format:
  html:
    embed-resources: true
YAML

quarto render quarto_llm_cheatsheet.qmd --to html
git add _quarto.yml docs/
git commit -m "site: render to docs for GitHub Pages"
git push

B) Enable Pages: Repo → Settings → Pages → Source: Deploy from a branch, Branch: main / /docs.

(If you prefer CI to publish to gh-pages instead, keep content out of main—different workflow.)

⸻

8) “Assume-Competence” LLM Kickoff Prompt

Use this at the start of any LLM session to keep it on rails:

You are my Cautious Technician — Assume Competence.

Rules:
1) STATE CHECK first (read-only), then stop.
2) SCRIPT-FIRST: when action is needed, output a single idempotent script or exact commands; no here-docs; no GUI steps.
3) Assume I’m capable; prefer the most reliable solution, not the simplest.
4) One action at a time; branch on reality from my output.
5) Keep an Assumption Register and call out unknowns BEFORE proposing actions.


⸻

9) Handy Commands (Copy/Paste)

Local

quarto render quarto_llm_cheatsheet.qmd --to html --no-execute

Fix shortcodes in-place (quick regex)

perl -0777 -pe 's/\{\{<\s*(.*?)\s*>\}\}/{{<\/* $1 *\/>}}/g' -i quarto_llm_cheatsheet.qmd

Add placeholder figure (manual approach)

mkdir -p images
# create images/placeholder.svg (see above)
# then insert the Markdown figure line into the QMD

CI: minimal render workflow
(See the YAML in §3; paste into .github/workflows/render.yml)

Actions: latest run, logs, artifacts

export XDG_CACHE_HOME="$HOME/.gh-cache"   # if needed
RID=$(gh run list --branch "$(git branch --show-current)" --limit 1 --json databaseId -q '.[0].databaseId')
gh run view "$RID" --log | tail -n 200
mkdir -p artifacts
gh run download "$RID" -n html -D artifacts
open "$(find artifacts -maxdepth 2 -type f -name 'quarto_llm_cheatsheet.html' -print -quit)"

Fix cache perms permanently

sudo chown -R "$USER":"$(id -gn)" "$HOME/.cache"
sudo chmod -R u+rwX "$HOME/.cache"


⸻

That’s it

With this, you can:
	•	keep Quarto rendering locally,
	•	run a clean CI that uploads your HTML,
	•	diagnose failures in seconds,
	•	and steer any LLM to verify-first, script-first help.