# 🦄 Magic Story Maker

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
| 2 | A children's story is written from that description | `text2story()` | `Qwen/Qwen2.5-1.5B-Instruct` |
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

> **Note on resources.** `Qwen/Qwen2.5-1.5B-Instruct` is a large model.
> `load_models()` sets no dtype, so the Transformers pipeline applies its
> default automatic dtype behaviour and loads the model in the precision
> recorded in the checkpoint configuration (`bfloat16`). The free Community
> Cloud tier is memory-limited and story generation on CPU takes noticeably
> longer than on a GPU. If the app runs out of memory, switching
> `STORY_MODEL` at the top of `app.py` to `Qwen/Qwen2.5-0.5B-Instruct` is
> the smallest change that fixes it.

## Credits

Built as an individual university assignment on Hugging Face pipelines.
Model experimentation and selection are documented in `IndividualProyect.ipynb`.
