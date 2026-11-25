# She Watch Genie Ops Zero?!

Four Databricks SQL notebooks for the Bricking News article on Genie ZeroOps. The fictional streaming service runs on three fake purchases, a short stack of viewing events and one extremely expensive hyphen.

| Lab | What you do | What to look for |
| --- | --- | --- |
| `lab_01_green_baseline` | Seed two revisions of the same purchase feed. | 3 distinct USD purchases, 2,500 cents total. The TV app accounts for 2 purchases and 2,000 cents. |
| `lab_02_schema_drift` | Publish against v1, switch to v2, then select the v2 amount column. | The old column fails against v2. The recovery run reconciles back to the v1 total of 2,500 USD cents. |
| `lab_03_silent_underfill` | Write a 100-row batch and a separate 10-row batch. | Both jobs finish successfully. The 10-row batch fails the displayed data contract. DQM is a separate, optional trial. |
| `lab_04_review_the_fix` | Reproduce a bad run parameter. Separately, stage a bad Git commit with a correct run parameter. | Both can raise the same assertion. The first needs a corrected input. The second needs a source diff and a candidate run. |

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
databricks bundle run -t dev --profile YOUR_PROFILE lab_03_silent_underfill
databricks bundle run -t dev --profile YOUR_PROFILE lab_04_review_the_fix
```

Use **Jobs & Pipelines → Open ZeroOps → Configure**. Ask to monitor only the named lab jobs and disposable schema and confirm the saved scope. ZeroOps also looks back over the previous 24 hours. Use **Run now with different parameters** for the failure variants below. If you open notebooks interactively, set the named text parameters indicated in their cells. A job run supplies the run evidence for ZeroOps.

1. For **Lab 02**, run the defaults, then set `feed_table=zops_feed_v2` and retain `amount_column=amount_cents`. Expect SQLSTATE `42703` in the publish cell. Then keep v2 and set `amount_column=amount_minor`. Inspect the reconciliation cell for three purchases and 2,500 USD cents. Its assertion returns `NULL` on success. Compare the output's Delta history before and after the failed and recovered runs.
2. For **Lab 03**, run `mode=baseline, batch_id=baseline_01`, then `mode=thin, batch_id=thin_01`. Replay `thin_01` with the same mode and confirm the ten rows stay ten. Try `mode=baseline, batch_id=thin_01` to verify that the mode guard rejects the conflicting reuse. Use one writer at a time per schema. For a DQM trial, use a fresh schema and follow the notebook's baseline and scan protocol. Record Training or a missing scan as pending.
3. For **Lab 04**, use the normal value `tv_app`, then fail it with `tv-app`. Check the failed run's parameters before asking for a patch. Restore `tv_app` to confirm recovery. To compare instructions, make a second copy of the job with the same bad parameter and scope both copies. Let the first investigation finish, save the guidance below, then fail the second job. Confirm whether the second report loaded the instructions. Score both reports on the run and source evidence, the diagnosis and the action. Record unavailable if the second investigation never appears.

For example, run the two Lab 02 variants from the CLI. The first command is expected to exit with an error.

```bash
databricks bundle run -t dev --profile YOUR_PROFILE --params feed_table=zops_feed_v2 lab_02_schema_drift
databricks bundle run -t dev --profile YOUR_PROFILE --params feed_table=zops_feed_v2,amount_column=amount_minor lab_02_schema_drift
```

### Sample investigation instructions

Paste this into your ZeroOps configuration through Genie Code. It applies to new investigations for that configuration.

> For the lab purchase pipeline, compare observed channel values with the v1 control before recommending a patch. Preserve one row per `purchase_id`, the USD-cents convention and the contract assertion. Cite the failed run and the exact predicate. If the source evidence is unavailable, say what is missing. For a thin viewing batch, check Delta history and the source batch before recommending a replay.

### Lab 04 Git-source variant

1. Fork this repository and create a disposable incident branch in the fork. In `lab_04__review_the_fix.ipynb`, commit a one-line regression by replacing `WHERE channel = :channel_value` with `WHERE channel = 'tv-app'`. Protect the incident branch against direct pushes after seeding the regression. Create a separate, unscheduled Lakeflow Job with a **Notebook** task using remote **Git** as its source. Point it at the incident branch and the repository-relative notebook path. Set its warehouse, `catalog`, `schema` and `channel_value=tv_app`. Keep the input correct on both revisions.
2. Scope ZeroOps to the Git job, run it and capture the failure and run's Git SHA. A qualifying code-patch insight can offer **Create PR**. Inspect the proposal, confirm the push to a separate review branch, then open the PR against the incident branch via the link or GitHub. If the action is absent, create a review commit manually and label its origin.
3. Run a separate scratch job pinned to the candidate SHA with the same parameters and fixture. Reject a change that removes the assertion, admits the web purchase or changes the money units. Capture the verified result and PR link separately. You can leave the PR unmerged.

For a read-only rehearsal of the Git history, the companion repository already contains the [literal typo](https://github.com/SauronShepherd/she-watch-genie-ops-zero/commit/f4590bbc04f59344084a96d104c0ce4a78a28140) and a [revision with the working parameterized predicate](https://github.com/SauronShepherd/she-watch-genie-ops-zero/commit/6c6b7b4d836865743e49b404fae20fb30fe7eb25). Pin scratch runs to those SHAs with identical inputs to isolate the source change. Use your own fork for any write or PR.

## Finish the session

Remove the lab jobs and schema from the ZeroOps scope. Disable anomaly detection on the sandbox schema if you enabled it. Remove any manually created jobs and review branches you want to retire. `databricks bundle destroy -t dev --profile YOUR_PROFILE` removes the deployed bundle resources. The Delta tables were written by notebooks, so inspect and clean up the disposable schema separately. Record observations in [VALIDATION.md](VALIDATION.md).

[Genie ZeroOps setup](https://docs.databricks.com/aws/en/zeroops/get-started) · [Data Quality Monitoring](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-quality-monitoring/anomaly-detection) · [Git with Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/git)
