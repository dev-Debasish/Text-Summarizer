# Text Summarizer

A small web application that generates a concise summary from text you enter. It uses a locally stored Hugging Face T5 sequence-to-sequence model, a FastAPI backend, and a simple browser interface.

## Features

- Paste or type text into a browser-based form and request a summary.
- Generate summaries with a locally loaded T5 model.
- Use the JSON API directly or interact through the web page.
- Select an available Apple MPS or NVIDIA CUDA device, with CPU as the fallback.
- View interactive API documentation through FastAPI.

## How it works

1. The browser sends the entered text to the backend as JSON.
2. The backend normalizes whitespace, removes HTML-like tags, and lowercases the text.
3. The tokenizer truncates or pads the input to 512 tokens.
4. The T5 model generates a summary with a maximum length of 150 tokens and beam search using four beams.
5. The generated text is returned to the browser and displayed on the page.

## Requirements

- A Python version supported by the installed PyTorch and Transformers releases
- A compatible T5 model and tokenizer saved in `saved_summary_model/` in Hugging Face Transformers format
- The Python packages below

Install the application dependencies:

```bash
python -m pip install fastapi uvicorn jinja2 transformers sentencepiece torch
```

PyTorch installation details vary by operating system and hardware. If you want GPU acceleration, install the PyTorch build that matches your CUDA or Apple Silicon setup by following the [official PyTorch installation instructions](https://pytorch.org/get-started/locally/). The app falls back to CPU when neither MPS nor CUDA is available.

## Model setup

The application loads its model and tokenizer from the relative path `./saved_summary_model`. That directory is not included in this repository. Before starting the server, place a compatible model checkpoint and tokenizer files there. The directory must be loadable by both:

- `T5ForConditionalGeneration.from_pretrained("./saved_summary_model")`
- `T5Tokenizer.from_pretrained("./saved_summary_model")`

Use a checkpoint that is compatible with the installed Transformers and PyTorch versions. The server must be launched from the project directory so that the relative model path resolves correctly.

## Run locally

From the project directory, start the FastAPI application with Uvicorn:

```bash
python -m uvicorn app:app --reload
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser. The first request may take longer while the model runs inference.

FastAPI's interactive API documentation is available at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs).

## API

### `POST /summarize/`

Request body:

```json
{
  "dialogue": "Paste the text you want summarized here."
}
```

Successful response:

```json
{
  "summary": "The generated summary appears here."
}
```

Example using `curl`:

```bash
curl -X POST "http://127.0.0.1:8000/summarize/" \
  -H "Content-Type: application/json" \
  -d "{\"dialogue\":\"Paste the text you want summarized here.\"}"
```

### `GET /`

Returns the web interface in `index.html`.

## Project structure

```text
.
├── app.py                 # FastAPI application, preprocessing, and model inference
├── index.html             # Browser interface and client-side API request
├── README.md              # Project documentation
└── saved_summary_model/   # Required local model/tokenizer files (not included)
```

## Input handling and limits

- The API accepts a JSON object with a `dialogue` string.
- Input is lowercased, whitespace is collapsed, and text matching the HTML-tag pattern is removed before tokenization.
- The tokenizer truncates inputs longer than 512 tokens; content beyond that limit is not summarized.
- The generated summary is limited to 150 tokens. These token limits are not character counts.
- The model and tokenizer are loaded when the application module starts, so the checkpoint must be present before launching Uvicorn.

## Troubleshooting

- **Model or tokenizer not found:** Check that `saved_summary_model/` is present in the project directory and contains the files required by the Hugging Face `from_pretrained` loaders.
- **Unexpected path errors:** Run Uvicorn from the project directory; the app uses relative paths for both the model and HTML template.
- **CUDA or MPS unavailable:** The app uses CPU when an accelerator is not available or recognized. CPU inference may be slower.
- **Dependency/import errors:** Install the packages listed in [Requirements](#requirements), and use a PyTorch build compatible with your Python version and hardware.

## Current scope

This project is a lightweight demonstration app. It does not include model training, checkpoint downloading, persistent storage, user accounts, or a configurable model path. The current input and output limits are set in `app.py`.
