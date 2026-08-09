# Setup Guide — Searchable Meeting Memory System

This guide reproduces the public n8n backend safely. It assumes that you have a meeting transcription client or another approved way to create a text transcript.

The original browser capture client is shown in the evidence screenshots, but its source code was not present in the project archive. This guide therefore defines the input contract instead of pretending that the missing client can be reproduced from this repository.

## 1. What you need

Create accounts or access for:

- n8n;
- Google Drive;
- Slack;
- Pinecone;
- OpenAI, for embeddings; and
- Google Gemini, for meeting summaries and the retrieval agent.

You also need a Slack channel for meeting summaries and questions.

## 2. Import the workflow

1. Open n8n.
2. Create a new workflow.
3. Import `workflows/searchable-meeting-memory-system.json`.
4. Keep the workflow inactive until every credential and resource ID is configured.

The public export has no credential bindings. This is intentional.

## 3. Create the Google Drive folder

Create one Google Drive folder for meeting transcript files. For example:

```text
Team Meeting Transcripts
```

Copy the folder ID from Google Drive.

Configure these nodes with that folder:

- `Upload file`
- `Google Drive Trigger`

The public workflow uses the placeholder:

```text
REPLACE_WITH_GOOGLE_DRIVE_FOLDER_ID
```

Replace it in the n8n UI. Do not commit your real folder ID back to a public workflow export.

## 4. Configure Google Drive credentials

Create a Google Drive OAuth2 credential in n8n. Attach it to:

- `Upload file`;
- `Google Drive Trigger`; and
- `Download file`.

Confirm that the credential can read and write the transcript folder.

## 5. Configure the meeting intake webhook

The public workflow exposes a POST webhook path named:

```text
meeting-memory-intake
```

The capture or transcription client must send JSON in this form:

```json
{
  "transcript": "The meeting transcript goes here."
}
```

n8n exposes webhook request data under the request body. The public workflow normalizes `body.transcript` into a field named `transcript` before it creates the text file.

### Minimum test

Send a short synthetic transcript first. Do not test with confidential meeting data.

Example content:

```text
The team approved the new onboarding flow. Jordan will update the checklist by Friday. The next review is Monday.
```

Expected result:

1. The webhook receives the request.
2. `Edit Fields` creates a normalized `transcript` field.
3. `Convert to File` creates a text file.
4. `Upload file` writes the file to Google Drive.

## 6. Configure Gemini for the summary path

Create a Google Gemini credential in n8n. Attach it to:

- `summary of key decisions`; and
- `Google Gemini Chat Model`.

The summary prompt uses the normalized transcript and requests three bullets that contain key decisions.

Test the node with a synthetic transcript. Check the output before you connect it to a real Slack channel.

## 7. Configure Slack

Create the required Slack credentials in n8n. The workflow uses Slack for two jobs:

1. post a new meeting summary; and
2. accept questions about past meetings and return an answer.

Choose one test channel first.

Replace:

```text
REPLACE_WITH_SLACK_CHANNEL_ID
```

in:

- `send summary to team`; and
- `Slack Trigger`.

Attach the correct Slack credential to:

- `send summary to team`;
- `Slack Trigger`;
- `Send a message in Slack`; and
- `Send a message`.

### Prevent reply loops

The workflow contains an `If` node that rejects Slack events created by bots. Keep this control. Without it, a bot can respond to its own message and create a loop.

For a production deployment, narrow the Slack trigger further. Listen only to the event types and channels that the assistant needs.

## 8. Create the Pinecone index

Create a Pinecone index for meeting memory. The public workflow uses this name:

```text
meeting-memory
```

Configure your Pinecone credential in n8n and attach it to both Pinecone Vector Store nodes.

The workflow has two Pinecone modes:

- **insert** — stores transcript chunks; and
- **retrieve as tool** — lets the AI agent search stored meeting content.

Pinecone documents direct n8n support for RAG workflows through its Vector Store integration. Its current guidance also supports managed Assistant workflows if you want a more managed retrieval layer later.

## 9. Configure OpenAI embeddings

