# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

### My notes - Part 1 Task 1

Working in my own fork, Codespace is up and running. Didn't touch the app code, just exploring for now.

We're watching `payment-service`. It logs metrics like response time, CPU, memory plus log level/message in `data/service_data.json`. Mostly normal (~130ms, INFO), but sometimes it spikes with timeouts and ERRORs.

Problem is to catch that weird behaviour early without checking logs manually.

That's where AIOps comes in - simple flow: data -> detector -> producer -> topic -> consumer -> output. Code for that is in `src/` (`anomaly_detector.py`, `event_producer.py`, `event_topic.py`, `event_consumer.py`, `aiops_pipeline.py`). No real Kafka, just in-memory simulation.

### My notes - Part 2 Task 2

Looked at `data/service_data.json` - 10 records for `payment-service`.

Metrics fields are `response_time_ms`, `cpu_percent`, `memory_percent`. Log info is `log_level` (INFO/ERROR), `message`, plus `service` name. `timestamp` is in both really - ISO format, one per minute from 10:00 to 10:09, so we can order events and spot when things went bad.

Normal is 8 records - 10:00-10:04 and 10:07-10:09. All INFO with "processed successfully", response 120-150ms, CPU 42-50%, memory 51-57%.

Unusual is 2 records:
- 10:05 - 610ms, CPU 75%, ERROR "Payment service timeout"
- 10:06 - 640ms, CPU 94%, memory 91%, ERROR "Database connection timeout"

So basically everything fine except that 2-minute spike where response time and resource use jump and errors show up.

### My notes - Part 3 Task 3

Ran it with `PYTHONPATH=src python3 src/aiops_pipeline.py`. Had to fix two small bugs first or it didn't work right - detector was checking for WARNING instead of ERROR, and producer/consumer were using two different topic objects so consumed was always 0. Now they share one `anomaly-events` topic. Didn't change the overall design, just those lines.

Result now: 10 processed, 2 anomalies detected, 2 consumed. Output lists service, timestamp, type and reasons, so you can see why each was flagged.

Detected:
- 10:05 - 610ms + ERROR "Payment service timeout" -> flagged High response time, Error log
- 10:06 - 640ms, CPU 94%, mem 91% + ERROR "DB connection timeout" -> flagged all four reasons

No miss - both ERROR spikes caught. No false alarm either - all 8 normal INFO records (120-150ms) left alone. Checked with `pytest` too, 8 passed.

One limitation: thresholds are fixed (500ms, 80%, 80%), so a slow drift like 400ms every time would never flag. Would be better with dynamic baselines or looking at trends.

### My notes - Part 3 Task 4

Checked the event flow step by step - all 6 pass:
1. detect makes an event (10:06 gives ANOMALY with 4 reasons)
2. that event goes to producer
3. producer publishes to `anomaly-events` topic (count goes 0->1)
4. consumer reads from same topic (gets 1 back)
5. consumer processes it (service/timestamp/reasons intact)
6. full `run_pipeline` gives 10 processed, 2 detected, 2 consumed - so it reaches the end.

Roles as I see it: Event/message is the dict with service/timestamp/type/reasons, Producer (`event_producer.py`) just publishes it, Topic (`event_topic.py`) is the in-memory list holding them, Consumer (`event_consumer.py`) reads them back for the AIOps output in `aiops_pipeline.py`.

Execution: `PYTHONPATH=src python3 src/aiops_pipeline.py` -> Records 10, Anomalies 2 (10:05, 10:06), Consumed 2. Same as Task 3.

### My notes - Part 4 Task 5

Found 2 real bugs that broke the flow, both within the existing design:

1. Detector missed ERROR logs - component `src/anomaly_detector.py:27`. Cause: it checked `log_level == "WARNING"` but data only has INFO/ERROR. Fix: changed to `"ERROR"`. Re-ran detector on 10:05 record - before it gave only [High response time], now gives [High response time, Error log detected]. Verified fixed.

2. Events never reached consumer - component `src/aiops_pipeline.py:17-22`. Cause: producer used `EventTopic("service-events")` and consumer used a separate `EventTopic("anomaly-events")` object, so consume always returned 0. Fix: both now share one `EventTopic("anomaly-events")`. Re-ran pipeline - before 2 detected / 0 consumed, now 2 detected / 2 consumed. Verified fixed.

No new architecture, just those lines. Also added `pytest.ini` so `pytest` works in CI for both import styles.

### My notes - Part 4 Task 6

Final end-to-end run with `PYTHONPATH=src python3 src/aiops_pipeline.py`:

Operational Data (10 records) -> Anomaly Detection (2 flagged) -> Event (ANOMALY dict) -> Producer (publish) -> Topic (`anomaly-events`) -> Consumer (consume) -> AIOps Output (printed list).

Result: Records 10, Detected 2, Consumed 2. All 7 checks pass - data processed, anomalies found (10:05 timeout, 10:06 DB timeout), events made, published, consumed, processed with service/timestamp/reasons intact, and output clearly shows the payment-service issue.

### My notes - Part 5 Task 7

This README already covers 1-8 above (scenario in Task 1, data in Task 2, observations in Task 2, findings in Task 3, flow in Task 4, final result in Task 6, fixes in Task 5, limitation in Task 3). Adding the reproduce steps here:

1. Open my fork codespace, no extra infra needed.
2. Install deps: `pip install -r requirements.txt` (plus `pytest`, `coverage` if you want CI checks).
3. Run pipeline: `PYTHONPATH=src python3 src/aiops_pipeline.py` - expect 10 / 2 / 2.
4. Run tests: `pytest --verbose` - expect 8 passed. Coverage version: `pytest --cov=src`.
5. Workflows run on push/PR via `.github/workflows/python-package.yml` and `python-coverage.yml`.

No screenshots, all written out as asked.

### My notes - Part 6 Task 8

Validation run just now, all green:
- `pytest --verbose` -> 8 passed (same as CI `python-package.yml`)
- `pytest --cov=src` -> 8 passed, ~60% total (CI `python-coverage.yml` will flag below-90 gate until Step 4 tests are added, that's expected)
- workflow YAML parses OK, both have required keyphrases (`pytest`, `pytest --cov=src`)
- AIOps checks: data processes (10), detection ok (normal=None, ERROR=flagged), 2 events made, 2 move through topic, consumer reads reasons, pipeline completes.

No failures to fix.

---
&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

