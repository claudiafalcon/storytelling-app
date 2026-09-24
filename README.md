# 🎈 Magic Story Maker

### ▶️ Live application

## https://storytelling-app-claudiafalcon.streamlit.app/

Deployed on Streamlit Community Cloud. The first story after the application
wakes up takes about 70 seconds, because the three models are loaded into
memory; later stories take about 55 seconds.

---

An AI-powered storytelling application for children aged 3–10.

A child uploads a picture, and the application looks at it, invents a short
story about it, and reads that story out loud.

Built with [Hugging Face Transformers](https://huggingface.co/docs/transformers)
pipelines and [Streamlit](https://streamlit.io).

---

## How it works

The application chains three pre-trained models together:

| Step | What happens | Function | Model |
|------|--------------|----------|-------|
| 1 | The picture is described in words | `image2text()` | `Salesforce/blip-image-captioning-large` |
| 2 | A children's story is written from that description | `text2story()` | `LiquidAI/LFM2-1.2B` |
| 3 | The story is turned into speech | `text2audio()` | `facebook/mms-tts-eng` |

The three models were chosen after comparing candidate models in
`IndividualProyect.ipynb`, using the same test images and the same prompt for
each candidate.

## Project structure

```
app.py                  The complete Streamlit application
requirements.txt        Python dependencies
.streamlit/config.toml  Storybook colour theme
README.md               This file
```

### Functions in `app.py`

| Function | Purpose |
|----------|---------|
| `get_hf_token()` | Reads an optional Hugging Face token from Streamlit secrets or the environment |
| `load_models()` | Loads the three pipelines once, cached with `@st.cache_resource` |
| `image2text(image, model)` | Returns a description of the uploaded picture |
| `text2story(caption, model)` | Returns a 50–100 word story for children aged 3–10 |
| `text2audio(text, model)` | Returns the narration waveform and its sampling rate |
| `audio_to_wav_bytes(audio, sampling_rate)` | Packs the waveform into a playable WAV file |
| `apply_styles()`, `render_*()` | Draw the child-friendly interface |
| `main()` | Orchestrates the whole application |

The script ends with the standard entry point:

```python
if __name__ == "__main__":
    main()
```

## Running it locally

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt

streamlit run app.py
```

The app opens at <http://localhost:8501>.

The first run downloads the three models from Hugging Face (about 4 GB in
total), so it takes a few minutes. After that the models are cached on disk
and start-up is much faster.

## Hugging Face access

All three models are **public**, so no token is required.

If a token is ever needed, the application reads it from Streamlit secrets or
from an `HF_TOKEN` environment variable. It is never written into the source
code.

```toml
# .streamlit/secrets.toml  (local only — this file is git-ignored)
HF_TOKEN = "hf_xxxxxxxxxxxxxxxx"
```

On Streamlit Community Cloud the same value goes in
**App settings → Secrets**.

## Deploying on Streamlit Community Cloud

1. Push this repository to GitHub.
2. Go to <https://share.streamlit.io> and create a new app from the repo.
3. Set the main file to `app.py`.
4. Deploy.

> **Note on performance.** `load_models()` sets no dtype, so the Transformers
> pipeline applies its default automatic dtype behaviour and loads each model in
> the precision recorded in its checkpoint configuration.
>
> Story generation is CPU-bound. Measured on a Colab CPU runtime, the selected
> model takes about **63 seconds per story**, compared with about 2 seconds on a
> GPU. Streamlit Community Cloud runs on CPU and throttles applications that
> sustain high CPU usage, so the first story may take significantly longer than
> this, or the app may be throttled during generation.
>
> Nine story models were evaluated before selecting this one. Every model below
> roughly 1B parameters failed at least one hard requirement, producing stories
> outside the 50-100 word range, incomplete stories, or content not appropriate
> for children. The full evaluation is documented in `IndividualProyect.ipynb`.

## Credits

Built as an individual university assignment on Hugging Face pipelines.
Model experimentation and selection are documented in `IndividualProyect.ipynb`.
