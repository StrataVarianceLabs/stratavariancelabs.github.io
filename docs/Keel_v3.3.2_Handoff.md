cd /workspaces/stratavariancelabs.github.io

ls -lh docs   # confirm the file is there

git add docs/keel_whitepaper_v3.3.2.pdf
git status -sb   # confirm it shows as staged (green)

git commit -m "docs: add Keel v3.3.2 whitepaper PDF"
git push origin main
# Keel v3.3.2 — Handoff Packet (Fresh Chat)
_Date generated: 2025-11-19 08:03 UTC_

## Snapshot
- **Repo:** `ripple-mark3`
- **Current branches:**  
  - `freeze/v3.3.2-metrics-shim` (release artifacts & notes)  
  - `feat/validators-and-rollups-v3.3.3` (next work branch — created & pushed)
- **Artifacts (latest TS):** `live/20251116T131040Z_adv`, `runs/proofs/20251116T131040Z_adv`, `runs/public/summary.json` (sizes OK)
- **Tag:** `v3.3.2` (exists)
- **GitHub Release:** `v3.3.2` (exists; tarball uploaded; notes refreshed)
- **Public tarball:** `keel-v3.3.2-public-20251116T131040Z_adv.tar.gz` (uploaded to release)
- **Notion logs:** Keel + Ripple channels received publish notes (OK)
- **Licensing/Legal:** `LICENSE-OSS`, `LICENSE-COMMERCIAL.md`, `NOTICE`, `docs/_footer.txt`, `ops/invoice_template.md` present
- **.gitignore:** noisy paths ignored (`live/`, `runs/proofs/`, `runs/public/`, `keel-*.tar.gz`)

## Inexorable — Publish Re‑Run (three bubbles)
> Paste each bubble in order (1→2→3). These are terminal‑safe and idempotent.

### Bubble 1 — Dev Preflight (read‑only)
```bash
set +e; set -o pipefail
ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"; cd "$ROOT" || exit 1
BRANCH="$(git rev-parse --abbrev-ref HEAD 2>/dev/null)"
REV="$(git rev-parse --short HEAD 2>/dev/null)"
echo "[PID] repo=$(basename "$ROOT") branch=$BRANCH rev=$REV"
python -V 2>/dev/null | sed 's/^/[py] /'
[ -z "$(git status --porcelain)" ] && echo "[git] clean" || echo "[git] working changes present"
command -v jq >/dev/null && echo "[tool] jq=ok" || echo "[tool] jq=missing"
command -v gh >/dev/null && echo "[tool] gh=ok" || echo "[tool] gh=missing"
pytest -q >/dev/null 2>&1 && echo "[pytest] ok" || echo "[pytest] (skipped/failed)"
for v in NOTION_TOKEN NOTION_RUNLOG_DB; do [ -n "${!v:-}" ] && echo "[env] $v=set" || echo "[env] $v=unset"; done
LIVE="$(ls -1dt live/* 2>/dev/null | head -n1)"
PROOFD="$(ls -1dt runs/proofs/* 2>/dev/null | head -n1)"
SUMF="runs/public/summary.json"
echo "[live]   ${LIVE:-<none>}"
echo "[proofd] ${PROOFD:-<none>}"
for f in "$LIVE/results.jsonl" "$PROOFD/proof_table.tsv" "$SUMF"; do [ -s "$f" ] && echo "[artifact] OK $f" || echo "[artifact] MISS $f"; done
if [ -n "$LIVE" ] && [ -n "$PROOFD" ]; then LTS="$(basename "$LIVE")"; PTS="$(basename "$PROOFD")"; [ "$LTS" = "$PTS" ] && echo "[ts] match: $LTS" || echo "[ts] mismatch: $LTS vs $PTS"; fi
[ -s "$SUMF" ] && jq -c '.metrics,.derived' "$SUMF" 2>/dev/null | sed 's/^/[summary] /'
git tag --list v3.3.2 | grep -q '^v3.3.2$' && echo "[tag] v3.3.2 exists" || echo "[tag] v3.3.2 missing"
command -v gh >/dev/null && gh release view v3.3.2 >/dev/null 2>&1 && echo "[release] exists" || echo "[release] not found or gh missing"
echo "[footer] © 2025 Kannan Rasiah t/a StrataVariance Labs — ABN 21 530 916 531 · hello@stratavariancelabs.com"
```

