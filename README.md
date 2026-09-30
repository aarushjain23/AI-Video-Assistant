# AI Video Assistant

An end-to-end AI assistant for meetings and videos. Give it a YouTube link or a local audio/video file and it transcribes the speech, summarises it, pulls out key decisions, action items and open questions, and lets you chat with the content using Retrieval-Augmented Generation (RAG).

## Features

- **Flexible input:** YouTube URL (via `yt-dlp`) or a local audio/video file (converted with FFmpeg/`pydub`).
- **Speech-to-text:**
  - English: OpenAI Whisper, running locally (model size configurable, default `small`).
  - Hinglish/Hindi: Sarvam AI speech-to-text-translate API, which transcribes and translates to English.
- **Summarisation:** chunked summarisation with a generated title, using Mistral AI.
- **Extraction:** key decisions, action items and questions raised.
- **RAG Q&A:** ask natural-language questions about the transcript. Built with LangChain (LCEL), ChromaDB and `all-MiniLM-L6-v2` sentence-transformer embeddings.
- **Streamlit UI:** custom-styled interface with a chat panel.

## How it works

```
YouTube URL / local file
        |
   yt-dlp + FFmpeg  ->  WAV (mono, 16 kHz)  ->  10-minute chunks
        |
   Whisper (English)  or  Sarvam AI (Hinglish -> English)
        |
     Transcript
     /        \
Summary +      ChromaDB vector store
Extraction     (500-char chunks, MiniLM embeddings)
(Mistral)              |
                 Retriever -> Mistral -> Answer
```

## Tech stack

Python, Streamlit, OpenAI Whisper, Sarvam AI, LangChain (LCEL), Mistral AI, ChromaDB, Sentence-Transformers, yt-dlp, FFmpeg, pydub.

## Project structure

```
app.py                  Streamlit app
main.py                 Command-line pipeline
core/
  transcriber.py        Whisper / Sarvam transcription
  summarizer.py         Summary and title generation
  extractor.py          Decisions, action items, questions
  vector_store.py       ChromaDB + embeddings
  rag_engine.py         RAG chain for Q&A
utils/
  audio_processor.py    Download, convert and chunk audio
Requirements.txt
```

## Setup

**Prerequisites:** Python 3.10+ and [FFmpeg](https://ffmpeg.org/download.html) installed and on your `PATH`.

```bash
git clone https://github.com/aarushjain23/AI-Video-Assistant.git
cd AI-Video-Assistant
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS / Linux
pip install -r Requirements.txt
```

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_key
SARVAM_API_KEY=your_sarvam_key     # only needed for Hinglish
WHISPER_MODEL=small                # optional: tiny, base, small, medium, large
```

## Usage

**Web app**

```bash
streamlit run app.py
```

Paste a YouTube URL or a local file path in the sidebar, pick the language (`english` or `hinglish`) and click **Analyse**. Then use the chat box to ask questions about the content.

**Command line**

```bash
python main.py
```

## Notes

- Whisper runs locally and downloads its model on first use.
- Hinglish transcription uses Sarvam AI's cloud API and needs a valid API key.
- Summaries, extraction and Q&A call the Mistral API.
