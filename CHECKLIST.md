# Submission checklist (deadline 2026-11-12 23:59 UTC, confirm on the Kaggle Timeline page)

## Bundles
| # | folder | paste as Writeup body | attach |
|---|---|---|---|
| 1 | `1_paper_and_notebook/` | `WRITEUP_paper.md` (about 2,400 words, cap 3,000) | `CoEvoGraph_SWE_C3GWM_Full_Implementation_v2026-11-11.ipynb`, `results_tables.md` |
| 2 | `2_contractbench/` | `WRITEUP_contractbench.md` | `contractbench.zip` |
| 3 | `3_contractgate/` | `WRITEUP_contractgate.md` | `contractgate.zip` |

Verified on 2026-11-11: both zips unpacked into a clean directory pass their tests (ContractBench 9, ContractGate 10); `contractbench selfcheck` resolves 480 of 480 tasks; the notebook re-executed top to bottom in the `reuse` profile with no cell errors.

## Release hygiene done
- No credentials, API keys, tokens or machine names in any file of the three bundles. The notebook no longer prints the host name and contains no absolute paths. The local vLLM client key is the placeholder `EMPTY`, which the unauthenticated local server ignores.
- The one real-looking email domain in the synthetic data (`x.io`) is replaced by the reserved `example.net`; every email-like string in the data is now under `example.com`, `example.org` or `example.net`.
- `NOTICE` names "the CoEvoGraph-SWE authors" and lists `THIRD_PARTY_NOTICES.md` (all dependencies are MIT or BSD-3-Clause). Natural-pair source repositories are MIT, BSD or Apache 2.0, and only identifiers and aggregate statistics appear in the notebook.

## You must do before submitting
2. **Confirm the copyright holder** in both `NOTICE` files. If any employer or institution has rights in this work, name it there and check its release policy.
3. **Two Final Submissions only.** The rules allow up to 2 Final Submissions. Submit three writeups as drafts if you like, and select two as Final.
4. **Manuscript PDF and DOCX.** They are no longer in this bundle. `coevo-witness/paper/` still holds the original manuscript, whose abstract and sections 4.3 to 4.4 describe the dual-agent, natural-pilot and DPO/RL results. If you attach it to any submission, bring it in line with the notebook first, because reviewers will compare the two.
5. **Authors.** Notebook, writeups and citations use "Senthilkumar Vijayakumar, Sidharth E". Check the names and team size on the form.

## Archive
Original (pre-2026-11-11) READMEs, datacard, writeups, baselines and the `v2026-11-09` notebook are kept outside the bundle in `coevo-witness/submissions/_archive_2026-11-10/` and `coevo-witness/notebooks/`. Do not upload them: the old notebook prints the machine name and absolute paths.
