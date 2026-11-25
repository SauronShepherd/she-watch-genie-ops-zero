# Validation record

Run in Databricks on AWS on 25 November 2026 with CLI 1.22.1, the `dev` target, a Serverless SQL warehouse and a disposable Unity Catalog schema. Bundle validation and deployment succeeded. Run IDs refer to the private test workspace. The fixture data are synthetic.

| Lab case | Run ID | What Databricks returned |
| --- | --- | --- |
| 01 · Seed and baseline | `346479869398460` | Success. The fixture assertion checked three distinct USD purchases and 2,500 cents, including two TV-app purchases and 2,000 cents. |
| 02 · v1 with `amount_cents` | `172300153188146` | Success. The published result passed reconciliation. |
| 02 · v2 with `amount_cents` | `159476446860733` | Expected failure with SQLSTATE `42703`. Databricks suggested `amount_minor`. |
| 02 · v2 with `amount_minor` | `904175530721566` | Success. Reconciliation passed. The output contained one USD summary with three purchases and 2,500 cents. |
| 03 · Normal batch | `542668437909294` | Success. 100 rows, 100 distinct event IDs, `PASS`. |
| 03 · Thin batch | `550377090563883` | Success. 10 rows, 10 distinct event IDs, `UNDERFILLED_OR_DUPLICATE`. |
| 03 · Replay the thin batch | `542995932839040` | Success. The batch still had 10 rows and 10 distinct event IDs. |
| 03 · Reuse thin batch ID in baseline mode | `339342823338609` | Expected failure in the replay guard. The existing batch stayed at ten rows. |
| 04 · Workspace code and `tv_app` | `110681501421053` | Success. The TV-app assertion passed. |
| 04 · Same workspace code and `tv-app` | `863275523711885` | Expected contract assertion failure with SQLSTATE `P0001`. |
| 04 · Restore `tv_app` | `61783004573596` | Success with the same code. The configuration change restored the contract. |
| 04 · Remote Git, literal typo | `70892926403499` | Expected contract assertion failure with SQLSTATE `P0001`. Databricks reported the requested snapshot `f4590bbc04f59344084a96d104c0ce4a78a28140`. |
| 04 · Remote Git, parameterized predicate | `505104972544114` | Success with identical run inputs and fixture. Databricks reported snapshot `6c6b7b4d836865743e49b404fae20fb30fe7eb25`. |

An independent SQL read after these runs confirmed 2 unique TV-app purchases and 2,000 cents, three purchases and 2,500 cents for recovered revenue, and 100 and 10 distinct event IDs in the two viewing batches. The initial Lab 03 version used a parameterized `DESCRIBE HISTORY` form rejected by the warehouse parser. The current notebook queries batch rows and ingestion timestamps. Inspect the Delta History tab separately for commit evidence.

The Git replay used one-time Jobs runs pinned to existing commits. The ZeroOps inbox, the custom-instruction comparison, a trained DQM alert and the agent-created PR remain separate product exercises. The run evidence above covers the incident inputs and recovery outputs.

For each ZeroOps investigation, record the failed run ID, source revision or notebook path, the evidence cited, what caused the failure, the proposed action and the verified outcome. Also record missing or inaccurate insights. Measure time to the first useful evidence separately from time to verified recovery. For DQM, add the training or scan status and timestamp. For Git, record the candidate commit, who reviewed it, the candidate run and the PR if one was opened.
