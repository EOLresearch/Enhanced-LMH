# Enhanced LMH: Bereavement Writing App

A **Streamlit** web app prototype for grief (bereavement) journaling. The user picks a writing
prompt and writes about the person they lost. When they click **"Get AI Recommendations"**, the
app:

1. Checks the text for **safety risk**, using rules first and then the AI.
2. If it's safe, sends the entry to **Google Gemini** (`gemini-2.5-flash-lite`). The reply is a
   neutral description of what was written, plus follow-up writing prompts that point toward
   specific memories, places, or moments.

The assistant is deliberately **neutral and non-relational**. It does not show empathy, interpret
emotions, give advice, or use "I/we" language. It describes the content and invites more concrete
writing.

> **Research prototype.** It does not provide crisis support or therapy.

For setup and run steps, see **[USAGE.md](USAGE.md)**.

---

## Files

| File | What it does |
|---|---|
| `app.py` | The Streamlit UI: prompt picker, journal text box, word count, "Get AI Recommendations" button, and the assistant panel with reflections, follow-up questions, starters, and theme tags |
| `ai_service.py` | Gemini API calls. `reflect()` is used by the app. `continue_writing()` and `session_summary()` are helpers. Also reads the API key. |
| `prompts.py` | All prompt text: the `COMPANION_SYSTEM` system prompt, the 7-day guided prompts, prompt sets by mode, `THEME_TAGS`, and extra prompt lists |
| `safety.py` | Rule-based risk check (regex) → `LOW` / `ELEVATED` / `HIGH`, plus the crisis and support messages |
| `database.py` | SQLite helpers (`journal.db`). **Not currently used by `app.py`.** It's kept for saving entries later. |
| `.streamlit/config.toml` | App theme (colours, font) |
| `.streamlit/secrets.toml.example` | Template for the API key. Copy it to `secrets.toml`, which is gitignored. |
| `requirements.txt` | `streamlit`, `google-genai` |

---

## How it works

```
User picks prompt + writes entry
        │
        ▼  clicks "Get AI Recommendations"
safety.check(text)  ── regex rules
        │
        ├── HIGH ──────────────► show CRISIS_MSG (988, Crisis Text Line, 911). No AI call.
        │
        ├── ELEVATED ──► show 988 banner, then continue ▼
        │
        ▼  LOW / ELEVATED
ai.reflect(text, prompt)  ── Gemini, JSON output
        │
        ├── safety_flag = true ─► show CRISIS_MSG (the AI's own second safety check)
        │
        ▼
Show: neutral reflection · 3 follow-up questions · 3 continuation starters · theme tags
```

### Safety (two layers)
1. **`safety.py` rules:** run on the text before any AI call.
   - **HIGH** phrases (e.g. "kill myself", "want to die", "overdose", "no way out") block the AI
     and show crisis resources.
   - **ELEVATED** phrases (e.g. "no hope", "burden to", "can't take it anymore") show the 988
     banner, and the AI still runs.
2. **Gemini check:** the `reflect()` prompt tells the model to return `safety_flag: true` if it
   sees suicide, self-harm, or crisis language. The app then shows the crisis message instead.

### AI output (`reflect()`)
Gemini returns JSON with these fields:
- `reflection`: 1–3 neutral sentences about the content
- `follow_up_questions`: 3 prompts about a specific memory, place, sensory detail, or trigger
- `continuation_starters`: 3 sentence openers (e.g. "The place I most associate with this is…")
- `theme_tags`: up to 3 tags from `THEME_TAGS` (memories, yearning, regret, guilt,
  continued bond, …)

Results are cached for 1 hour (`@st.cache_data`), so the same text and prompt don't trigger a new
API call. Temperature is `0.75`.

---

## Notes for whoever takes this over

- **API key:** `GOOGLE_API_KEY`, read from `.streamlit/secrets.toml` or an environment variable.
  **Never commit the real `secrets.toml`.**
- **Changing the model:** `MODEL` at the top of `ai_service.py`.
- **Changing AI behaviour:** edit `COMPANION_SYSTEM` in `prompts.py` and the prompt inside
  `reflect()` in `ai_service.py`.
- **Changing the safety rules:** edit the `_HIGH` / `_ELEVATED` regex lists in `safety.py`.
- **Saving entries:** `database.py` is ready (`db.init()`, `db.save(...)`), but `app.py` doesn't
  call it yet. Entries are not stored anywhere right now.
- **Deploying:** the app is built for Streamlit Community Cloud. Put the key in
  App settings → Secrets.
