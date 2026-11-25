# She Watch Genie Ops Zero?!

Four Databricks SQL notebooks for the Bricking News article on Genie ZeroOps. The fictional streaming service runs on three fake purchases, a short stack of viewing events and one extremely expensive hyphen.

| Lab | What you do | What to look for |
| --- | --- | --- |
| `lab_01_green_baseline` | Seed two revisions of the same purchase feed. | 3 distinct USD purchases, 2,500 cents total. The TV app accounts for 2 purchases and 2,000 cents. |
| `lab_02_schema_drift` | Publish against v1, then switch the run parameter to v2. | v1 succeeds. v2 fails on `amount_cents`. The new name is `amount_minor`, and the unit stays USD cents. |
| `lab_03_silent_underfill` | Write a 100-row batch and a separate 10-row batch. | Both jobs finish successfully. The 10-row batch fails the displayed data contract. DQM is a separate, optional trial. |
| `lab_04_review_the_fix` | Run the green contract, then use `channel_value=tv-app` to trigger a reviewable failure. | The normal value passes. The deliberate value fails, and a correct patch preserves 2 unique TV-app purchases and 2,000 cents. Optional Git-source variant exercises the PR path. |

## Requirements

- A recent Databricks CLI (the reference project uses 1.22.1), an authenticated profile and permission to deploy and run jobs in a development workspace.
- A Serverless or Pro SQL warehouse with `CAN USE` for the identity running the jobs. All notebook code cells use SQL.
- A writable Unity Catalog catalog and an existing disposable schema named `zeroops_lab_<alias>` with a lowercase letters/digits/underscores suffix. The run identity needs `USE CATALOG`, `USE SCHEMA`, `CREATE TABLE` and access to read, write or replace the tables it creates. Use a fresh schema for each participant. Catalog names in this bundle use simple alphanumeric identifiers with underscores.
- For the product exercises, a workspace admin must enable Genie ZeroOps in **Previews**. The configuring user needs access to the scoped assets. For the optional DQM exercise, anomaly detection requires its own setup and billing, serverless compute and `MANAGE` on the schema or parent catalog. For the optional PR exercise, connect a Git credential that can read your fork and write to a review branch.

## Deploy and run when ready

Set these variables in your shell before invoking the bundle CLI. The bundle defines four independent, unscheduled jobs. Run one at a time against a given lab schema.

```bash
export BUNDLE_VAR_catalog=YOUR_DEV_CATALOG
export BUNDLE_VAR_schema=zeroops_lab_your_alias
export BUNDLE_VAR_warehouse_id=YOUR_SQL_WAREHOUSE_ID

databricks bundle validate -t dev --profile YOUR_PROFILE
databricks bundle deploy -t dev --profile YOUR_PROFILE
databricks bundle run -t dev --profile YOUR_PROFILE lab_01_green_baseline
databricks bundle run -t dev --profile YOUR_PROFILE lab_02_schema_drift
```

Use **Jobs & Pipelines → Open ZeroOps → Configure**. Ask to monitor only the named lab jobs and disposable schema and confirm the saved scope. ZeroOps also looks back over the previous 24 hours. Use **Run now with different parameters** for the failure variants below. If you open notebooks interactively, set the named text parameters indicated in their cells. A job run supplies the run evidence for ZeroOps.

1. For **Lab 02**, use `feed_table=zops_feed_v2` after the successful v1 run. Save the run IDs, schema output and failed task error. Review the proposed fix against the v1 control. A v2 scratch copy using `SUM(amount_minor) AS revenue_cents` should return 2,500 cents for three USD purchases.
2. For **Lab 03**, run `mode=baseline, batch_id=baseline_01`, then `mode=thin, batch_id=thin_01`. New IDs create new batches. Repeating the same batch and mode preserves the existing rows. For a genuine DQM trial, start in a fresh schema and follow the notebook's baseline and scan protocol before introducing underfill. DQM training and intelligent scans make detection timing variable. The direct SQL check works independently of the beta. If you lack access to DQM, record only what the job and tables show.
3. For **Lab 04**, run the fourth bundle job with its default `channel_value=tv_app` to obtain a green acceptance run. Run it again with `channel_value=tv-app` to obtain the deliberate failure in workspace code. To compare instructions, make a second copy of the job with the same broken parameter and scope both copies. Let the first investigation start and finish, save the guidance below, then fail the second job and compare the evidence and proposed patch. Confirm the investigation reports which instructions it used. Record unavailable if ZeroOps does not produce a separate investigation.

### Sample investigation instructions

Paste this into your ZeroOps configuration through Genie Code. It applies to new investigations for that configuration.

> For the lab purchase pipeline, compare observed channel values with the v1 control before recommending a patch. Preserve one row per `purchase_id`, the USD-cents convention and the contract assertion. Cite the failed run and the exact predicate. If the source evidence is unavailable, say what is missing. For a thin viewing batch, check Delta history and the source batch before recommending a replay.

### Lab 04 Git-source variant

1. Fork this repository and protect the fork's base branch against direct pushes. Create a separate, unscheduled Lakeflow Job. Add a **Notebook** task with remote **Git** as its source. Set the URL to your fork, select GitHub and set the base branch to the original lab revision. The repository-relative notebook path is `lab_04__review_the_fix.ipynb`. Use the same SQL warehouse and task parameters `catalog` and `schema` as the bundle job.
2. Scope ZeroOps to the Git job, run it and capture the failure and run's Git SHA. A code-patch insight for a Git-backed job can offer **Create PR**. Inspect the proposal, confirm the push to a dedicated review branch, and open the PR against the protected base using the link or GitHub. Availability depends on the insight class and Git permissions.
3. Run a separate scratch job pinned to the candidate SHA. Compare it with the original failed run. Reject a change that removes the check, admits the web purchase or converts cents into dollars. This experiment is complete when you can explain the diff and show the contract result. The PR can stay unmerged.

## Finish the session

Remove the lab jobs and schema from the ZeroOps scope. Disable anomaly detection on the sandbox schema if you enabled it. Remove any manually created jobs and review branches you want to retire. `databricks bundle destroy -t dev --profile YOUR_PROFILE` removes the deployed bundle resources. The Delta tables were written by notebooks, so inspect and clean up the disposable schema separately. Record observations in [VALIDATION.md](VALIDATION.md).

[Genie ZeroOps setup](https://docs.databricks.com/aws/en/zeroops/get-started) · [Data Quality Monitoring](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-quality-monitoring/anomaly-detection) · [Git with Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/git)
