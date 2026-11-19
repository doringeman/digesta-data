# Digesta: statute retrieval evaluation set for the Romanian Civil Code

Digesta links statements from Romanian legal admission exams to the articles of the
Romanian Civil Code that decide whether each statement is true or false. The
statements come from 863 civil-law questions of the admission examinations for
magistrates (INM) and lawyers (INPPA), taken from the JuRO dataset. The set is meant
for evaluating systems that, given a statement or a case, return the applicable
articles, and optionally say whether each article supports or opposes the statement.

The set is described in the paper *Building a Statute Retrieval Evaluation Set for
the Romanian Civil Code: LLM-Assisted Annotation and Expert Validation* (RoEduNet
2026). This repository holds the data only.

## Quick start

The dataset is **`eval/dev.jsonl`**: one statement per line, with the Civil Code
articles that decide it. The article text is in `corpus/cciv.jsonl`, under the same ID.

```python
import json

code = {a["id"]: a for a in map(json.loads, open("corpus/cciv.jsonl", encoding="utf-8"))}
for line in open("eval/dev.jsonl", encoding="utf-8"):
    s = json.loads(line)
    print(s["enunt"], s["varianta"], "->", "true" if s["adevarata"] else "false")
    for aid in s["gold"]:
        print("  art.", code[aid]["numar"], "-", code[aid]["denumire_marginala"])
    break
```

## Status

- The decisive articles were **proposed by a language model** under a written
  protocol. The model could only choose article identifiers, and every quoted passage
  was checked mechanically against the corpus. No statutory text in this repository
  was written by a model: all of it comes from the official consolidated Code.
- **Review by legal experts is under way.** In a first review of 60 pilot statements,
  lawyers and magistrates judged the proposed articles correct for 54 (90%). The
  experts saw the proposals and the pilot is not a random sample, so treat this as an
  upper bound. Until the review is complete, treat `gold` as model proposals.
- **The test labels are withheld.** The test statements are released without their
  decisive articles. They are no less finished than the dev labels: both are model
  proposals awaiting expert review. Test is held back so that it stays a fair
  benchmark. Systems are tuned on dev, where the answers are visible, and scored on
  test, where they are not. Published answers invite tuning on them, and files in
  public repositories can end up in the training data of future language models. The
  test labels will be published once experts have validated them.

## Files

| Path | Content |
|---|---|
| `corpus/cciv.jsonl` | the Civil Code, one article per line (2,674 articles) |
| `corpus/cciv.meta.json` | source, download date, SHA-256 of the downloaded file, last incorporated amendment |
| `questions/grile.jsonl` | the 863 questions as parsed from JuRO, with options and answer keys |
| `questions/grile.meta.json` | JuRO commit, files used, counts, removed duplicates |
| `eval/dev.jsonl` | **development set**: 2,070 statements with their decisive articles |
| `eval/test.jsonl` | **test set**: 519 statements, decisive articles withheld |
| `annotations/a1_dev.jsonl` | proposals of annotator A1 on dev: passages, confidence, flags, rationale |
| `annotations/a2_dev.jsonl` | proposals of annotator A2 on dev: chosen sections, passages, check result |
| `protocol/instructiuni_adnotare.md` | the annotation protocol (Romanian) |

## Corpus record

```json
{"id": "cciv:art-1724", "numar": "1724",
 "denumire_marginala": "Sancțiunea neplății prețului",
 "cale": "Cartea V > Titlul IX > Capitolul I > Secțiunea 1",
 "cale_denumiri": "Cartea V — Despre obligații > ... > Secțiunea 1 — Dispoziții generale",
 "text": "...", "alineate": [{"nr": "1", "text": "..."}], "trimiteri": ["cciv:art-..."]}
```

- The source is the consolidated Civil Code (Law 287/2009, republished in 2011) from
  legislatie.just.ro, downloaded on 2026-09-25. It includes amendments up to Law
  57/2026, in force since 3 May 2026. The downloaded HTML is not redistributed; its
  SHA-256 is in `cciv.meta.json`.
- Identifiers are stable and cover inserted articles: article 2029^1 is `cciv:art-2029-1`.
- Editorial notes and amendment annotations are removed. Three defects of the source
  are corrected and listed in `cciv.meta.json`.
- `trimiteri` lists explicit cross-references; ranges longer than five articles are
  left out, because they refer to a whole legal regime.

## Evaluation record

```json
{"id": "inm-train-0166-C",
 "enunt": "Prestația compensatorie:",
 "varianta": "încetează prin recăsătorirea debitorului",
 "adevarata": false,
 "gold": ["cciv:art-395"],
 "gold_extins": [],
 "sursa": "craciuncg/GRAF@7e52178:JuRo/train_set/inm/inm_civil.csv_train.csv#166"}
```