Create an OpenAI credential in n8n. Attach it to both embedding nodes:

- `Embeddings OpenAI`;
- `Embeddings OpenAI1`.

The same embedding model and vector dimensions must be compatible with the Pinecone index configuration.

Do not change the embedding model after you have indexed production data unless you plan a controlled re-index.

## 10. Configure transcript loading and chunking

The ingestion branch uses:

- `Download file`;
- `Default Data Loader`;
- `Recursive Character Text Splitter`;
- `Embeddings OpenAI`; and
- `Pinecone Vector Store`.

The original workflow uses 200 characters of overlap between chunks. Treat that as a prototype setting, not a universal optimum. Test chunk size and overlap against the questions your team actually asks.

The loader also attaches a date value derived from the transcript file name. In a stronger production design, add more metadata, such as:

- meeting ID;
- meeting title;
- date and time;
- project or customer;
- participants, where policy allows;
- source file ID; and
- access-control scope.

Good metadata improves filtering and source traceability.

## 11. Configure the Slack retrieval agent

The retrieval path is:

```text
Slack Trigger
  -> If
  -> AI Agent
     -> Gemini chat model
     -> Pinecone retrieval tool
     -> optional Slack forwarding tool
  -> Send a message
```

The agent's primary job is to answer questions about past meetings with retrieved context from Pinecone.

Example test questions:

```text
What did we decide about the onboarding flow?
```

```text
Who owns the checklist update?
```

```text
What deadline was agreed in the last onboarding meeting?
```

Use synthetic transcripts for the first retrieval tests.

## 12. Test the system in three stages

### Stage A — intake and storage

Confirm that one webhook request creates one transcript file in Google Drive.

### Stage B — indexing

Confirm that the new Drive file triggers the ingestion branch and creates vectors in Pinecone.

### Stage C — retrieval

Ask a Slack question whose answer exists in the synthetic transcript. Confirm that the reply is grounded in the indexed meeting content.

Do not activate the full workflow until all three stages pass.

## 13. Production hardening

The portfolio workflow demonstrates the core architecture. A production deployment should add stronger controls.

### Authenticate the intake webhook

Do not expose an unauthenticated transcript-ingestion endpoint to the public internet. Add an API key, signed request, gateway, or another appropriate authentication layer.

### Add idempotency

A Drive trigger or client retry can submit the same transcript more than once. Store a meeting ID or source-file ID and reject duplicate ingestion.

### Add source citations

Return the meeting title, date, and source file with each answer where possible. A retrieval answer is more useful when a user can verify the source.

### Add access control

Do not allow one Slack user to retrieve meeting content that they could not normally access. Separate indexes or namespaces by team, customer, or permission boundary when required.

### Define retention and deletion

Meeting transcripts can contain sensitive business and personal information. Define who can record, who can search, how long data is stored, and how deletion requests propagate through Drive and Pinecone.

### Add evaluation

Create a small test set of real business questions and expected answers. Measure retrieval quality before you depend on the assistant for important decisions.

### Add observability

Log ingestion failures, embedding failures, Pinecone write failures, Slack delivery failures, and retrieval errors. Add an error workflow or alert channel.

## 14. Activation checklist

Before activation, confirm all of the following:

- [ ] The webhook accepts only approved requests.
- [ ] The transcript payload uses `body.transcript`.
- [ ] Google Drive writes to the correct folder.
- [ ] Gemini creates the expected summary format.
- [ ] Slack posts only to the intended channel.
- [ ] The Drive trigger detects a new transcript.
- [ ] Pinecone receives the transcript vectors.
- [ ] Slack questions retrieve the expected meeting context.
- [ ] Bot messages do not create response loops.
- [ ] No secret or private resource ID is stored in the public repository.
- [ ] The organization has a clear recording, access, and retention policy.

## 15. Current implementation note

The screenshots show a custom browser meeting-capture prototype and a successful “Transcript sent to n8n” state. The source code for that capture client was not present in the archive used to prepare this public repository. The n8n backend is therefore the reproducible part of this release.
