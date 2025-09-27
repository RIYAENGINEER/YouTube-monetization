# YouTube-monetization
YouTube Ad Revenue Predictor

A small project that fetches public YouTube video/channel metrics and uses a saved scikit-learn pipeline to estimate ad revenue for a single video.
This README explains how to set up, run, debug, and extend the project.

Features

Paste a YouTube video URL (web UI) or pass it on the command line.

Fetches public video statistics from YouTube Data API v3.

Prepares missing features required by the trained pipeline (best-effort).

Predicts estimated ad revenue using models/best_model.joblib.

Diagnostic helpers to inspect feature mismatch, dtypes and model metadata.

Repository layout (important files)
.
├── app.py                    # Streamlit web app (UI)
├── app_simple.py             # Alternative minimal Streamlit app (if present)
├── predict_revenue.py        # CLI script: predict via Python (console only)
├── diagnostics.py            # Diagnostics script: prints fetched DF, model info & traceback
├── models/
│   └── best_model.joblib     # Saved sklearn.pipeline.Pipeline (required)
├── src/
│   ├── __init__.py
│   ├── youtube_api.py        # wrapper to call YouTube Data API
│   ├── config.py             # optional: keep keys here (avoid committing)
│   └── ...                   # other helper modules
└── README.md

Quick start (recommended)

Create / activate virtualenv

python -m venv .venv
.\.venv\Scripts\Activate.ps1   # PowerShell
pip install --upgrade pip


Install dependencies
If the project has a requirements.txt:

pip install -r requirements.txt


If not, at minimum install:

pip install streamlit pandas scikit-learn joblib google-api-python-client


Set YouTube API key (temporary for current terminal)
Replace YOUR_KEY with the API key created in Google Cloud Console.

$env:YT_API_KEY="YOUR_KEY"


The code also checks YOUTUBE_API_KEY and src.config when available.

Enable the YouTube Data API v3

Visit Google Cloud Console → API Library → enable YouTube Data API v3 for the project that owns your API key.

Place the trained model
Make sure the trained pipeline is saved at:

models/best_model.joblib


The saved object should be a sklearn.pipeline.Pipeline that includes preprocessing and feature_names_in_ (recommended).

Run the Streamlit app (web UI)

Standard:

streamlit run app.py


Alternative (invoking streamlit as a module):

python -m streamlit run app.py


Open http://localhost:8501 and paste a YouTube URL. The UI will show the estimated ad revenue.

If you want a console-only option (no web UI), see the CLI section below.

Run from the command line (console)

Use the CLI helper:

# set the API key first (PowerShell)
$env:YT_API_KEY="YOUR_KEY"

python predict_revenue.py "https://www.youtube.com/watch?v=VIDEO_ID"


Output example:

Fetching stats for video id: 2kyXX8cRrX8

============================
Estimated Ad Revenue: $252.93
============================

Diagnostics & debugging
Diagnostics script

If prediction fails, run the diagnostics to see:

the DataFrame returned by the API,

dtypes and NaN counts,

model type and feature_names_in_,

full traceback on predict() failure.

# set API key
$env:YT_API_KEY="YOUR_KEY"
python diagnostics.py

Common problems & fixes

ModuleNotFoundError: No module named 'src.youtube_api'
Ensure you run from the project root (the folder that contains src/), and that src/__init__.py exists. Also check for accidental file name issues (e.g., youtube_api..py instead of youtube_api.py).

YouTube API key errors / 403 accessNotConfigured

Ensure the API key is correctly set in the environment variable expected by src.youtube_api.py: YT_API_KEY (or YOUTUBE_API_KEY, or via src/config.py, or Streamlit secrets).

Enable YouTube Data API v3 in Google Cloud Console for the key’s project.

If you changed API settings, wait a few minutes for propagation.

ValueError: columns are missing: {...}
Model expects more features than the API returned. Use prepare_features() (already included in the app) which:

computes video_length_minutes, watch_time_minutes, engagement, upload_month where possible,

fills reasonable defaults for missing columns (subscribers=0, country='unknown', etc.),

reorders columns to model.feature_names_in_.

ValueError: Cannot use median strategy with non-numeric data
This means the pipeline has numeric imputers but some columns are strings. The app’s prepare_features() converts:

date → numeric timestamp (seconds since epoch),

numeric columns to numeric dtype,

categorical fields to strings.
If you still hit this, run diagnostics.py to view dtypes and fix preprocessing.

Configuration options
Environment variables

YT_API_KEY — primary expected var. If you prefer, set YOUTUBE_API_KEY too, or add a YT_API_KEY entry in src/config.py.

Streamlit secrets: create .streamlit/secrets.toml:

YT_API_KEY = "YOUR_KEY"

src/config.py

You can (temporarily) store keys here during local development:

YT_API_KEY = "YOUR_KEY"


Do not commit src/config.py with real keys to a public repository.

Security note

Never commit API keys or secrets to version control.

If a key is exposed (for example, pasted into a chat or public place), revoke it immediately in Google Cloud Console and generate a new key. Then update your environment or secrets and re-run.

How the model expects features

The trained pipeline used in this project expects a specific set of columns (example order learned from the training run):

['video_id', 'date', 'views', 'likes', 'comments', 'watch_time_minutes',
 'video_length_minutes', 'subscribers', 'category', 'device', 'country',
 'engagement', 'upload_month']


Make sure your model file (best_model.joblib) contains a pipeline that either:

does all preprocessing within the pipeline, and exposes feature_names_in_ (recommended), or

you reproduce the same preprocessing in prepare_features() before calling predict().

Extending & improving accuracy

Add retrieving of channel subscriber count and country / device breakdown (if available) from the API or analytics dataset.

Improve watch_time_minutes by using real watch time if your data source provides it (YouTube Analytics, not public Data API).

Include geographic CPM proxies (higher CPM for US/UK etc.) as additional model features.

Retrain the model and save the final pipeline with preprocessing steps included (Pipeline([...preprocessors..., ('est', estimator)])) — this reduces runtime mismatches.

Troubleshooting checklist (short)

Are you in the project root and is src/__init__.py present?

Is models/best_model.joblib present and loadable?

Is YT_API_KEY set, and is YouTube Data API v3 enabled for the key's project?

Run python diagnostics.py to see fetched DF, dtypes, feature_names_in_, and predict traceback.

Adjust prepare_features() or update src.youtube_api.py to include missing fields (date, subscribers, watch_time, country, device, etc.).

Example commands
# 1) activate venv (PowerShell)
.\.venv\Scripts\Activate.ps1

# 2) set API key temporarily in this terminal
$env:YT_API_KEY="YOUR_KEY"

# 3) diagnostics
python diagnostics.py

# 4) predict via CLI
python predict_revenue.py "https://www.youtube.com/watch?v=2kyXX8cRrX8"

# 5) run streamlit web UI
python -m streamlit run app.py
# or simply:
streamlit run app.py

License

MIT — see LICENSE (or add your chosen license).
