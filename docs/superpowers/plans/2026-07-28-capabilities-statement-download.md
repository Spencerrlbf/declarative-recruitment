# Capabilities Statement Download Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the supplied capabilities-statement design the PDF downloaded by both existing website links without changing the public URL.

**Architecture:** This is an isolated static-asset replacement. The supplied one-page PDF is copied byte-for-byte onto the tracked public PDF filename; the HTML remains unchanged, and verification covers binary identity, page recognition, rendered appearance, and both link targets.

**Tech Stack:** Static HTML, PDF asset, POSIX shell utilities, macOS Quick Look rendering

## Global Constraints

- Preserve the public filename `Declarative-Recruitment-Capabilities-Statement.pdf`.
- Keep both existing download links in `index.html` unchanged.
- Keep `Capabilities Statement Update-design.pdf` unchanged as the source artifact.
- Do not modify unrelated working-tree files.
- Require the source and public target to have identical SHA-256 hashes after replacement.
- Require the final public target to be recognized and visually verified as a one-page PDF.

---

## File Structure

- Modify: `Declarative-Recruitment-Capabilities-Statement.pdf` - public asset served by both website download links.
- Read only: `Capabilities Statement Update-design.pdf` - approved replacement source.
- Read only: `index.html` - existing download-link targets.
- Create temporarily: `tmp/pdfs/final/Declarative-Recruitment-Capabilities-Statement.pdf.png` - rendered verification image; it remains untracked.

### Task 1: Replace and Verify the Public Capabilities Statement

**Files:**
- Modify: `Declarative-Recruitment-Capabilities-Statement.pdf`
- Read: `Capabilities Statement Update-design.pdf`
- Verify: `index.html:1475`
- Verify: `index.html:1644`

**Interfaces:**
- Consumes: the approved bytes in `Capabilities Statement Update-design.pdf` and the two existing HTML anchors targeting the stable public filename.
- Produces: a tracked `Declarative-Recruitment-Capabilities-Statement.pdf` that is byte-for-byte identical to the approved source and remains reachable from both download buttons.

- [ ] **Step 1: Run preflight checks**

```bash
test -f 'Capabilities Statement Update-design.pdf'
test -f 'Declarative-Recruitment-Capabilities-Statement.pdf'
file 'Capabilities Statement Update-design.pdf' 'Declarative-Recruitment-Capabilities-Statement.pdf'
test "$(rg -c 'href=\"Declarative-Recruitment-Capabilities-Statement\.pdf\"[^>]*download' index.html)" -eq 2
if cmp -s 'Capabilities Statement Update-design.pdf' 'Declarative-Recruitment-Capabilities-Statement.pdf'; then
  echo 'Preflight failed: the public PDF already matches the source.' >&2
  exit 1
else
  echo 'Preflight passed: the public PDF needs replacement.'
fi
```

Expected: both files are recognized as one-page PDFs, exactly two download anchors target the stable filename, and the preflight reports that the public PDF needs replacement.

- [ ] **Step 2: Replace the public asset**

```bash
cp 'Capabilities Statement Update-design.pdf' 'Declarative-Recruitment-Capabilities-Statement.pdf'
```

Expected: the tracked public filename now contains the approved replacement bytes; the source file remains unchanged.

- [ ] **Step 3: Verify binary identity and PDF recognition**

```bash
cmp -s 'Capabilities Statement Update-design.pdf' 'Declarative-Recruitment-Capabilities-Statement.pdf'
shasum -a 256 'Capabilities Statement Update-design.pdf' 'Declarative-Recruitment-Capabilities-Statement.pdf'
file 'Declarative-Recruitment-Capabilities-Statement.pdf'
```

Expected: `cmp` exits successfully, both SHA-256 lines show the same digest, and `file` reports a one-page PDF.

- [ ] **Step 4: Render and inspect the final public PDF**

```bash
mkdir -p tmp/pdfs/final
qlmanage -t -s 2200 -o tmp/pdfs/final 'Declarative-Recruitment-Capabilities-Statement.pdf'
```

Inspect `tmp/pdfs/final/Declarative-Recruitment-Capabilities-Statement.pdf.png` at original detail.

Expected: one complete portrait page with a white background; all headings, company data, differentiators, past performance, contact details, and footer content are legible, with no clipping, overlap, missing text, or rendering artifacts.

- [ ] **Step 5: Verify the website references and implementation scope**

```bash
test "$(rg -c 'href=\"Declarative-Recruitment-Capabilities-Statement\.pdf\"[^>]*download' index.html)" -eq 2
rg -n 'href=\"Declarative-Recruitment-Capabilities-Statement\.pdf\"[^>]*download' index.html
git diff --check
git diff --name-only HEAD
```

Expected: the two anchors remain at their existing locations, `git diff --check` reports no errors, and the only implementation file listed by `git diff --name-only HEAD` is `Declarative-Recruitment-Capabilities-Statement.pdf`.

- [ ] **Step 6: Commit the public asset**

```bash
git add 'Declarative-Recruitment-Capabilities-Statement.pdf'
git commit -m 'Update downloadable capabilities statement'
```

Expected: the commit contains only the public PDF asset. The supplied source PDF and all pre-existing unrelated files remain untracked and unchanged.
