# How to Run (Quick Guide)

This app does **not** need AWS or saml2aws. It only needs a Google Gemini API key.

---

## 1. Get the code and install (one time)

```bash
git clone https://github.com/EOLresearch/Enhanced-LMH.git
cd Enhanced-LMH
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

(Next time, just `cd` into the folder and run `source .venv/bin/activate`.)

## 2. Add your API key (one time)

1. Get a key from https://aistudio.google.com/apikey, or from the lab's shared account.
2. Copy the template and paste the key in:

```bash
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
```

Then edit `.streamlit/secrets.toml`:
```toml
GOOGLE_API_KEY = "paste-your-key-here"
```

`secrets.toml` is gitignored. Never commit it.

## 3. Run

```bash
streamlit run app.py
```

Your browser opens at http://localhost:8501. Pick a prompt, write, and click
**Get AI Recommendations ✦**.

Stop the app with `Ctrl+C` in the terminal.

---

## Deploy online (Streamlit Community Cloud)

1. Go to https://share.streamlit.io and choose **New app**. Pick this repo, branch `main`, and
   file `app.py`.
2. In **App settings → Secrets**, add `GOOGLE_API_KEY = "..."`.
3. Deploy. If you change the secret later, restart the app.

---

## Common errors

| Error / message | Fix |
|---|---|
| "Missing GOOGLE_API_KEY" | `.streamlit/secrets.toml` is missing, or the key isn't set. See step 2. |
| "Replace the placeholder GOOGLE_API_KEY value" | You copied the template but didn't paste a real key |
| `ModuleNotFoundError: streamlit` / `google` | Activate the virtual environment (`source .venv/bin/activate`), then run `pip install -r requirements.txt` |
| "Could not generate a reflection" / 429 / quota | The API key is invalid or out of quota. Check it in Google AI Studio. |
| `streamlit: command not found` | Activate the virtual environment first |
