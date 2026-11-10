---
name: "dsu-qp-scrutiny"
description: "Fix DSU question papers to DSU's Bloom's verb sheet and the review panel's precedents, then fill the SoE Scrutiny Form from the fixed paper. Outputs the fixed .docx and the filled form PDF."
---

# DSU question paper: Bloom's fixer + Scrutiny Form

Souvik's DSU paperwork pipeline, stages 1 and 2. Use it for any MSE / CAT / question paper that must follow
DSU's approved Bloom's action verbs and go to the scrutiny committee with the SoE scrutiny form.

**Outputs:** (1) the fixed question paper (.docx), (2) the Scrutiny Form filled from that fixed paper (.pdf),
plus a short change log and the list of things the setter must confirm.

Sources of truth, in order:
1. **Panel precedents** (`PANEL` in bloom_check.py): what DSU's review panel actually decided. They win.
2. **DSU's sheet** "REVISED Bloom's Taxonomy Action Verbs" (after Anderson & Krathwohl, 2001), exactly in `DSU`.
Levels and form numbers come from the scripts, never from guessing. Rewording is your judgment, under the rules.

## Inputs
- The paper as `.docx` (PDF/text: audit only; ask for the .docx before fixing). Work on copies, never the original.
- Each question's intended Bloom level (BL / BT Level / RBT / K-level column, or tags like `[L2]`).
- The blank **Scrutiny_Form_QP_SoE_DSU** PDF (one page, flat, no form fields). Ask for it if not attached.
- Academic year (not printed on papers), and the setter's name and designation. Default setter:
  "Prof. Souvik Ghosh, Assistant Professor, Department of CSE (AI&ML)".
- Optional: panel review notes; a level-spread rule (never invent one).

## Stage 1: fix the paper
Write `bloom_check.py` (below; needs `python-docx`).
1. Audit: `python bloom_check.py audit paper.docx --json audit.json [--level-rule "L1L2<=40"] [--strict]`
   Status: **OK** fits (sheet or panel precedent, note cites which) · **FIX** verb not on DSU's sheet at all
   (Verify, Check, Describe, Calculate...), always fix · **REVIEW** verb on the sheet but at another level and the
   panel has not ruled; show the setter, change only if asked (`--strict` makes these FIX) · **CHECK** no verb/level.
   Verbs count only at the start of a clause, so nouns ("the function of") and set-up clauses
   ("Suppose we have...", "where |psi> = ...") are ignored.
2. Decide each FIX (and REVIEW the setter wants fixed):
   - Keep the setter's intended level; if the honest demand differs, offer a retag instead of a verb change.
   - Take suggestions in order: panel preferences first (Verify -> Inspect; L3 -> "Make use of"/"Solve"), then sheet.
   - Minimal edits: the verb or the few words around it, still reading naturally. Content, numbers, symbols,
     marks, CO, PO unchanged. Multi-part questions take the level of their most demanding part.
   - Sheet surprises: "Discuss" L6 only; "Evaluate" L5 only; "Why/What/How" L1; "Explain" L2/L5;
     "Inspect" L4 (panel also accepts L2); "Find" L1 (panel also accepts L2).
3. `fixes.json` keyed by audit row ids (check each `old` exists in its cell), then apply:
   `{"t0r1": {"replace": [["Determine whether the set T", "Make use of the definition of linear independence to identify whether the set T"]]}, "t0r5": {"bl": "L2"}}`
   `python bloom_check.py apply paper.docx fixes.json -o paper_FIXED.docx`
   Only matched words change; formatting, equations, images kept. "fix by hand" in the log: tell the setter.
4. Re-audit until nothing is FIX. Confirm only Question/BL cells changed (compare all paragraph and cell text).

## Stage 2: fill the Scrutiny Form from the FIXED paper
Write `scrutiny_form.py` (below; needs `pdfplumber`, `reportlab`, `pypdf`; same folder as bloom_check.py).
1. Draft: `python scrutiny_form.py draft paper_FIXED.docx -o form_data.json --setter "<name, designation>"`
   It reads the paper's header and table and fills: department, MSE/CAT/QP box (CAT number in the box),
   course name + code, semester, Bloom's % of marks per level (L1-L6, from the fixed paper), and the Yes/No
   checks the paper proves: CO mapping (every question has a CO), Bloom's (no FIX/CHECK left), marks and
   assessment plan (question totals vs "Each question carries N marks" and "Max Marks"). A "No" carries the
   reason as the suggestion.
2. Resolve `_confirm` with the setter, then edit form_data.json:
   - `academic_year` (e.g. "2026-27"): not on the paper.
   - `syllabus` and `format` default to "Yes"; the setter must confirm (portion covered; assessment manual).
   - `difficulty` is an estimate from the Bloom mix (Low if L1+L2 >= 60%, High if L4-L6 >= 40%,
     High & Moderate if L4-L6 >= 25%, else Moderate); say it is an estimate.
3. Fill: `python scrutiny_form.py fill Scrutiny_Form.pdf form_data.json -o Scrutiny_Form_FILLED.pdf`
   Values are written over DSU's flat PDF at the positions of its own labels (found on the page each time);
   ticks are drawn strokes, so they print the same everywhere. If the form layout changed, the script stops
   with a message instead of misplacing text.
4. Render the filled page to PNG and look at it: nothing overlapping, every value in its box.

**Never fill on the setter's behalf:** the committee's final decision (only if the setter states the committee's
actual decision), committee names (only if given), and **any signature or date of signing**. These stay blank.
Flag mismatches you notice, e.g. a course code on the paper that differs from the one you know for the course.

