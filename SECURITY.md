# Security and Privacy Notes

Meeting transcripts can contain confidential company information and personal data. Treat this workflow as a sensitive-data system.

## Do not publish

Do not commit:

- API keys or OAuth tokens;
- Slack workspace or private channel configuration;
- live webhook secrets;
- private Google Drive IDs when they reveal internal resources;
- real meeting transcripts;
- participant names or contact details without a clear reason and permission; or
- production Pinecone credentials.

## Deployment controls

For a production deployment:

1. authenticate transcript ingestion;
2. restrict Slack and Drive permissions to the minimum required scope;
3. enforce access boundaries during retrieval;
4. define retention and deletion rules;
5. log security-sensitive actions;
6. test duplicate-ingestion behavior; and
7. ensure participants know when meeting content is recorded or processed under your organization's policy.

The workflow in this repository is a sanitized portfolio implementation. It is not a security certification or a claim of compliance with a specific regulatory framework.
