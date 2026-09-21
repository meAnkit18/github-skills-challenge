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

---
&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

