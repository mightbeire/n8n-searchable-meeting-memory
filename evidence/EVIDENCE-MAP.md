# Evidence Map

This folder contains selected proof from the original project archive. The images were reviewed before publication. One meeting screenshot was redacted to remove participant information and the private meeting URL.

## 01 — Executed meeting-memory workflow

`01-executed-meeting-memory-workflow.png`

**Supports:** The n8n canvas contains the intake, Google Drive, summary, Slack, Pinecone ingestion, and Slack retrieval paths. Several nodes show successful execution states.

**Does not prove:** production load, retrieval accuracy at scale, or long-term reliability.

## 02 — Browser capture client

`02-browser-capture-client.png`

**Supports:** A custom “Universal Meeting Minutes Agent” browser interface existed and exposed recording/transcription controls.

**Does not prove:** the source code is included in this repository. It is not.

## 03 — Live meeting capture, redacted

`03-live-meeting-capture-redacted.png`

**Supports:** The browser capture interface was used beside a Google Meet session and showed an active recording state.

**Redaction:** participant information and the meeting URL were obscured.

**Does not prove:** transcription accuracy or participant consent.

## 04 — Transcript sent to n8n

`04-transcript-sent-to-n8n.png`

**Supports:** The capture interface reached a state that reported the transcript as sent to n8n.

**Does not prove:** every downstream node completed successfully for that exact meeting.

## 05 — Ingestion and indexing pipeline

`05-ingestion-and-indexing-pipeline.png`

**Supports:** The workflow architecture connects meeting intake to Drive storage, summarization, Slack delivery, and a Drive-to-Pinecone indexing path.

## 06 — Slack RAG retrieval pipeline

`06-slack-rag-retrieval-pipeline.png`

**Supports:** The workflow contains a Slack-triggered AI Agent with Pinecone retrieval, embeddings, memory, and Slack response nodes.

**Does not prove:** factual accuracy for all possible questions.

## Evidence rule

Use these images only for claims they support. Do not convert architecture evidence into claims about production scale, accuracy, cost savings, or business results that were not measured.