| Field | Meaning |
|---|---|
| `id` | `<source>-<JuRO split>-<question>-<option>`; the question number is its position in the JuRO file |
| `enunt`, `varianta` | the question stem and one option; the query is the stem followed by the option |
| `adevarata` | truth value under the official answer key. It also gives the direction: for a true statement the `gold` articles support it, for a false one they refute it |
| `gold` | the decisive articles: the minimal set without which the truth value cannot be established (dev only) |
| `gold_extins` | relevant but not decisive articles, such as explicit cross-references; retrieving them is not required, but is not an error (dev only) |
| `sursa` | file, row and commit in the JuRO repository |

## Splits and statistics

The splits follow JuRO: its train and validation questions form dev, its test questions
form test. Use dev to develop and tune; test is for reporting results once its labels
are released (see Status).

| | dev | test |
|---|---|---|
| INM statements | 600 | 150 |
| INPPA statements | 1,470 | 369 |
| total | 2,070 | 519 |
| true / false | 962 / 1,108 | 239 / 280 |

On dev: 2,668 statement–article assignments (814 distinct articles, 1.29 per
statement); 8 statements have no decisive article in the Civil Code; 407 have a
non-empty extended set.

## Measures

Compute each measure per statement, average over statements, and report separately
by source (INM, INPPA) and by truth value:

- **F2 on `gold`**, the primary measure, as in COLIEE Task 3: missing a decisive
  article costs more than returning an extra one.
- **Tolerant F2**: articles in `gold_extins` are not counted as errors.
- **Direction**: for systems that label articles as supporting or opposing, the share
  of retrieved decisive articles placed on the correct side.

## Annotations

`annotations/a1_dev.jsonl` (A1, Claude `claude-opus-5-5`), one statement per line:

| Field | Meaning |
|---|---|
| `articole` | proposed decisive articles: `numar`, `alineat`, and `citat`, the decisive passage copied verbatim. An article can appear once per relevant paragraph |
| `suport` | articles needed to apply a decisive article, such as definitions |
| `motivare` | the annotator's rationale, generated by the model: not legal advice, and not reviewed |
| `incredere` | `sigur` (certain), `probabil` (probable) or `nesigur` (uncertain) |
| `semnale` | flags: `problema_sursa` (damaged source text), `barem_vs_text_actual` (the key seems inconsistent with the current Code), `enunt_ambiguu` (ambiguous), `in_afara_codului` (decided outside the Civil Code) |

`annotations/a2_dev.jsonl` (A2, gpt-oss-120b run locally, a different model family):
`articole` (its final set), `sectiuni` (the parts of the Code it chose to read) and
`propuneri` (each passage, with `citat_valid` from the mechanical check).

The protocol mentions a command-line search tool (`cauta_corpus.py`); it will be
released with the code.

## Known issues in the source questions

- **Some INM answer keys appear truncated.** No JuRO question has three correct
  options. 33 statements were flagged where a one-letter INM key marks false an option
  that reproduces the Code; in 32 of them the option comes right after the key letter
  (chance would give about 54%). Some two-letter keys seem to keep only their first
  letter. For these statements the decisive article is likely right, but the truth
  value and the direction are inverted.
- **Damaged text.** 123 of 613 INPPA and 20 of 250 INM questions have damaged text: 40
  INPPA stems are empty, and many options are cut off. The defects are in the source
  files; the flags let you exclude or correct these statements.
- **Temporal validity.** Answer keys reflect the law at the examination date, while
  the corpus is the 2026 consolidated text, and JuRO does not record examination years.

## Licences

- The data in this repository is released under **CC BY 4.0** (see `LICENSE.md`).
- The JuRO questions keep their MIT licence (`LICENSE-JuRO`).
- Romanian official legislative texts are not protected by copyright (Law 8/1996,
  art. 9 lit. b).

This is a research resource. It does not constitute legal advice.

## Citation

```bibtex
@inproceedings{geman2026digesta,
  author    = {Geman, Dorin-Andrei and Dieaconu, Vlad-{\c{S}}tefan and
               Rughini{\c{s}}, R{\u{a}}zvan-Victor and Resul, Ebru},
  title     = {Building a Statute Retrieval Evaluation Set for the {R}omanian Civil Code:
               {LLM}-Assisted Annotation and Expert Validation},
  booktitle = {Proceedings of the RoEduNet Conference 2026},
  year      = {2026},
  note      = {To appear}
}
```

## Contributing as a legal expert

Lawyers and magistrates who would like to help review the annotations can write to
dorin_andrei.geman@stud.acs.upb.ro or doringeman@gmail.com for an access code to the
review form.
