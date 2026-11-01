# Threads automation

Two n8n workflows for running a Threads account:

- `threads-autoposter` posts twice a day
- `threads-autoresponder` answers comments every hour

They're independent. Import either on its own. The only thing they share is the
playbook text you paste into both.

You need n8n, a Meta app with the Threads use case (and a long lived token off
it), and an Anthropic API key.

Neither workflow keeps a database. Post history, engagement and which comments
were already answered all come back from the Threads API on every run.