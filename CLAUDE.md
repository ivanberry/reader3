# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reader3 is a lightweight, self-hosted EPUB reader web application designed for reading books alongside LLMs. It processes EPUB files into a structured format and serves them via a web interface with a "Copy Chapter" feature for easy LLM integration.

## Commands

This project uses [uv](https://docs.astral.sh/uv/) for dependency management.

```bash
# Process EPUB files (place .epub files in books/ directory first)
uv run reader3.py

# Start the web server (runs on http://127.0.0.1:8123)
uv run server.py
```

## Architecture

### Two-Phase Design

1. **Processing Phase** (`reader3.py`): Parses EPUB files and creates a `book.pkl` pickle file containing structured data (metadata, chapters, TOC, images). Each book gets its own `{bookname}_data/` directory under `books/`.

2. **Serving Phase** (`server.py`): FastAPI web server that loads pickled books and serves them via Jinja2 templates.

### Key Data Structures (reader3.py)

- `Book`: Master object containing metadata, spine (chapters), TOC, and image mappings
- `ChapterContent`: Represents a spine item with cleaned HTML content and plain text extraction
- `TOCEntry`: Navigation tree entry with support for nested children
- `BookMetadata`: Title, authors, cover image, and other EPUB metadata

### Directory Structure

```
books/
  ├── example.epub           # Source EPUB files
  └── example_data/          # Processed book data
      ├── book.pkl           # Pickled Book object
      └── images/            # Extracted images
templates/
  ├── library.html           # Book grid listing
  └── reader.html            # Chapter reader with TOC sidebar
```

### Server Routes (server.py)

- `GET /` - Library view listing all processed books
- `GET /read/{book_id}/{chapter_index}` - Read a specific chapter
- `GET /read/{book_id}/images/{image_name}` - Serve book images
- `GET /cover/{book_id}/{image_name}` - Serve cover images

### Pickle Module Path Fix

When running `reader3.py`, classes are aliased to ensure pickle files use `reader3.X` paths instead of `__main__.X`. This allows `server.py` to correctly unpickle the book objects via `from reader3 import Book`.
