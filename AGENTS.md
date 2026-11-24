# ClinicOS

## Start and verify

- Start the demo stack with `docker compose up -d`. Read [local setup](docs/local-setup.md) before applying migrations, mapping demo accounts, or configuring n8n credentials.
- Run Care Board with `python -B staff/server.py` at `127.0.0.1:8094`. Set `CLINICOS_SUPABASE_URL` and `CLINICOS_PUBLISHABLE_KEY` in the same shell. Use separate browser profiles for the demo admin and doctor.
- Verify changed handoffs with the [fictional demo](README.md#fictional-demo). For billing changes, check that a finalized general consultation yields a 350 EGP invoice and one 350 EGP payment leaves a zero balance. Test replay with the same payment request ID.
- For static checks, run `docker compose config -q` on Compose changes and `node --check staff/staff.js` on Care Board JavaScript changes.

## Workflow, data, and access

- Edit the shared n8n export at `workflows/clinicos-patient-to-payment.json`. Generate the instance-specific import with `python -B deploy/configure-workflow.py .local/workflow-config.json .local/clinicos-import.json`. Keep the generated import and credential IDs in the ignored `.local/` directory.
- Keep credentials in n8n or local environment variables. Use the publishable Supabase key for Care Board and preserve its authenticated staff RPCs and role checks.
- Use `supabase/migrations/` for schema and access changes and `supabase/seed.sql` for fictional fixtures. Test in a dedicated demo project with a separate Telegram bot. Keep visit attachments in the private `visit-files` bucket.
- Keep the n8n editor local. The public tunnel targets the Caddy proxy in `deploy/Caddyfile` and forwards only the Telegram webhook. Booking listeners need a direct or session-pooled PostgreSQL connection.
- Interpret appointment dates and clinic-day billing reports in `Africa/Cairo`. Keep payment amounts as decimal strings at the API boundary and preserve database-side balance checks and idempotency.

## Summary drafting

- Start local drafting with `docker compose --profile summary up -d`. Pull the configured model with `docker compose exec ollama ollama pull qwen3:4b-instruct-2507-q4_K_M`.
- Edit the summary prompt in the n8n node **Draft summary with local Ollama** in the workflow export. Keep instructions in the system message and fictional visit notes in the user message.
- Check prompt or model changes against a fictional note with a missing fact. The draft must stay within the source notes. Keep doctor review and approval before PDF generation, and confirm the patient and filename before Telegram delivery.
