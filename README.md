# CORES: an AI assistant for government forms

**Closed and Open Repository Enhanced System**
MSc Artificial Intelligence dissertation, University of Stirling (2024).
Author: [Adeshina Ayodele](https://engr-ayodele.vercel.app)

CORES is a conversational AI assistant that guides people, step by step, through completing government and care-service forms in plain language. It was designed with vulnerable adults in mind, including people applying for Scotland's Adult Disability Payment, who often depend on another person to get through long and complex forms.

## The problem

Benefit and referral forms are long, technical and stressful. Many applicants need a support worker or family member to complete them. CORES explores whether an AI assistant, grounded in the actual forms, can give people clear, patient guidance so they can complete forms more independently.

## How it works

![CORES architecture](docs/architecture.svg)

1. **Knowledge base.** Scanned form documents (PDF or Word) are read with PyMuPDF and docx2txt. Tesseract OCR extracts text from images inside the documents, which matters because many real forms are scans. The text is split into 1,000-character chunks with 200-character overlap, embedded with OpenAI embeddings and stored in a FAISS vector index.
2. **Retrieval.** For every user message, the three most relevant chunks are retrieved and added to the prompt (retrieval-augmented generation).
3. **Generation.** GPT-4, called through LangChain, answers using a carefully designed prompt, the retrieved context and a windowed conversation memory.
4. **Measurement.** Each response is timed, the full dialogue is saved to a timestamped log, and the user rates the session from 1 to 5 at the end.

### Prompt design

The prompt is a core part of the work. It instructs the assistant to:

* introduce itself with a consistent persona name for the session;
* use short, friendly, culturally sensitive sentences, one per line, for readability;
* guide the user one field at a time, completing each before moving on;
* reference the relevant form, section and page;
* follow the official Adult Disability Payment procedures published on mygov.scot;
* ask the user to rephrase rather than guess when it cannot find an answer;
* stay on task when a conversation drifts off topic;
* offer reassurance and, on request, summarise everything entered so far.

## Evaluation

| Measure | Result |
| --- | --- |
| Accuracy | **91.67%** (22 of 24 test exchanges correct) |
| Response time | 1.2 to 9.8 seconds per answer |
| Average response time by question type | Logical/clarification 6.44 s; procedural 4.85 s; inappropriate/incoherent 2.39 s; identity/existential 2.20 s |
| Robustness | Stayed polite and on task when users went off topic or tried to derail it, and corrected its own mistakes when challenged |
| Usability | Early user testing gave positive feedback, especially on the step-by-step approach; session ratings were mostly high |

Accuracy = correct responses ÷ total user queries × 100.

**Finding:** complex questions (procedural, logical, clarification) take longer because they need detailed, grounded answers. Simple or off-topic messages receive short, quick replies, which limits time spent on unproductive interactions.

## Limitations

* This is a research prototype, not an approved government or clinical service.
* The evaluation used a small number of test exchanges and users. Larger-scale testing is needed.
* The assistant follows mygov.scot procedures through its instructions; it does not fetch the website live.
* It runs as a text chat in a notebook. A web interface was planned but is not part of this prototype.
* The code targets `langchain==0.1.0`; newer LangChain versions changed these APIs.

## Responsible AI and data protection

* No real personal data is included in this repository.
* The original form documents are not published here; they belong to their publisher.
* API keys are never stored in the code. They are loaded from Colab Secrets or a local `.env` file.
* The assistant is instructed to stay within its purpose and to ask for clarification rather than invent answers.

## Running it

```bash
pip install -r requirements.txt
cp .env.example .env    # then add your own OpenAI API key
```

Put your own blank form templates (PDF or Word) in `documents/`, update the file list in the notebook, and run the cells in order. Tesseract OCR must be installed on your system (`apt-get install tesseract-ocr` on Linux).

## Next steps

* Larger evaluation with more users and more form types
* A web interface for non-technical users
* Live retrieval from official guidance, with source links in answers
* Extending the approach to other organisations' own knowledge bases

## Tech stack

Python · LangChain · OpenAI GPT-4 · OpenAI embeddings · FAISS · PyMuPDF · docx2txt · Tesseract OCR · Google Colab
