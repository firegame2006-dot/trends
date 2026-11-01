# Threads automation

Two n8n workflows for running a Threads account:

- `threads-autoposter` posts twice a day
- `threads-autoresponder` answers comments every hour

They're independent. Import either on its own. The only thing they share is the
playbook text you paste into both.

You need n8n, a Meta app with the Threads use case (and a long lived token off
it), and an Anthropic API key.

## Credentials

No keys in the JSON. Both workflows use n8n generic credentials:

- Threads: **Query Auth**, name `access_token`
- Anthropic: **Header Auth**, name `x-api-key`

The "Name" field is the actual header or query parameter the API expects, not a
label you pick. Getting that wrong costs an evening, ask me how I know.

Neither workflow keeps a database. Post history, engagement and which comments
were already answered all come back from the Threads API on every run.
