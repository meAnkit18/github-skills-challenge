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

---
&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

