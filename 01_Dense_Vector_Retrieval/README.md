<p align = "center" draggable="false" ><img src="https://github.com/AI-Maker-Space/LLM-Dev-101/assets/37101144/d1343317-fa2f-41e1-8af1-1dbb18399719"
     width="200px"
     height="auto"/>
</p>

<h1 align="center" id="heading">Session 1: Dense Vector Retrieval</h1>

### [Quicklinks]()

| 📰 Module Sheet                                                                 | ⏺️ Recording | 🖼️ Slides | 👨‍💻 Repo       | 📝 Homework | 📁 Feedback |
| :------------------------------------------------------------------------------- | :----------- | :-------- | :------------ | :---------- | :---------- |
| [Dense Vector Retrieval](../00_Docs/Modules/01_Dense_Vector_Retrieval/README.md) |[Recording!](https://us02web.zoom.us/rec/share/sHWvo0Nd1aI0SEhKecOLEX9kFGVJJAdYfsKiuTmm8t85W48Z2lnjpnzTy8jAd8R5.PwuqibGwAZhvDd8c) <br> passcode: `C62n^@Q!`| [Session 1 Slides](https://canva.link/htfqf8i39yejyhn) | You are here! | [Session 1 Assignment](https://forms.gle/Z9qskfVaAvPjn6gz8) | [Feedback 6/2](https://forms.gle/21a2uoL9DVZPwgJP6) |


## 🏗️ How AIM Does Assignments

> 📅 **Assignments will always be released to students as live class begins.** We will never release assignments early.

Each assignment will have a few of the following categories of exercises:

- ❓ **Questions** - these will be questions that you will be expected to gather the answer to. These can appear as general questions, or questions meant to spark a discussion in your breakout rooms.

- 🏗️ **Activities** - these will be work or coding activities meant to reinforce specific concepts or theory components.

- 🚧 **Advanced Builds (optional)** - Take on a challenge. These builds require you to create something with minimal guidance outside of the documentation.

## Main Assignment

In this assignment, you will build a vector RAG application using LangChain v1, OpenAI embeddings, and Qdrant.

The main notebook is:

```text
01_Cat_Health_Vector_RAG_LangChain_Qdrant.ipynb
```

The notebook uses the bundled cat health guideline PDF in `data/cat_health_guidelines.pdf`.

### Setup

From this folder, install the environment with uv:

```bash
uv sync
```

Then open the notebook in Cursor or VS Code and select the Python/Jupyter environment created by uv.

You will also need an OpenAI API key available when running the notebook.

---

## 🏗️ Activity #1: Embedding Similarity

Run the embedding similarity primer in the notebook.

You will compare embeddings for terms like:

- `king`
- `queen`
- `banana`
- `cat`
- `veterinarian`
- `cat health guidelines`

#### ❓Question #1

Why is cosine similarity useful for dense vector retrieval?

##### ✅ Answer: 
Cosine similarity is useful because it compares the direction of two embedding vectors rather than their raw magnitude. In an embedding space, that direction can act as a useful signal for semantic similarity. By dividing the dot product by each vector's length, cosine similarity normalizes the comparison so one vector does not look more similar simply because it has a larger magnitude.

This makes cosine similarity useful for retrieval because the scores are comparable and easy to rank. In my run, this showed up clearly: `"king"` and `"queen"` scored `0.591`, while `"king"` and `"banana"` scored only `0.310`. Similarly, `"cat"` and `"cat health guidelines"` scored `0.496`, compared with `0.356` for `"cat"` and `"veterinarian"`. The more related pairs landed closer together, which is what I would expect from a semantic embedding model.

Vector databases like Qdrant use similarity measures such as cosine similarity to find the nearest chunk vectors to a query vector at scale. The caveat is that the score is a ranking signal, not absolute proof of meaning or correctness. Different embedding models can also produce different scores, so the score should be inspected along with the retrieved text.

---

## 🏗️ Activity #2: Build the Vector RAG Pipeline

Run the notebook sections that:

1. Load the PDF into LangChain `Document` objects
2. Split the document into chunks
3. Embed the chunks
4. Store the chunk embeddings in in-memory Qdrant
5. Retrieve relevant chunks with similarity scores
6. Generate an answer grounded in retrieved context

#### ❓Question #2

Why is metadata important for a RAG application?

##### ✅ Answer:
Metadata is important in a RAG application because it gives each retrieved chunk context beyond the raw text. In this notebook, metadata tells us the source file, page number, document type, and chunk location. That makes the system easier to debug, helps the model cite where an answer came from, and allows the application to filter or prioritize the right content. For the cat health guideline PDF, this is especially useful because recommendations depend on context such as life stage, section, and page. Embeddings help find semantically similar chunks, but metadata helps verify, cite, filter, and trust the retrieved information.

#### ❓Question #3

What tradeoff do we make when choosing chunk size and chunk overlap?

##### ✅ Answer:
Using the Chunk Visualizer makes the tradeoff clear: smaller chunks create more focused pieces of text, which can improve retrieval precision, but they can also fragment the surrounding context. Larger chunks preserve more context, but they can mix multiple ideas together and add noise to retrieval. Overlap helps carry context across chunk boundaries, but too much overlap duplicates text, increases the number of embedded chunks, and adds cost. In my run, 22 pages became 135 chunks using a chunk size of 1000 and overlap of 200, which seems like a reasonable balance between focused retrieval and preserving enough surrounding context.

#### ❓Question #4

What does a similarity score help you understand, and what does it not prove by itself?

##### ✅ Answer:

A similarity score helps me understand how closely a retrieved chunk matches the meaning of the user's query compared with the other chunks. In this example, the top results with scores around 0.58–0.55 are all related to signs that may require veterinary attention, such as pain, anxiety, vomiting, diarrhea, changes in activity, nocturnal vocalization, mobility issues, and behavior changes. However, the score does not prove that the chunk is fully correct, complete, or sufficient to answer the question. It is only a ranking signal. I still need to inspect the retrieved text to make sure the context is relevant, grounded in the PDF, and not missing important information.

---


## 🏗️ Activity #3: Vibe Check Retrieval Quality

Run the notebook's vibe check queries and inspect both:

- The retrieved context
- The generated answer

#### ❓Question #5

For the vibe check queries, did the retrieved context seem relevant before generation? Why or why not?

##### ✅ Answer:

Yes, the retrieved context seemed relevant before generation for the cat-health questions because those questions matched the topic of the PDF. For questions about preventive care, symptoms requiring veterinary attention, and feeding a healthy adult cat, the retrieved chunks contained related guideline content such as exam frequency, parasite prevention, appetite changes, vomiting, diarrhea, urination/thirst changes, behavior changes, mobility issues, nutrition, BCS/MCS, and energy requirements.
For the unrelated tax question, the retrieved context was not sufficient because the PDF is about feline health, not taxes. The model correctly responded that it did not have enough information in the provided cat health guideline PDF. Overall, the retrieval looked relevant for questions covered by the document and appropriately insufficient for a question outside the document’s scope.

---


## 🏗️ Activity #4: Tune Retrieval

Improve retrieval quality by changing one or more of:

- Chunk size
- Chunk overlap
- Retrieval `k`
- Query wording

Document what changed and whether retrieval improved.

##### Settings Changed:

- I tested two changes. First, chunk_size 1000 to 800 and chunk_overlap 200 to 150, rebuilding the splitter and vector store. Second, a more specific retrieval query using terminology closer to the source document, keeping the original chunking and k = 4.


##### Results:

1. Baseline (1000/200, 135 chunks): for "What symptoms should make me call a veterinarian?", the top three scores were moderate at about 0.435, 0.404, and 0.400, and the chunks were relevant.
2. Smaller chunks (800/150, 169 chunks): more chunks overall, but some were narrower and less self-contained, with no clear improvement in relevance.
3. Query rewording (original vector store): the top three scores rose to 0.682, 0.664, and 0.653, the biggest improvement of the two changes. Conclusion: the original 1000/200 chunking was reasonable, and query wording had the largest positive impact.

---

## Optional Deep Dive: RAG From Scratch

If you want to look underneath the library abstractions, run the optional reference notebook:

```text
02_Cat_Health_Vector_RAG_From_Scratch.ipynb
```

It builds the same retrieval pipeline again with only:

- `pypdf` for extracting text from the PDF
- Python standard-library HTTP requests for calling OpenAI
- Handcrafted document, chunking, embedding, similarity-search, vector-store, and generation primitives

This notebook is a reference walkthrough, not an additional assignment. Its purpose is to make the responsibilities hidden by LangChain, Qdrant, and provider SDKs visible.

---

## Submitting Your Homework

### Main Assignment

Follow these steps to prepare and submit your homework:

1. Pull the latest updates from upstream into the main branch of your AIE9 repo:

```bash
git checkout main
git pull upstream main
git push origin main
```

2. Start Cursor from the `01_Dense_Vector_Retrieval` folder.
3. Complete the notebook.
4. Answer the questions in this `README.md`.
5. Add, commit, and push your modified work to your origin repository.

When submitting your homework, provide the GitHub URL to your AIE9 repo.