## Deliver
The fixed .docx, the filled form PDF, a change log (Q, level, before, after), the Bloom's spread, REVIEW items
left as chosen, the `_confirm` answers used, and any CO alignment worry (a CO worded at L1-L2 assessed at L3+).

## Learning from the panel
When the setter brings panel notes: apply exactly those changes to the ORIGINAL draft as its own file (the panel
outranks this skill), then fix any remaining FIX items and regenerate the form from that final paper. Record each
new decision in `PANEL` (`accept`, `prefer`, `source`), point out where the panel contradicts the sheet or itself,
and propose an updated version of this skill.

## Precedents so far
- QC mid-sem Set C panel (Nov 2026): Verify -> Inspect (Q3a, Q4b); "Find the eigenvalues..." retagged L2;
  "Inspect whether |psi> is a valid state" kept at L2; L3 questions to use "Make use of" / "Solve".
  The panel left "Check whether" (Q1c) and "verify" (Q2b); treated as oversights and fixed (Identify / show).

## DSU approved verbs (reference)
| Level | Verbs |
|---|---|
| L1 Remembering | Choose, Define, Find, How, Label, List, Match, Name, Omit, Recall, Relate, Select, Show, Spell, Tell, What, When, Where, Which, Who, Why |
| L2 Understanding | Classify, Compare, Contrast, Demonstrate, Explain, Extend, Illustrate, Infer, Interpret, Outline, Relate, Rephrase, Show, Summarize, Translate |
| L3 Applying | Apply, Build, Choose, Construct, Develop, Experiment with, Identify, Interview, Make use of, Model, Organize, Plan, Select, Solve, Utilize |
| L4 Analyzing | Analyze, Assume, Categorize, Classify, Compare, Conclusion, Contrast, Discover, Dissect, Distinguish, Divide, Examine, Function, Inference, Inspect, List, Motive, Relationships, Simplify, Survey, Take part in, Test for, Theme |
| L5 Evaluating | Agree, Appraise, Assess, Award, Choose, Compare, Conclude, Criteria, Criticize, Decide, Deduct, Defend, Determine, Disprove, Estimate, Evaluate, Explain, Importance, Influence, Interpret, Judge, Justify, Mark, Measure, Opinion, Perceive, Prioritize, Prove, Rate, Recommend, Rule on, Select, Support, Value |
| L6 Creating | Adapt, Build, Change, Choose, Combine, Compile, Compose, Construct, Create, Delete, Design, Develop, Discuss, Elaborate, Estimate, Formulate, Happen, Imagine, Improve, Invent, Make up, Maximize, Minimize, Modify, Original, Originate, Plan, Predict, Propose, Solution, Solve, Suppose, Test, Theory |

