# Validation record

This release contains the lab source and expected outcomes. Deployment, notebook runs and Genie ZeroOps observations are pending. The expected values below come from the checked-in synthetic inputs.

| Check | Expected | Record on first run |
| --- | --- | --- |
| Bundle | Four independent manual jobs with one SQL notebook task each | CLI version, target, workspace cloud/region, validation and deployment status |
| Lab 01 | 3 distinct USD purchases, 2,500 total cents, 2 TV-app purchases and 2,000 TV-app cents | Run ID and control output |
| Lab 02 v1 | One USD summary row with 3 purchases and 2,500 cents | Successful run ID and Delta output version |
| Lab 02 v2 | Failed task references unresolved `amount_cents`; v2 schema has `amount_minor` | Failed run ID, error, time to insight and evidence cited |
| Lab 03 | 100 vs. 10 rows for two new batch IDs; both runs succeed | Run IDs, contract output and commit times |
| Lab 03 optional DQM | An underfill or late-commit signal is possible after a trained baseline and a scan | Training state, time of scan, actual Quality result, insight time or pending |
| Lab 04 original | Assertion fails with `tv-app`; evidence query shows `tv_app` | Run ID, Git SHA if applicable, detected cause |
| Lab 04 candidate | 2 distinct USD purchases and 2,000 cents with the gate intact | Candidate SHA, diff, scratch result and PR link if applicable |

For each investigated incident, capture the signal, first useful evidence, root-cause accuracy, proposed action, what a reviewer changed and the recovery result. Keep the times for the actual failure, first insight, reviewed fix and verified recovery separate. A notification, a generated PR and a closed insight are distinct outcomes.