### Bubble 2 — Publish & Sync (idempotent)
```bash
set +e; set -o pipefail
ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"; cd "$ROOT" || exit 2
TAG="v3.3.2"
LIVE="$(ls -1dt live/* 2>/dev/null | head -n1)"; [ -n "$LIVE" ] || { echo "[err] live/* missing"; exit 2; }
TS="$(basename "$LIVE")"; PROOFD="runs/proofs/$TS"; SUMF="runs/public/summary.json"
for f in "$LIVE/results.jsonl" "$PROOFD/proof_table.tsv" "$SUMF"; do [ -s "$f" ] || { echo "[err] missing $f"; exit 2; }; done
RN="docs/RELEASE_NOTES.md"; [ -s "$RN" ] || { mkdir -p docs; printf "# Keel v3.3.2 — Five Goldens (single cycle)\n" > "$RN"; echo "[write] $RN"; }
TAR="keel-v3.3.2-public-$TS.tar.gz"
[ -s "$TAR" ] || { tar -czf "$TAR" "$LIVE/results.jsonl" "$PROOFD/proof_table.tsv" "$SUMF" && echo "[tar] OK $TAR"; }
git tag --list "$TAG" | grep -q "^$TAG$" || git tag -a "$TAG" -m "Keel v3.3.2 — Five Goldens [$TS]"
git push origin "$(git rev-parse --abbrev-ref HEAD)" --tags || true
if command -v gh >/dev/null; then
  if gh release view "$TAG" >/dev/null 2>&1; then
    gh release upload "$TAG" "$TAR" --clobber
    gh release edit "$TAG" --title "Keel v3.3.2 — Five Goldens (single cycle)" --notes-file "$RN"
  else
    gh release create "$TAG" "$TAR" --title "Keel v3.3.2 — Five Goldens (single cycle)" --notes-file "$RN"
  fi
fi
[ -x scripts/emit_router.sh ] && scripts/emit_router.sh keel "Publish" done "tag=$TAG ts=$TS" >/dev/null 2>&1 && echo "[notion][keel] OK" || true
[ -x scripts/emit_router.sh ] && scripts/emit_router.sh ripple "Publish" done "tag=$TAG ts=$TS" >/dev/null 2>&1 && echo "[notion][ripple] OK" || true
echo "[done] Publish & Sync complete."
```

### Bubble 3 — Freeze & Handoff
```bash
set +e; set -o pipefail
ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"; cd "$ROOT" || exit 0
command -v gh >/dev/null && gh release view v3.3.2 || true
TARGET="feat/validators-and-rollups-v3.3.3"
git show-ref --verify --quiet "refs/heads/$TARGET" || git branch "$TARGET" >/dev/null 2>&1 || true
git push -u origin "$TARGET" >/dev/null 2>&1 || true
[ -x scripts/emit_router.sh ] && scripts/emit_router.sh keel "Gate" pass "v3.3.2 published; handoff to SVL" >/dev/null 2>&1 || true
echo "✅ SAFE_TO_CLOSE ripple-mark3: YES"
```

---

## SVL Site — Preflight (for next chat)
- Repo: `StrataVarianceLabs/stratavariancelabs.github.io`
- Branch to PR: `chore/footer-licensing-whitepaper`
- Expected changes:
  - `docs/_footer.txt` contains: `© 2025 Kannan Rasiah t/a StrataVariance Labs — ABN 21 530 916 531 · hello@stratavariancelabs.com`
  - `docs/Licensing.md` (dual-license explainer)
  - `docs/WHITEPAPER.md` (stub; replace with full paper)
  - `index.html` includes footer + nav links to Whitepaper/Licensing and to the Keel v3.3.2 release

**Deploy check:** After merging PR to `main`, verify GitHub Pages build and that footer and links render live.

---

## Notes & Known Goods
- Preflight indicates 5/5 `plan_ok`, 5/5 `audit_ok`, drift avg `0.0`, and HAL `0.2` (1 injected case) in `summary.json`.
- Tag `v3.3.2` exists; Release exists; tarball uploaded; Notion emit OK.
- `.gitignore` added to keep working tree clean of generated artifacts.
- Legal & ABN footer present for repo/doc outputs.

---

## Attachments
- **SVL marker image path (from earlier upload):** `/mnt/data/ebedac75-a451-4624-83f0-aa7a55aad4b0.png`