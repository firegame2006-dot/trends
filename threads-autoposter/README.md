# Threads autoresponder

Goes through the comments on recent posts once an hour, answers the ones worth
answering and stays quiet on the rest.

There's no database behind it. `GET /{post-id}/conversation` returns the whole
thread flat, and every entry carries `replied_to` and `is_reply_owned_by_me`, so
a comment counts as handled once the thread contains a reply of ours pointing at
it. Our own entries, hidden comments and empty text get dropped before the model
sees anything.

The one piece of local state is the SKIP list: comment ids the model decided not
to answer, kept in workflow static data so you're not paying to re-read the same
spam every hour. Capped at 2000, pruned oldest first. If it vanishes nothing
breaks, you just spend a few cents deciding again.

Two things to know before trusting it:

- static data isn't persisted on manual runs, so SKIP only sticks on scheduled
  executions
- a reply published seconds ago may not be in `/conversation` yet, so running
  this by hand several times in a row can produce a duplicate reply. Once an
  hour leaves enough room

`Generate reply` fires once per comment and `Extract reply` pairs the responses
back to their comments by index. Don't put anything between those two that
reorders or drops items.

## Setup

Import, then put the Query Auth credential into the four `graph.threads.com`
nodes and the Header Auth one into `Generate reply`. Set the timezone under the
three dots, Settings, Timezone, and paste in the same playbook the poster uses.

Before activating, run `Extract reply` with Execute step and read the `reply`
column. The three nodes after it post for real. The first scheduled run answers
everything it finds up to `maxRepliesPerRun`, so on an account with a backlog,
turn that down first.

## Config fields

- `playbook`, the same text the poster uses
- `model`, claude-sonnet-5-5. One call per comment, so cost tracks volume
- `maxReplyChars`, 180. A comment reply is a sentence or two
- `maxRepliesPerRun`, 25, oldest comments first
- `replyWindowDays`, 14, only posts newer than this get scanned
- `scanPosts`, 25

## API

Same app as the poster, plus the scopes `threads_manage_replies` and
`threads_read_replies`. 1000 replies per 24h.
