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

