# k8s-Observability

A lightweight Helm chart for deploying observability components (metrics, logging, tracing) into a Kubernetes namespace. Designed to be a starting point that can be extended with Prometheus, Grafana, Jaeger, Fluentd, or any other telemetry tools.

The chart is intentionally minimal so that it can be adapted for different cloud providers and on-prem clusters. Configuration defaults are provided but everything can be overridden via values files or CLI arguments when installing with Helm.

## Quick Start

1. Clone or download the repository:

   ```sh
   git clone https://github.com/your-org/k8s-Observability.git
   cd k8s-Observability
   ```

2. Preview the rendered templates with default values:

   ```sh
   helm template observability . | kubectl apply -f - --dry-run=client
   ```

3. Install or upgrade the chart in the `observability` namespace:

   ```sh
   helm upgrade --install observability . \
     -n observability --create-namespace
   ```

4. Supply custom settings using a personal values file:
   ```sh
   helm upgrade --install observability . -f my-values.yaml
   ```

## Included Components

This chart bundles a basic observability stack. Each component is controlled by values in `values.yaml` and can be enabled or disabled independently.

- **Grafana** – Web-based dashboard and visualization tool. Charts, datasources and plugins can be provisioned via configuration. Default installation creates a `ClusterIP` service with basic admin credentials.
- **Prometheus (kube-prometheus-stack)** – Metrics collection, alerting, and service discovery for Kubernetes. Configured with sensible defaults such as 7‑day retention, nodeSelector/affinity for observability nodes, and remote write support.
- **Loki** – Log aggregation system optimized for Kubernetes logs. Grafana is configured to use Loki as a datasource.
- **Tempo** – Distributed tracing backend. Grafana uses Tempo for trace visualization and connects it to Prometheus and Loki for metrics/log correlations.
- **Namespace template** – Ensures the `observability` namespace exists before other resources are created.

Components not needed for a particular deployment can be toggled off in `values.yaml` or removed from the chart templates.

## Chart Configuration

All configurable values are defined in `values.yaml`. Common options include:

- `image.repository` / `image.tag` – base container image for components
- `resources` – CPU/memory requests and limits
- `nodeSelector`, `tolerations`, `affinity` – scheduling constraints
- `service.type` – ClusterIP/NodePort/LoadBalancer for exposed services

Use `helm show values .` to dump the current defaults.

## Directory Layout

```text
k8s-Observability/
├── Chart.yaml          # chart metadata and dependencies
├── values.yaml         # default configuration values
├── charts/             # nested chart dependencies (if any)
└── templates/          # Kubernetes manifest templates
    ├── _helpers.tpl    # template helpers and partials
    ├── namespace.yaml  # namespace manifest
    └── ...            # other resource templates
```

## Development & Testing

- Lint the chart for style and common mistakes:

  ```sh
  helm lint .
  ```

- Render templates locally for inspection:

  ```sh
  helm template observability .
  ```

- Package the chart for distribution:

  ```sh
  helm package .
  ```

- Add new templates or tweak existing ones; use `helm template` to see results.

## Contributing

Contributions are welcome! Feel free to open issues or pull requests with enhancements, bug fixes, or additional documentation.

## License

This project is open source and available under the [MIT License](LICENSE).

A copy of the full license text is also included in the `LICENSE` file.

---

### MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
