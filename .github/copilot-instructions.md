# Copilot Instructions

This project is an XAI-powered stock prediction system. It is a Python project utilizing a virtual environment (`.venv`).

## Project Context
- **Language**: Python (backend), HTML/JS (frontend)
- **Environment**: Virtual environment (`.venv`) is used for dependency management.
- **Architecture**: The architecture has two parallel ML processes combined with multi-layered XAI:
  1. **Time-Series Model**: Facebook Prophet to model the non-linear nature of historical price data (e.g., NIFTY 50 using `yfinance`).
  2. **Semantic Analyzer Model**: FinBERT (`yiyanghkust/finbert-tone`) to extract sentiment signals from financial news (scraped via Google News RSS).
- **Integration**: Models are combined via a Random Forest classifier (Buy/Hold/Sell signal) with TimeSeriesSplit cross-validation.
- **XAI Layer** (Explainable AI — core differentiator):
  1. **SHAP (Shapley Additive Explanations)**: TreeExplainer on the RandomForest to show per-feature impact on the predicted signal.
  2. **FinBERT Attention Maps**: Extracts last-layer CLS→token attention weights from FinBERT to visualize which words drove sentiment scoring. Subword tokens are merged; attention is min-max normalized to [0,1].
  3. **Prophet GradCAM (Temporal Sensitivity)**: Perturbation-based sensitivity analysis — divides historical prices into N segments, re-fits Prophet with noise injected into each segment, and measures forecast deviation. Higher deviation = that time window was more influential.
  4. **Prophet Changepoints**: Extracted from fitted Prophet model — dates where the trend slope changed significantly, with magnitude values.
  5. **Forecast Visualization**: Full historical + 30-day future forecast with confidence bands rendered as an interactive chart.
- **Domain**: XAI, Deep Learning, NLP, and Financial Forecasting.

## Module Guide
- `app/predictor.py` — Main pipeline orchestrator. Fetches prices, fits Prophet, scrapes news, runs FinBERT, trains classifier, generates SHAP/attention/sensitivity data. Results cached for 1 hour.
- `app/api.py` — FastAPI backend serving the frontend and `/api/predict/{symbol}` endpoint.
- `app/scraper.py` — Google News RSS scraper. Maps yfinance symbols to human-readable queries.
- `app/sentiment.py` — Standalone batch FinBERT sentiment scorer (for offline analysis on the JSON dataset).
- `app/time_series.py` — Standalone Prophet fitting (for offline analysis).
- `app/combine_and_xai.py` — Standalone merge + SHAP analysis with saved plots (offline mode).
- `app/frontend/index.html` — Single-page dashboard (Bootstrap 5 + Chart.js). Renders signal, SHAP bars, probability chart, forecast line chart, temporal sensitivity bars, attention heatmaps, changepoint cards, and news feed.

## Developer Workflows
- **Dependencies**: Ensure the virtual environment is activated before installing dependencies or running scripts.
  ```bash
  source .venv/bin/activate
  ```
- **Running the server**:
  ```bash
  uvicorn app.api:app --reload --port 8000
  ```
- **Adding Code**: Follow PEP 8. All new functions must have docstrings.
- **Data**: Data files are stored in the `data/` directory. Runtime outputs go to `results/`.
- **App**: Application code is stored in the `app/` directory.
- **Frontend**: Single HTML file at `app/frontend/index.html` — uses vanilla JS, Chart.js, Bootstrap 5.
- **Testing**: Run the server and test via browser at `http://localhost:8000`. Select any supported stock and verify all XAI panels render correctly.

## AI Agent Guidelines
- **Code Generation**: Write clean, idiomatic Python code.
- **Documentation**: Include docstrings for all new functions, classes, and modules.
- **Testing**: As the project grows, ensure tests are written alongside new features.
- **XAI**: When adding new XAI features, always: (1) compute the explanation data in `predictor.py`, (2) include it in the result dict, (3) add a visualization in the frontend HTML.
- **Performance**: The attention map and Prophet sensitivity analyses are compute-heavy. Keep `max_headlines` and `n_segments` reasonable (8 and 10 respectively).

*(Note: This file should be updated as the project architecture, specific frameworks, and conventions are established.)*
