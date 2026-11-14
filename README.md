# Lecture Q&A Assistant (RAG)

A retrieval-augmented generation (RAG) assistant that answers questions about a university course
("Computer Organization and Architecture") using only the content of six lecture files (3 PowerPoint, 3 Word).
Questions and answers are in Russian.

## How it works

1. **Load and chunk.** Text is extracted from `.pptx` (slide text) and `.docx` (paragraphs and tables) files
   and split into ~900-character chunks with 150-character overlap, trying not to cut sentences.
   The six lectures produce 234 chunks.
2. **Embed.** Each chunk is converted into a 3072-dimensional vector with the Gemini embedding model
   (`gemini-embedding-001`, task type `RETRIEVAL_DOCUMENT`). Questions use `RETRIEVAL_QUERY`.
3. **Store and search.** Vectors are stored in ChromaDB (cosine distance). For each question, the 5 closest chunks are retrieved.
4. **Answer.** The retrieved chunks are passed to a Gemini model with a system prompt that tells it to answer
   only from the given context, cite the source lectures, and say "В лекциях этого нет" ("Not in the lectures")
   when the context has no answer. If one model is overloaded, the code falls back to the next model in a list.

## Example results

| Question | Retrieved from | Behavior |
|---|---|---|
| Что такое дополнительный код и зачем он нужен? | Lecture 2 | Definition plus purpose, with source |
| Чем JK-триггер отличается от RS-триггера? | Lecture 5 | Explains the J = K = 1 case, with source |
| Из каких полей состоит число в формате IEEE 754? | Lecture 3 (slides) | Sign, biased exponent, mantissa, with slide numbers |
| Кто выиграл чемпионат мира по футболу в 2018 году? | n/a (out of scope) | "В лекциях этого нет" instead of a made-up answer |

## Tech stack

Python, Google Colab, Gemini API (`google-genai`), ChromaDB, `python-docx`, `python-pptx`.

## Run it

1. Open the notebook in Google Colab.
2. Add your Gemini API key as a Colab secret named `GEMINI_API_KEY` (never paste it into the code).
3. Upload your own lecture files (`.docx` / `.pptx`) named `Лекция_*` to `/content`.
4. Run the cells in order: install packages, chunk, embed and index, then ask questions with `ask("...")`.

The lecture files themselves are not included in this repository.

## Limitations

- Runs in a notebook with an in-memory vector database, so the index is rebuilt on each run. It is not deployed as a service.
- Only text is indexed; images and diagrams in the lectures are ignored.
- Retrieval on slide decks is weaker than on documents because slides contain little text. Some title and agenda slides rank high.
- No formal evaluation set yet; quality was checked manually on a handful of questions.

## Possible next steps

- Persist the index (ChromaDB persistent storage) and wrap the assistant in a small web API.
- Build a small evaluation set of questions with expected source lectures and measure retrieval accuracy.
- Deploy on Google Cloud Run.
