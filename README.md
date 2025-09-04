## NIM App Scaffold

This project scaffolds a Python app that calls NVIDIA NIM OCR and LLM microservices, with Docker, docker-compose, Kubernetes manifests, Prometheus, and Grafana.

### Structure

```
project-root/
├── Dockerfile
├── docker-compose.yaml
├── k8s/
│   ├── nim-helm-values.yaml
│   └── deploy-app.yaml
├── mock_nim/
│   ├── Dockerfile
│   └── server.py
├── app/
│   ├── main.py
│   ├── nim_client.py
│   ├── requirements.txt
│   └── config.yaml
├── monitoring/
│   ├── prometheus.yaml
│   └── grafana-dashboard.json
├── scripts/
│   ├── build.sh
│   └── run_compose.sh
├── tests/
│   └── test_nim_client.py
└── README.md
```

### Quickstart (docker-compose)

1. Build and start:
```bash
./scripts/build.sh
./scripts/run_compose.sh
```

Note: `./scripts/run_compose.sh` runs `docker compose up --build`, so it will build images automatically; running `build.sh` is optional.

2. The app runs and calls a local mock NIM service `mock-nim` at `http://mock-nim:8000` (exposed on the host as `http://localhost:8000`).

2a. The app's web server is available at `http://localhost:9000`:
- `/` basic status
- `/run` executes the OCR→LLM pipeline
- `/metrics` Prometheus metrics
- `/healthz` health check

3. Monitoring:
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000` (add Prometheus data source, import dashboard JSON from `monitoring/grafana-dashboard.json`)

Note: The compose file includes a local `mock-nim` for development. Replace it (and any placeholder images) with your actual NVIDIA NIM proxy/services when deploying.

### Kubernetes

- Apply `k8s/deploy-app.yaml` (namespaced `nim`).
- Use `k8s/nim-helm-values.yaml` with your Helm charts for NIM services (proxy, OCR, LLM) and monitoring.

### Configuration

- `app/config.yaml` controls `nim_host`, model name, timeouts, and thresholds. You can override `NIM_HOST` and `CONFIG_PATH` via env vars.

### Running tests

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r app/requirements.txt
pytest -q
```

### main.py behavior

- Loads config, encodes `sample_scan.png` as data URL, sends to OCR `POST /v1/infer`, then sends extracted text to LLM `POST /v1/chat/completions` and prints the result.

### Notes

- Replace placeholder NIM images and proxy with your actual NIM services.
- Consider adding auth (API keys) and TLS per your deployment.


