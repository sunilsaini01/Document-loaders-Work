# LangChain Document Loaders

Small practice repo for learning how LangChain's `document_loaders` module works — how different source types (CSV, plain text, PDF, a folder of PDFs, and a live web page) get converted into LangChain's `Document` objects, and how those documents feed into a simple LLM chain.

This isn't a library or a package, it's just five standalone scripts I wrote while going through the loaders one at a time.

## Why loaders matter

Every LangChain pipeline (RAG, summarization, QA over your own data) needs raw external data converted into a consistent format first. A `Document` is just:

```python
Document(page_content="...", metadata={...})
```

The loader's whole job is turning something messy (a CSV row, a PDF page, an HTML page) into that shape so downstream code — text splitters, embeddings, chains — doesn't need to know or care where the data originally came from.

## Files

| File | Loader | What it does |
|---|---|---|
| `csv_loader.py` | `CSVLoader` | Loads `Social_Network_Ads.csv`, one `Document` per row |
| `text_loader.py` | `TextLoader` | Loads `cricket.txt`, pipes it into a summarization chain |
| `pdf_loader.py` | `PyPDFLoader` | Loads a single PDF, one `Document` per page |
| `directory_loader.py` | `DirectoryLoader` + `PyPDFLoader` | Batch-loads every PDF in a `books/` folder |
| `webbase_loader.py` | `WebBaseLoader` | Scrapes a live URL, pipes it into a Q&A chain |

Data files included: `Social_Network_Ads.csv`, `cricket.txt` (a poem, used to test `TextLoader` + summarization).

Not included (referenced in the scripts but you need to supply your own): `dl-curriculum.pdf`, and a `books/` folder with a few PDFs in it for `directory_loader.py` to actually have something to glob.

## Setup

```bash
python -m venv venv
source venv/bin/activate        # venv\Scripts\activate on Windows

pip install langchain langchain-community langchain-openai python-dotenv pypdf beautifulsoup4
```

`text_loader.py` and `webbase_loader.py` call an LLM, so you need a `.env` file in the root:

```
OPENAI_API_KEY=sk-...
```

## Loader notes

**`CSVLoader`** — treats every row as its own `Document`. For a 400-row CSV that's 400 documents, each with the row's key-value pairs stuffed into `page_content` as text and `{'source': ..., 'row': n}` in metadata. Fine for small files; for anything large you'd want to think about whether per-row documents actually make sense for your use case (e.g. RAG over structured data usually needs a different strategy than one-doc-per-row).

**`TextLoader`** — the simplest one. Whole file becomes a single `Document`. Used here to feed a poem into a `PromptTemplate → ChatOpenAI → StrOutputParser` chain for summarization. Good minimal example of loader → chain wiring.

**`PyPDFLoader`** — one `Document` per PDF page, with page number in metadata. This is the detail that trips people up at first: `len(docs)` is the page count, not 1.

**`DirectoryLoader`** — wraps another loader (`PyPDFLoader` here) and applies it to every file matching `glob='*.pdf'` in a folder. Used `.lazy_load()` in this script, which returns a generator instead of a list — matters if you're loading a folder with a lot of large PDFs, since it avoids holding everything in memory at once. Note this script only prints `document.metadata`, not the actual page content.

**`WebBaseLoader`** — fetches a URL and parses it with BeautifulSoup, then hands the extracted text to a Q&A chain. Worth knowing: this does a plain HTTP GET + static HTML parse, it doesn't run JavaScript. The example URL here is a Flipkart product page, which is a JS-heavy SPA — so `docs[0].page_content` is likely to be mostly navigation/boilerplate text rather than clean product details, and the LLM's answer quality depends entirely on whatever actually made it into that static HTML. Good enough for simple static pages/docs sites; for JS-rendered pages you'd want something like `SeleniumURLLoader` or `PlaywrightURLLoader` instead.

## Things I want to try next

- `JSONLoader` and `UnstructuredLoader` for messier real-world files
- Comparing `WebBaseLoader` output against a headless-browser loader on the same JS-heavy page, to actually see how much content gets lost
- Chunking the CSV/PDF docs with a text splitter before doing anything RAG-related, instead of using the raw per-row/per-page documents directly
- A proper `requirements.txt` instead of installing things ad hoc

## Mental model

A loader's only contract is: **turn some external source into a list of `Document(page_content, metadata)` objects.** Everything downstream in LangChain — splitters, embeddings, retrievers, chains — is built to consume that one shape, regardless of whether the data came from a spreadsheet, a PDF, or a website. Once that clicks, the loaders stop feeling like separate APIs to memorize and start feeling like the same pattern with different parsers underneath.