## bloom_check.py
```python
#!/usr/bin/env python3
"""DSU Bloom's taxonomy checker for question papers.

  python bloom_check.py audit  paper.docx|paper.txt  [--json out.json] [--level-rule L1L2<=40] [--strict]
  python bloom_check.py apply  paper.docx fixes.json  -o fixed.docx

audit: finds each question's action verb, maps it to DSU's approved Bloom levels (L1-L6),
       compares with the paper's own BL tag, and reports the level spread.
apply: writes approved fixes back into the .docx surgically (only the changed words),
       so equations, images and formatting in the question are left untouched.

The verb lists are DSU's sheet (Revised Bloom's Taxonomy Action Verbs, after
Anderson & Krathwohl 2001). On top sits PANEL: decisions DSU's review panel actually made,
which override the literal sheet. Everything is plain code: no AI decides a level here.

Status per question:
  OK      verb fits the tagged level (on the sheet, or accepted by a panel precedent)
  FIX     a verb that is not on DSU's sheet at all: always fix
  REVIEW  verb is on the sheet but at another level, and the panel has not ruled on it:
          shown to the setter, changed only if they ask (or with --strict, treated as FIX)
  CHECK   no action verb or no level tag found: needs a human look
"""
import json, re, sys, argparse
from collections import Counter, defaultdict

# ── DSU approved verbs, exactly as on the sheet ────────────────────────────
DSU = {
    1: "Choose, Define, Find, How, Label, List, Match, Name, Omit, Recall, Relate, Select, Show, Spell, "
       "Tell, What, When, Where, Which, Who, Why",
    2: "Classify, Compare, Contrast, Demonstrate, Explain, Extend, Illustrate, Infer, Interpret, Outline, "
       "Relate, Rephrase, Show, Summarize, Translate",
    3: "Apply, Build, Choose, Construct, Develop, Experiment with, Identify, Interview, Make use of, Model, "
       "Organize, Plan, Select, Solve, Utilize",
    4: "Analyze, Assume, Categorize, Classify, Compare, Conclusion, Contrast, Discover, Dissect, Distinguish, "
       "Divide, Examine, Function, Inference, Inspect, List, Motive, Relationships, Simplify, Survey, "
       "Take part in, Test for, Theme",
    5: "Agree, Appraise, Assess, Award, Choose, Compare, Conclude, Criteria, Criticize, Decide, Deduct, Defend, "
       "Determine, Disprove, Estimate, Evaluate, Explain, Importance, Influence, Interpret, Judge, Justify, Mark, "
       "Measure, Opinion, Perceive, Prioritize, Prove, Rate, Recommend, Rule on, Select, Support, Value",
    6: "Adapt, Build, Change, Choose, Combine, Compile, Compose, Construct, Create, Delete, Design, Develop, "
       "Discuss, Elaborate, Estimate, Formulate, Happen, Imagine, Improve, Invent, Make up, Maximize, Minimize, "
       "Modify, Original, Originate, Plan, Predict, Propose, Solution, Solve, Suppose, Test, Theory",
}
LEVEL_NAME = {1: "Remembering", 2: "Understanding", 3: "Applying", 4: "Analyzing", 5: "Evaluating", 6: "Creating"}

# British spellings and inflections map onto the sheet's entries.
VARIANTS = {"analyse": "analyze", "summarise": "summarize", "organise": "organize", "categorise": "categorize",
            "criticise": "criticize", "prioritise": "prioritize", "utilise": "utilize", "maximise": "maximize",
            "minimise": "minimize"}

# Within DSU's list, the verbs examiners actually use, most natural first (for suggestions only).
PREFERRED = {1: ["Define", "List", "Name", "Recall", "Label", "Match"],
             2: ["Explain", "Illustrate", "Summarize", "Outline", "Interpret", "Classify"],
             3: ["Make use of", "Solve", "Apply", "Construct", "Develop", "Identify"],
             4: ["Analyze", "Examine", "Distinguish", "Categorize", "Inspect", "Simplify"],
             5: ["Justify", "Evaluate", "Assess", "Defend", "Criticize", "Prove"],
             6: ["Design", "Formulate", "Propose", "Create", "Develop", "Modify"]}

# ── Panel precedents: what DSU's review panel actually decided. Add a line after every review. ──
PANEL = {
    # verb: levels the panel accepted that the sheet does not list for it
    "accept": {"find": {2}, "inspect": {2}},
    # unapproved or rejected verb: the panel's replacement, tried first
    "prefer": {"verify": ["Inspect"]},
    "source": {"find": "QC MSE Set C panel, Nov 2026: Q2(a) 'Find the eigenvalues...' retagged L2",
               "inspect": "QC MSE Set C panel, Nov 2026: Q3(a) 'Inspect whether |psi> is valid' kept at L2",
               "verify": "QC MSE Set C panel, Nov 2026: 'Verify' replaced by 'Inspect' (Q3a, Q4b)",
               "L3": "QC MSE Set C panel, Nov 2026: L3 questions to use 'Make use of' / 'Solve'"},
}

APPROVED = defaultdict(set)              # verb (lower) -> {levels}
for lvl, words in DSU.items():
    for w in words.split(","):
        APPROVED[w.strip().lower()].add(lvl)
def allowed(verb, lvl):
    """On DSU's sheet at this level, or accepted there by a panel precedent."""
    v = verb.lower()
    return lvl in APPROVED[v] or lvl in PANEL["accept"].get(v, set())


for lvl, words in PREFERRED.items():     # every suggestion must itself be allowed at that level
    assert all(allowed(w, lvl) for w in words), (lvl, words)

# Common exam verbs that are NOT on DSU's sheet, with approved replacements by level.
# These suggestions are ours, not DSU's: the person setting the paper decides.
NOT_APPROVED = {
    "describe":     {1: ["Define", "Tell"], 2: ["Explain", "Outline", "Illustrate"], 4: ["Examine"]},
    "differentiate": {2: ["Contrast", "Compare"], 4: ["Distinguish", "Contrast"]},
    "derive":       {3: ["Develop", "Construct"], 5: ["Prove", "Deduct"]},
    "calculate":    {3: ["Solve", "Apply"], 5: ["Determine", "Estimate"]},
    "compute":      {3: ["Solve", "Apply"], 5: ["Determine", "Estimate"]},
    "write":        {1: ["List", "Define"], 3: ["Construct", "Develop"], 6: ["Compose", "Formulate"]},
    "draw":         {2: ["Illustrate"], 3: ["Construct"], 6: ["Design"]},
    "sketch":       {2: ["Illustrate"], 3: ["Construct"]},
    "state":        {1: ["Define", "Recall", "Name"]},
    "enumerate":    {1: ["List"]},
    "mention":      {1: ["List", "Name"]},
    "give":         {1: ["List", "Name"], 2: ["Illustrate"]},
    "implement":    {3: ["Apply", "Build", "Develop"], 6: ["Design"]},
    "normalize":    {3: ["Apply"], 4: ["Simplify"]},
    "convert":      {2: ["Translate"], 3: ["Apply"]},
    "prepare":      {3: ["Construct", "Develop"]},
    "elucidate":    {2: ["Explain"]},
    "highlight":    {2: ["Outline"]},
    "verify":       {2: ["Inspect", "Show"], 4: ["Inspect"], 5: ["Prove", "Justify"]},
    "comment":      {5: ["Justify", "Criticize"]},
    "trace":        {3: ["Apply"], 4: ["Examine"]},
    "simulate":     {3: ["Model", "Apply"]},
    "check":        {3: ["Identify", "Apply"], 4: ["Examine", "Test for", "Inspect"], 5: ["Determine", "Justify"]},
}

# Words on DSU's sheet that also open a set-up clause ("Suppose we have...", "where |psi> = ...").
# Followed by these, they describe the scene rather than ask for something, so they are not the task verb.
SETUP_WORDS = {"suppose", "assume", "where", "when"}
SETUP_NEXT = {"we", "you", "a", "an", "the", "that", "it", "there", "this", "these", "those", "our", "i", "x", "y"}


def is_setup(key, after):
    if key not in SETUP_WORDS:
        return False
    nxt = re.match(r"\s*(\S+)", after)
    if not nxt:
        return False
    w = nxt.group(1)
    return w.lower().strip(",") in SETUP_NEXT or not w[0].isalpha() or (len(w) == 1 and w.isalpha())

for opts in NOT_APPROVED.values():
    for lvl, words in opts.items():
        assert all(allowed(w, lvl) for w in words), (lvl, words)

PHRASES = sorted(list(APPROVED) + list(NOT_APPROVED), key=lambda s: -len(s))   # longest first
# A verb counts only at the start of a clause: start of text, after . ? ! ; : , or after "and"/"then"/"also".
CLAUSE = r"(?:^|[.?!;:,]\s+|\b(?:and|then|also|hence|thus)\s+|\(\s*[a-z0-9ivx]+\s*\)\s*|^\s*[a-z0-9ivx]+[.)]\s*)"
LEVEL_RE = re.compile(r"\b(?:L|BL|K|RBT[- ]?L?|Level)\s*[-:]?\s*([1-6])\b|^\s*([1-6])\s*$", re.I)
NUM_PREFIX = re.compile(r"^\s*(?:Q\.?\s*)?(?:\d+|[a-z]|[ivx]+)\s*[.)]\s*", re.I)


def norm(word):
    w = word.lower()
    return VARIANTS.get(w, w)


def find_verbs(text):
    """Clause-initial action words in order: [(phrase_as_written, key, approved_levels_or_None)]."""
    t = NUM_PREFIX.sub("", text.strip())
    found = []
    for m in re.finditer(CLAUSE, t, flags=re.I | re.M):
        rest = t[m.end():]
        low = rest.lower()
        for p in PHRASES:
            if low.startswith(p) and (len(low) == len(p) or not low[len(p)].isalpha()):
                if not is_setup(p, rest[len(p):]):
                    found.append((rest[:len(p)], p, APPROVED.get(p)))
                break
        else:
            first = re.match(r"[A-Za-z]+", rest)
            if first and norm(first.group()) in APPROVED:
                k = norm(first.group())
                found.append((first.group(), k, APPROVED[k]))
    seen, out = set(), []
    for f in found:                       # de-duplicate by position-independent phrase order
        if (f[0], f[1]) not in seen:
            seen.add((f[0], f[1])); out.append(f)
    return out


def parse_level(cell):
    if cell is None:
        return None
    s = str(cell).strip()
    m = LEVEL_RE.search(s)
    if m:
        return int(m.group(1) or m.group(2))
    for lvl, name in LEVEL_NAME.items():
        if name[:5].lower() in s.lower():
            return lvl
    return None


def check(text, tagged, strict=False):
    verbs = find_verbs(text)
    approved = [v for v in verbs if v[2]]
    unapproved = [v for v in verbs if not v[2]]
    sheet = sorted(set().union(*[v[2] for v in approved])) if approved else []
    panel_ok = [v for v in approved if tagged and tagged in PANEL["accept"].get(v[1], set())]
    r = {"verbs": [v[0] for v in verbs], "possible_levels": sheet, "tagged": tagged, "issues": [], "suggest": [],
         "severity": None}
    if unapproved:
        r["severity"] = "FIX"
        for v in unapproved:
            opts = NOT_APPROVED[v[1]]
            pick = opts.get(tagged) or [w for ws in opts.values() for w in ws]
            pref = [w for w in PANEL["prefer"].get(v[1], []) if tagged is None or allowed(w, tagged)]
            pick = pref + [w for w in pick if w not in pref]
            r["issues"].append(f'"{v[0]}" is not on DSU\'s list' + (" (panel prefers " + "/".join(pref) + ")" if pref else ""))
            r["suggest"].append({"replace": v[0], "with": pick, "at_level": tagged})
    if not verbs:
        r["issues"].append("no action verb found at the start of a clause")
    if approved and tagged:
        if tagged not in sheet and not panel_ok:
            r["issues"].append(f"tagged L{tagged} but its verb(s) {', '.join(v[0] for v in approved)} are "
                               f"L{'/'.join(map(str, sheet))} on DSU's sheet")
            r["suggest"].append({"replace": approved[0][0], "with": PREFERRED[tagged], "at_level": tagged,
                                 "or_retag": [f"L{l}" for l in sheet]})
            r["severity"] = r["severity"] or ("FIX" if strict else "REVIEW")
        elif tagged not in sheet and panel_ok:
            v = panel_ok[0]
            r["note"] = f'"{v[0]}" is L{"/".join(map(str, sorted(v[2])))} on the sheet; panel accepted it at L{tagged} ' \
                        f'({PANEL["source"].get(v[1], "panel precedent")})'
        elif len(approved[0][2]) > 1:
            r["note"] = f'"{approved[0][0]}" sits at L{"/".join(map(str, sorted(approved[0][2])))}: OK for L{tagged} ' \
                        f"if the question really demands {LEVEL_NAME[tagged].lower()}"
    if not tagged:
        r["issues"].append("no Bloom level tagged")
    r["status"] = "OK" if not r["issues"] else (r["severity"] or "CHECK")
    return r


# ── Reading papers ─────────────────────────────────────────────────────────
HEAD = {"q": r"^questions?\b|question$",
        "bl": r"^(bl|bt|rbt|kl|cl|k)\b|^(k|bt|rbt)[- ]?level|bloom|cognitive|^level",
        "marks": r"marks?\b|^m$", "co": r"^cos?$|^co[ -]?(no|map)", "no": r"^(q\.? ?no|qno|no|sl\.? ?no|q)$"}


def header_map(cells):
    hm = {}
    for i, c in enumerate(cells):
        t = re.sub(r"\s+", " ", c.strip().lower()).rstrip(".").replace("’", "").replace("'", "")  # CO’s → cos
        for key, pat in HEAD.items():
            if key not in hm and re.search(pat, t):
                hm[key] = i
                break
    return hm if "q" in hm else None


def find_header(table):
    for ri, row in enumerate(table.rows[:5]):            # header may sit under a title row
        hm = header_map([c.text for c in row.cells])
        if hm:
            return ri, hm
    return None, None


def read_docx(path):
    import docx
    d = docx.Document(path)
    rows = []
    for ti, table in enumerate(d.tables):
        hr, hm = find_header(table)
        if hm is None:
            continue
        for ri, row in enumerate(table.rows):
            if ri <= hr:
                continue
            cells = [c.text for c in row.cells]
            q = cells[hm["q"]].strip()
            if len(q) < 8:
                continue
            rows.append({"id": f"t{ti}r{ri}", "no": cells[hm["no"]].strip() if "no" in hm else "",
                         "question": q, "bl": parse_level(cells[hm["bl"]]) if "bl" in hm else None,
                         "marks": to_num(cells[hm["marks"]]) if "marks" in hm else None,
                         "co": cells[hm["co"]].strip() if "co" in hm else ""})
    if not rows:                          # no question table: fall back to paragraphs
        for pi, p in enumerate(d.paragraphs):
            rows += para_row(p.text, f"p{pi}")
    return rows


def para_row(text, rid):
    t = text.strip()
    if len(t) < 12 or not re.search(r"[A-Za-z]{3}", t):
        return []
    lv = re.search(r"[\[(]\s*(?:L|BL|K|RBT)\s*-?\s*([1-6])\s*[\])]", t, re.I)
    mk = re.search(r"[\[(]\s*(\d+(?:\.\d+)?)\s*(?:M|marks?)\s*[\])]", t, re.I)
    co = re.search(r"\bCO\s*-?\s*(\d+)", t)
    q = re.sub(r"[\[(][^\])]*(?:L|BL|K|RBT|CO|M|marks?)\s*-?\s*\d*[^\])]*[\])]", "", t).strip()
    return [{"id": rid, "no": "", "question": q, "bl": int(lv.group(1)) if lv else None,
             "marks": float(mk.group(1)) if mk else None, "co": f"CO{co.group(1)}" if co else ""}]


def read_text(path):
    rows = []
    for i, line in enumerate(open(path, encoding="utf-8").read().splitlines()):
        rows += para_row(line, f"l{i}")
    return rows


def to_num(s):
    m = re.search(r"\d+(?:\.\d+)?", s or "")
    return float(m.group()) if m else None


# ── Report ─────────────────────────────────────────────────────────────────
def audit(path, rule=None, strict=False):
    rows = read_docx(path) if path.lower().endswith(".docx") else read_text(path)
    for r in rows:
        r.update(check(r["question"], r["bl"], strict))
    count, marks = Counter(), Counter()
    for r in rows:
        if r["bl"]:
            count[r["bl"]] += 1; marks[r["bl"]] += r["marks"] or 0
    total_m = sum(marks.values()) or 1
    spread = {f"L{l}": {"questions": count[l], "marks": marks[l], "pct_marks": round(100 * marks[l] / total_m, 1)}
              for l in range(1, 7)}
    rule_result = None
    if rule:                              # e.g. "L1L2<=40" or "L3L4L5L6>=50"
        m = re.fullmatch(r"((?:L[1-6])+)\s*(<=|>=)\s*(\d+)", rule.replace(" ", ""))
        lv = [int(x) for x in re.findall(r"L([1-6])", m.group(1))]
        pct = sum(spread[f"L{l}"]["pct_marks"] for l in lv)
        ok = pct <= int(m.group(3)) if m.group(2) == "<=" else pct >= int(m.group(3))
        rule_result = {"rule": rule, "actual_pct": round(pct, 1), "ok": ok}
    return {"file": path, "questions": rows, "spread": spread, "rule": rule_result,
            "summary": dict(Counter(r["status"] for r in rows))}


def markdown(rep):
    out = ["| Q | Question (start) | Tagged | Verb(s) | DSU levels | Status | Issue / suggestion |",
           "|---|---|---|---|---|---|---|"]
    for r in rep["questions"]:
        sug = "; ".join(f'{s["replace"]} → {" / ".join(s["with"][:4])}' +
                        (f' (or retag {"/".join(s["or_retag"])})' if s.get("or_retag") else "") for s in r["suggest"])
        issue = "; ".join(r["issues"]) or r.get("note", "")
        out.append(f'| {r["no"] or r["id"]} | {r["question"][:60].replace("|", "/")} | '
                   f'{"L%d" % r["bl"] if r["bl"] else "—"} | {", ".join(r["verbs"]) or "—"} | '
                   f'{"/".join("L%d" % l for l in r["possible_levels"]) or "—"} | {r["status"]} | '
                   f'{issue}{(" → " + sug) if sug else ""} |')
    out += ["", "| Level | Questions | Marks | % of marks |", "|---|---|---|---|"]
    out += [f'| {k} {LEVEL_NAME[int(k[1])]} | {v["questions"]} | {v["marks"]:g} | {v["pct_marks"]} |'
            for k, v in rep["spread"].items()]
    if rep["rule"]:
        rr = rep["rule"]
        out.append(f'\nRule {rr["rule"]}: actual {rr["actual_pct"]}% → {"PASS" if rr["ok"] else "FAIL"}')
    return "\n".join(out)


# ── Writing fixes back (surgical) ──────────────────────────────────────────
def apply_fixes(path, fixes, out):
    """fixes: {row_id: {"replace": [[old, new], ...], "bl": "L3"}}. Edits only the matched words."""
    import docx
    d = docx.Document(path)
    log = []
    for rid, fx in fixes.items():
        m = re.fullmatch(r"t(\d+)r(\d+)", rid)
        if not m:
            log.append(f"{rid}: only table rows can be fixed automatically; edit by hand"); continue
        row = d.tables[int(m.group(1))].rows[int(m.group(2))]
        hm = find_header(d.tables[int(m.group(1))])[1] or {}
        qcell = row.cells[hm.get("q", 0)]
        for old, new in fx.get("replace", []):
            log.append(f"{rid}: " + replace_in_cell(qcell, old, new))
        if fx.get("bl") and "bl" in hm:
            log.append(f"{rid}: BL → " + replace_in_cell(row.cells[hm["bl"]], row.cells[hm["bl"]].text.strip(), fx["bl"]))
    d.save(out)
    return log


def replace_in_cell(cell, old, new):
    for p in cell.paragraphs:
        for run in p.runs:                                  # best case: the words sit inside one run
            if old in run.text:
                run.text = run.text.replace(old, new, 1)
                return f'"{old}" → "{new}"'
        joined = "".join(r.text for r in p.runs)
        start = joined.find(old)
        if start >= 0:                                      # spans runs: edit only the runs it touches
            if p._p.xpath(".//m:oMath | .//w:drawing | .//w:object"):
                return f'"{old}" spans formatting next to an equation/image: fix by hand'
            end, pos, placed = start + len(old), 0, False
            for run in p.runs:
                a, b = pos, pos + len(run.text)
                pos = b
                if b <= start or a >= end:
                    continue                                # untouched run keeps its text and formatting
                keep_head = run.text[:max(0, start - a)]
                keep_tail = run.text[max(0, end - a):] if end <= b else ""
                run.text = keep_head + ("" if placed else new) + keep_tail
                placed = True
            return f'"{old}" → "{new}" (across runs; other formatting kept)'
    return f'"{old}" not found: no change'


def main():
    ap = argparse.ArgumentParser()
    sub = ap.add_subparsers(dest="cmd", required=True)
    a = sub.add_parser("audit"); a.add_argument("paper"); a.add_argument("--json"); a.add_argument("--level-rule")
    a.add_argument("--strict", action="store_true", help="treat sheet level mismatches as FIX, ignoring panel leniency")
    f = sub.add_parser("apply"); f.add_argument("paper"); f.add_argument("fixes"); f.add_argument("-o", required=True)
    args = ap.parse_args()
    if args.cmd == "audit":
        rep = audit(args.paper, args.level_rule, args.strict)
        if args.json:
            json.dump(rep, open(args.json, "w"), indent=2)
        print(markdown(rep))
        print("\nSummary:", rep["summary"])
    else:
        for line in apply_fixes(args.paper, json.load(open(args.fixes)), args.o):
            print(line)


if __name__ == "__main__":
    main()
```

