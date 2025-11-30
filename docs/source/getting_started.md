# Getting Started

## Prerequisites

-   **Python 3.11+**
-   **uv** (for dependency management)

## Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Tianyi-Billy-Ma/AutoData.git
    cd AutoData
    ```

2.  **Install dependencies and environment:**
    ```bash
    uv sync --group dev,test,docs
    ```

3.  **Install browser binaries:**
    ```bash
    playwright install
    playwright install-deps
    ```

## Usage

To run a sample task using the default configuration:

```bash
uv run python -m autodata.main --config configs/default.yaml
```

### Inspecting Outputs

Results are saved in the `outputs/` directory:

```bash
ls outputs/default_run/
# ├── summary.json  (Metadata & Dataset reference)
# └── artifacts/...
```
