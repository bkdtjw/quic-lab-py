# quic-lab-py

A teaching-oriented QUIC/HTTP3 lab built with `aioquic`.
The project now supports:

- Minimal HTTP/3 client/server round-trip.
- QLOG generation for client and server.
- Parsing QLOG files into structured events.
- Metric extraction (handshake time, TTFB, drops, cwnd trend).
- Text/Markdown report generation.
- One-command experiment orchestration with `python -m quic_lab run`.
- Multi-scenario comparison runs.

## Requirements

- Python `>=3.9`
- Windows, Linux, or macOS for baseline flow
- Linux + `tc` (`iproute2`) only if you want `netem` weak-network injection

## Install

From project root:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e ".[dev]"
```

## Quick Start

Run a full end-to-end experiment (server start -> client request -> qlog -> metrics -> report):

```powershell
.\.venv\Scripts\python.exe -m quic_lab run --config config/default.yaml
```

Expected terminal output (example):

```text
[run] completed: run_20260314_133735
[run] HTTP status: 200
[run] qlogs: C:\Users\nirvana\Desktop\quic-lab\output\runs\run_20260314_133735\qlogs
[run] text report: C:\Users\nirvana\Desktop\quic-lab\output\runs\run_20260314_133735\report.txt
[run] markdown report: C:\Users\nirvana\Desktop\quic-lab\output\runs\run_20260314_133735\report.md
[run] metrics json: C:\Users\nirvana\Desktop\quic-lab\output\runs\run_20260314_133735\metrics.json
```

## Where Data Is Saved

`python -m quic_lab run` writes one run directory under:

- `output/runs/<run_id>/qlogs/*.qlog`
- `output/runs/<run_id>/report.txt`
- `output/runs/<run_id>/report.md`
- `output/runs/<run_id>/metrics.json`
- `output/runs/<run_id>/summary.json`

Fixed sample test set is in:

- `sample_logs/client_sample.qlog`
- `sample_logs/server_sample.qlog`

## Analyze Existing Logs

```powershell
.\.venv\Scripts\python.exe -m quic_lab.analyzer.qlog_parser --input sample_logs
.\.venv\Scripts\python.exe -m quic_lab.analyzer.metrics --input sample_logs
.\.venv\Scripts\python.exe -m quic_lab.analyzer.report --input sample_logs --format text
```

Save markdown report:

```powershell
.\.venv\Scripts\python.exe -m quic_lab.analyzer.report --input sample_logs --format markdown --output output\report.md
```

## Interactive Visualization with qvis

Generated qlog files follow standard qlog JSON structure and can be loaded directly in qvis:

- qvis: [https://qvis.quictools.info](https://qvis.quictools.info)

Steps:

1. Run one experiment:

```powershell
.\.venv\Scripts\python.exe -m quic_lab run --config config/default.yaml
```

2. Open the run output folder (printed by CLI), then find:

- `output/runs/<run_id>/qlogs/client_*.qlog`
- `output/runs/<run_id>/qlogs/server_*.qlog`

3. Open qvis in browser and drag the `.qlog` files into the page.

You can interactively inspect:

- connection / handshake timeline
- packet sequence and ACK behavior
- congestion window evolution
- stream-level HTTP/3 events

## Compare Scenarios

Run QUIC vs HTTP/2 side-by-side:

```powershell
.\.venv\Scripts\python.exe -m quic_lab compare --config config/default.yaml --protocol quic,h2 --rounds 1 --path /small?bytes=256 --path /large?bytes=4096
```

Outputs:

- `output/compare/<compare_id>/summary.md`
- `output/compare/<compare_id>/summary.json`

Notes:

- `--protocol` supports `quic`, `h2`, or `quic,h2`.
- On Windows/macOS, add `--no-netem` to skip Linux `tc` injection.
- HTTP/2 baseline raw run artifacts are saved under `output/baseline_h2/<run_id>/`.

## Local Dashboard

Run an interactive local dashboard (no CDN required):

```powershell
.\.venv\Scripts\python.exe -m quic_lab dashboard --host 127.0.0.1 --port 8080
```

Then open:

- `http://127.0.0.1:8080`

Main API endpoints:

- `GET /api/experiments`
- `GET /api/experiments/{id}/metrics`
- `GET /api/experiments/{id}/timeseries?metric=cwnd`
- `POST /api/compare`
- `POST /api/live/run`
- `WS /ws/live/{run_id}`

## Multiplexing Experiment (Single Connection, Multi Stream)

Run multiple HTTP/3 streams concurrently on one QUIC connection:

```powershell
.\.venv\Scripts\python.exe -m quic_lab multiplexing --path /large?bytes=1048576 --path /small?bytes=1024 --path /small?bytes=1024
```

Outputs:

- `output/multiplexing/<run_id>/qlogs/*.qlog`
- `output/multiplexing/<run_id>/report.txt`
- `output/multiplexing/<run_id>/metrics.json`

`metrics.json` now includes `per_stream` metrics such as:

- completion time per stream
- stream payload / throughput
- retransmission event counts per stream

## Weak Network (Linux Only)

These commands require Linux and `tc`:

```bash
python -m quic_lab weak-network --interface lo apply --delay-ms 50 --loss-pct 1
python -m quic_lab weak-network --interface lo show
python -m quic_lab weak-network --interface lo clear
```

On non-Linux systems the command exits with a clear message.

## Tests

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```

Current tests cover:

- QLOG parser/metrics/report chain with real sample logs.
- Experiment helpers and output artifact generation.
- Weak-network command construction behavior.

## Project Layout

```text
quic-lab-py/
  src/quic_lab/
    client/
    server/
    analyzer/
    baseline/
    dashboard/
    experiments/
  config/default.yaml
  sample_logs/
  output/
  tests/
```
