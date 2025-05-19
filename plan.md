Below is a concise **TODO plan** distilled from the analysis you provided.  It walks through each step—locating the placeholder, creating the update‐script, and wiring it into your workflow—along with pointers to the relevant files/blocks in your repo.

---

## 📋 TODO Plan

| #  | Task                                                                                         | Location / File                             |
|:--:|:------------------------------------------------------------------------------------------------|:---------------------------------------------|
| 1  | **Verify (and add) the placeholder block** in `README.md`                                       | `README.md`                                  |
| 2  | **Create the update script** (`update-readme-timestamp.sh`) under `scripts/`                     | `scripts/update-readme-timestamp.sh`         |
| 3  | **Make the script executable**                                                                  | `chmod +x scripts/update-readme-timestamp.sh`|
| 4  | **(Optional) Test the script locally**, ensure it injects the current date/time correctly      | N/A                                           |
| 5  | **Add a GitHub Actions workflow** to auto-run the script on a schedule or on pushes to `main`  | `.github/workflows/update-readme.yml`        |
| 6  | **Commit and push** these changes                                                               | —                                             |

---

### 1. Verify (and add) the placeholder block in `README.md`

Ensure your README contains exactly the following marker lines where the timestamp should go:

```markdown
<!-- TIMESTAMP:START -->
<!-- TIMESTAMP:END -->
```
【F:README.md†L6-L7】

---

### 2. Create the update script

Create a new script at `scripts/update-readme-timestamp.sh` with the contents below.  This script finds the placeholder block and replaces it with the current date/time:

```bash
#!/usr/bin/env bash
set -euo pipefail

README="README.md"
START="<!-- TIMESTAMP:START -->"
END="<!-- TIMESTAMP:END -->"
NOW=$(date '+%Y-%m-%d %H:%M:%S %z')

# Rewrite README.md in-place, inserting the timestamp
awk -v start="$START" -v end="$END" -v now="$NOW" '
  $0 == start { print; print ""; print "更新日時: " now; print ""; in_block=1; next }
  $0 == end   { in_block=0 }
  !in_block   { print }
' "$README" > "$README.tmp" && mv "$README.tmp" "$README"
```
【F:scripts/update-readme-timestamp.sh†new】

---

### 3. Make the script executable

```bash
chmod +x scripts/update-readme-timestamp.sh
```

---

### 4. (Optional) Test the script locally

Run it and verify that `README.md` is updated in the placeholder block:

```bash
./scripts/update-readme-timestamp.sh
```

---

### 5. Add a GitHub Actions workflow

To automate this on every push to `main` (or on a cron schedule), add the following file:

```yaml
# .github/workflows/update-readme.yml
name: Update README Timestamp

on:
  schedule:
    - cron: '0 * * * *'      # every hour
  push:
    branches:
      - main

jobs:
  update-timestamp:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update README timestamp
        run: scripts/update-readme-timestamp.sh
      - name: Commit changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git commit -am "chore: update README timestamp" || echo "No changes to commit"
      - uses: ad-m/github-push-action@v0.6.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```
【F:.github/workflows/update-readme.yml†new】

---

### 6. Commit and push your changes

```bash
git add README.md scripts/update-readme-timestamp.sh .github/workflows/update-readme.yml
git commit -m "chore: add script and workflow to auto-update README timestamp"
git push
```

---

**That’s it!**  Once merged, your README will carry a live “更新日時” entry that’s kept up to date automatically. Let me know if you need anything else.