## scrutiny_form.py
```python
#!/usr/bin/env python3
"""Fill DSU SoE's "Scrutiny Format for Formative Assessments" (MSE / CAT question paper) from a fixed QP.

  python scrutiny_form.py draft  paper_FIXED.docx  -o form_data.json  [--setter "Prof. X, Assistant Professor"]
  python scrutiny_form.py fill   Scrutiny_Form.pdf form_data.json     -o Scrutiny_Form_FILLED.pdf

draft: reads the fixed paper (header + question table, via bloom_check.audit) and writes form_data.json:
       course, semester, type, Bloom's % per level, and the Yes/No checks the paper itself proves.
       Anything the paper cannot prove is marked in "_confirm" for the setter to check.
fill:  writes the values onto DSU's flat PDF (no form fields) at the positions of its own labels, which are
       located on the page each time, so the form underneath is never altered. The committee's decision,
       committee names and all signatures are left blank unless the data file explicitly provides them.
"""
import argparse, json, re, sys, os
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
import bloom_check

YESNO_ROWS = ["syllabus", "plan", "co", "bloom", "marks", "format"]       # top-to-bottom order on the form
BOX_ORDER = ["question_paper", "mse", "cat", "high", "moderate", "high_moderate", "low",
             "accepted_without_corrections", "accepted_with_minor_corrections"]


# ── draft: derive the form data from the fixed paper ──────────────────────
def header_text(path):
    import docx
    return "\n".join(p.text for p in docx.Document(path).paragraphs)


def draft(paper, setter=None):
    rep = bloom_check.audit(paper)
    head = header_text(paper)
    rows = rep["questions"]
    data, confirm = {}, []

    m = re.search(r"Course Title:\s*(.+?)\s{2,}|Course Title:\s*(.+?)\t", head)
    title = (m.group(1) or m.group(2)).strip() if m else ""
    code = re.search(r"Course Code:\s*([A-Z0-9]+)", head)
    data["course"] = f"{title} ({code.group(1)})" if code else title
    sem = re.search(r"\b([IVX]+)\s+Semester", head)
    data["semester"] = sem.group(1) if sem else ""
    dept = re.search(r"Computer Science and Engineering\s*\(([^)]+)\)", head)
    data["department"] = f"CSE ({dept.group(1)})" if dept else ""
    data["paper_type"] = "mse" if re.search(r"Mid Semester", head, re.I) else \
        "cat" if re.search(r"\bCAT\b|Continuous Assessment", head, re.I) else "question_paper"
    cat = re.search(r"CAT\s*-?\s*([123])", head)
    data["cat_number"] = cat.group(1) if cat else ""
    data["academic_year"] = ""
    confirm.append("academic_year: not printed on the paper; fill it in (e.g. 2026-27)")

    # Bloom's % of marks, from the fixed paper
    data["btl_percent"] = {k: v["pct_marks"] for k, v in rep["spread"].items()}

    # Yes/No checks the paper proves
    checks = {}
    no_co = [r["no"] for r in rows if not r.get("co")]
    checks["co"] = ["No", "No CO for: " + ", ".join(no_co)] if no_co else ["Yes", ""]
    open_fix = [r["no"] for r in rows if r["status"] in ("FIX", "CHECK")]
    checks["bloom"] = ["No", "Fix verbs/levels in: " + ", ".join(open_fix)] if open_fix else ["Yes", ""]

    max_m = re.search(r"Max\.?\s*Marks\s*:\s*(\d+)", head, re.I)
    per_q = re.search(r"Each question carries\s*(\d+)\s*marks", head, re.I)
    total = sum(r["marks"] or 0 for r in rows)
    groups = {}
    for r in rows:
        g = re.match(r"\s*Q\.?\s*(\d+)", r["no"] or "")
        if g:
            groups[g.group(1)] = groups.get(g.group(1), 0) + (r["marks"] or 0)
    problems = []
    if max_m and total != float(max_m.group(1)):
        problems.append(f"marks add to {total:g}, paper says {max_m.group(1)}")
    if per_q:
        bad = [f"Q{q}={m:g}" for q, m in groups.items() if m != float(per_q.group(1))]
        if bad:
            problems.append(f"each question should carry {per_q.group(1)}: " + ", ".join(bad))
    checks["marks"] = ["No", "; ".join(problems)] if problems else ["Yes", ""]
    checks["plan"] = ["No", "; ".join(problems)] if problems else ["Yes", ""]
    confirm.append("plan: checked only marks/question totals against the paper's own instructions")
    checks["syllabus"] = ["Yes", ""]
    confirm.append("syllabus: assumed Yes (questions cover CO " +
                   ", ".join(sorted({r['co'] for r in rows if r.get('co')})) + "); confirm the portion covered")
    checks["format"] = ["Yes", ""]
    confirm.append("format: assumed Yes (DSU QP template); confirm against the assessment manual")
    data["checks"] = checks

    # Difficulty: an estimate from the Bloom mix, for the setter to confirm (the committee has the final say)
    s = rep["spread"]
    low = s["L1"]["pct_marks"] + s["L2"]["pct_marks"]
    high = s["L4"]["pct_marks"] + s["L5"]["pct_marks"] + s["L6"]["pct_marks"]
    data["difficulty"] = "low" if low >= 60 else "high" if high >= 40 else "high_moderate" if high >= 25 else "moderate"
    confirm.append(f"difficulty: '{data['difficulty']}' is an estimate from the Bloom mix "
                   f"(L1+L2 {low:g}%, L4-L6 {high:g}%); the committee may judge differently")

    data["setter"] = setter or ""
    if not setter:
        confirm.append("setter: name and designation not given")
    data["decision"] = None          # committee's call: left blank on purpose
    data["committee"] = []           # names only if the setter provides them; signatures are never filled
    data["_confirm"] = confirm
    data["_review_items"] = [f'{r["no"]}: {"; ".join(r["issues"])}' for r in rows if r["status"] == "REVIEW"]
    return data


# ── fill: write onto the flat PDF ─────────────────────────────────────────
def locate(pdf_path):
    """Find every anchor on the page from its own text, so small layout shifts don't break the fill."""
    import pdfplumber
    with pdfplumber.open(pdf_path) as pdf:
        p = pdf.pages[0]
        words = p.extract_words(use_text_flow=True)
        H = p.height
        boxes, seen = [], set()
        for c in p.curves + p.rects:
            w, h = c["x1"] - c["x0"], c["bottom"] - c["top"]
            key = (round(c["x0"]), round(c["top"]))
            if 8 < w < 35 and 8 < h < 30 and key not in seen:
                seen.add(key); boxes.append(c)
        boxes.sort(key=lambda b: (round(b["top"] / 5), b["x0"]))
        if len(boxes) != len(BOX_ORDER):
            sys.exit(f"Expected {len(BOX_ORDER)} tick boxes on the form, found {len(boxes)}: the form changed")
        tables = p.find_tables()
        cells = tables[0].rows if tables else []

        def find(text, nth=0):
            hits = [w for w in words if w["text"] == text]
            if len(hits) <= nth:
                sys.exit(f'Label "{text}" (#{nth + 1}) not found on the form')
            return hits[nth]

        yesno = sorted([w for w in words if w["text"] == "Yes/No:"], key=lambda w: w["top"])
        sugg = sorted([w for w in words if w["text"] == "suggestions:"], key=lambda w: w["top"])
        if len(yesno) != len(YESNO_ROWS) or len(sugg) != len(YESNO_ROWS):
            sys.exit("Yes/No rows on the form do not match the expected 6")
        btl_row = next(r for r in cells if any(c and c[0] < 50 and 400 < c[1] < 460 for c in r.cells))
        btl_cells = [c for c in btl_row.cells if c]
        course_row = next(r for r in cells if r.cells[0] and abs(r.cells[0][1] - (find("Course")["top"] - 10)) < 15)
        setter_lbl = min((w for w in words if w["text"] == "Name" and w["x0"] < 50), key=lambda w: w["top"])
        setter_row = next(r for r in cells if r.cells[0] and r.cells[0][1] <= setter_lbl["top"] <= r.cells[0][3])
        return {"H": H, "boxes": dict(zip(BOX_ORDER, boxes)), "yesno": dict(zip(YESNO_ROWS, yesno)),
                "sugg": dict(zip(YESNO_ROWS, sugg)), "btl": btl_cells,
                "course": [c for c in course_row.cells if c][1], "setter": [c for c in setter_row.cells if c][1],
                "dept": find("Department:"), "year": find("year:"), "sem": find("Semester:"),
                "dept_room": min(w["x0"] for w in words if abs(w["top"] - find("Department:")["top"]) < 3
                                 and w["x0"] > find("Department:")["x1"]) - find("Department:")["x1"] - 9}


def fill(form, data, out):
    from reportlab.pdfgen import canvas
    from reportlab.lib.utils import simpleSplit
    from pypdf import PdfReader, PdfWriter
    import io
    L = locate(form)
    H = L["H"]
    y = lambda top: H - top                                  # pdfplumber top → PDF y
    buf = io.BytesIO()
    c = canvas.Canvas(buf, pagesize=(PdfReader(form).pages[0].mediabox.width, H))
    c.setFillColorRGB(0, 0, 0)

    def text(x, baseline_top, s, size=10, font="Times-Roman"):
        c.setFont(font, size); c.drawString(x, y(baseline_top), s)

    def tick(name, char=None):
        b = L["boxes"][name]
        if char:
            c.setFont("Times-Bold", 10)
            c.drawCentredString((b["x0"] + b["x1"]) / 2, y(b["bottom"] - 3), char)
        else:                                                 # a drawn tick: same in every viewer and on paper
            x0, x1, t, bt = b["x0"], b["x1"], b["top"], b["bottom"]
            w, h = x1 - x0, bt - t
            c.setLineWidth(1.4); c.setLineCap(1); c.setLineJoin(1)
            p = c.beginPath()
            p.moveTo(x0 + 0.28 * w, y(t + 0.52 * h))
            p.lineTo(x0 + 0.45 * w, y(t + 0.78 * h))
            p.lineTo(x0 + 0.74 * w, y(t + 0.22 * h))
            c.drawPath(p, stroke=1, fill=0)

    dept, room = data.get("department", ""), L["dept_room"]          # shrink to the gap before "Question paper:"
    from reportlab.pdfbase.pdfmetrics import stringWidth
    size = 10
    while size > 6 and stringWidth(dept, "Times-Roman", size) > room:
        size -= 0.5
    text(L["dept"]["x1"] + 3, L["dept"]["bottom"] - 2, dept, size)
    pt = data.get("paper_type")
    if pt == "cat":
        tick("cat", data.get("cat_number") or None)
    elif pt in ("mse", "question_paper"):
        tick(pt)
    cx0, ct, cx1, cb = L["course"]
    text(cx0 + 5, (ct + cb) / 2 + 4, data.get("course", ""), 10.5, "Times-Bold")
    text(L["year"]["x1"] + 5, L["year"]["bottom"] - 2, data.get("academic_year", ""), 10.5)
    text(L["sem"]["x1"] + 5, L["sem"]["bottom"] - 2, data.get("semester", ""), 10.5)

    for key in YESNO_ROWS:
        ans, why = (data.get("checks", {}).get(key) or ["", ""])
        yn, sg = L["yesno"][key], L["sugg"][key]
        text(yn["x1"] + 5, yn["bottom"] - 2, ans, 10, "Times-Bold")
        if why:                                   # one line if it fits (shrinking to 6.5pt), else two tight lines
            from reportlab.pdfbase.pdfmetrics import stringWidth
            width, size = 553 - (sg["x1"] + 4), 8
            while size > 6.5 and stringWidth(why, "Times-Roman", size) > width:
                size -= 0.5
            if stringWidth(why, "Times-Roman", size) <= width:
                text(sg["x1"] + 4, sg["bottom"] - 2, why, size)
            else:
                lines = simpleSplit(why, "Times-Roman", 6.5, width)
                if len(lines) > 2:
                    lines = [lines[0], " ".join(lines[1:])[:int(width / 2.6)] + "..."]
                for i, line in enumerate(lines):
                    text(sg["x1"] + 4, sg["bottom"] - 4.5 + i * 7, line, 6.5)

    pct = data.get("btl_percent", {})
    for i, cell in enumerate(L["btl"][:6]):
        v = pct.get(f"L{i + 1}", 0)
        x0, t, x1, b = cell
        c.setFont("Times-Roman", 10.5)
        c.drawCentredString((x0 + x1) / 2, y((t + b) / 2 + 4), f"{v:g}%")

    if data.get("difficulty"):
        tick(data["difficulty"])
    if data.get("decision") in ("without", "minor"):            # only if the committee's decision was supplied
        tick("accepted_without_corrections" if data["decision"] == "without" else "accepted_with_minor_corrections")
    sx0, st, sx1, sb = L["setter"]
    for i, line in enumerate(simpleSplit(data.get("setter", ""), "Times-Roman", 10, sx1 - sx0 - 10)[:2]):
        text(sx0 + 5, st + 14 + i * 12, line, 10)
    c.save()

    base = PdfReader(form)
    overlay = PdfReader(io.BytesIO(buf.getvalue()))
    page = base.pages[0]
    page.merge_page(overlay.pages[0])
    w = PdfWriter(); w.add_page(page)
    with open(out, "wb") as fh:
        w.write(fh)
    total = sum(pct.get(f"L{i}", 0) for i in range(1, 7))
    return f"wrote {out} (Bloom's % total {total:g}%)"


def main():
    ap = argparse.ArgumentParser()
    sub = ap.add_subparsers(dest="cmd", required=True)
    d = sub.add_parser("draft"); d.add_argument("paper"); d.add_argument("-o", required=True); d.add_argument("--setter")
    f = sub.add_parser("fill"); f.add_argument("form"); f.add_argument("data"); f.add_argument("-o", required=True)
    a = ap.parse_args()
    if a.cmd == "draft":
        data = draft(a.paper, a.setter)
        json.dump(data, open(a.o, "w"), indent=2, ensure_ascii=False)
        print(json.dumps({k: v for k, v in data.items() if not k.startswith("_")}, indent=1, ensure_ascii=False))
        print("\nConfirm with the setter:"); [print(" -", x) for x in data["_confirm"]]
        if data["_review_items"]:
            print("REVIEW items left as the setter/panel chose:"); [print(" -", x) for x in data["_review_items"]]
    else:
        print(fill(a.form, json.load(open(a.data)), a.o))


if __name__ == "__main__":
    main()
```