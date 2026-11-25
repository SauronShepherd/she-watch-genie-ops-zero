# Validation record

Validated on 2026-11-25 in workspace `dbc-916267ef-3b9b.cloud.databricks.com` with Databricks CLI 1.22.1, profile `sda`, catalog `sda_dev`, schema `zeroops_lab_angel_20261125` and warehouse `c0df22691f7b9ffc`. Genie ZeroOps inbox observations and DQM anomaly detection remain optional product exercises. The bundle itself was validated, deployed and exercised through both green and deliberate failure paths.

| Check | Expected | Record on first run |
| --- | --- | --- |
| Bundle | Four independent manual jobs with one SQL notebook task each | `Validation OK!`, deployment created four jobs and uploaded eight files |
| Lab 01 | 3 distinct USD purchases, 2,500 total cents, 2 TV-app purchases and 2,000 TV-app cents | Run `481521479257870` succeeded. Independent query returned `3 / 3 / 2500`. |
| Lab 02 v1 | One USD summary row with 3 purchases and 2,500 cents | Run `637874234260576` succeeded. `zops_daily_revenue` returned `1 / 3 / 2500`. |
| Lab 02 v2 | Failed task references unresolved `amount_cents`; v2 schema has `amount_minor` | Run `42345021874650` failed with the expected `UNRESOLVED_COLUMN` suggestion for `amount_minor`. |
| Lab 03 baseline | 100 rows for `sample_01` with 100 distinct event IDs | Run `80988534842520` succeeded. Independent query returned `100 / 100 / baseline`. |
| Lab 03 underfill | 10 rows for `thin_01` with 10 distinct event IDs and a visible non-PASS contract status | Run `92789719652764` succeeded. Independent query returned `10 / 10 / thin`. |
| Lab 04 green | 2 distinct USD TV-app purchases and 2,000 cents with the gate intact | Run `821258069765679` succeeded with the default `channel_value=tv_app`. |
| Lab 04 deliberate failure | Assertion fails with `tv-app` while the source value is `tv_app` | Run `1063653230259510` failed with the expected user-raised contract exception. |

For each future ZeroOps investigation, capture the signal, first useful evidence, root-cause accuracy, proposed action, what a reviewer changed and the recovery result. Keep the times for the actual failure, first insight, reviewed fix and verified recovery separate. A notification, a generated PR and a closed insight are distinct outcomes.
