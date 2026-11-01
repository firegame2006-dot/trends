# Threads autoposter

Publishes two posts a day. Each one is written with the account's playbook, the
last dozen posts and their like and reply counts in the prompt, so the feed
keeps one line of thought instead of drifting into unrelated updates.

## What goes into a post

The playbook in the `Config` node does most of the work. It's the system prompt
and it's the only part really worth your time; everything else is context.

That context is the last 12 posts (so it doesn't repeat a topic or reuse an
opening line), their insights, and one line from `postTypes`. The three posts
with the most engagement get shown separately as examples of *form*: how the
first line is built, how long it runs, what kind of question closes it. Not
topic, form. Replies count triple a like when ranking them, since a reply means
the post actually landed.

`postTypes` rotates. Which line you get is derived from the clock,
`Date.now() / 12h` modulo the list length, rather than from a stored counter.
n8n doesn't persist workflow static data on manual runs, so a counter would
break exactly where you'd be testing it. Six types, two posts a day, so the
list comes round every three days.

The list that ships alternates hook and offer. A hook raises something the
reader recognises and sells nothing. The next post picks up that same thing and
closes with one line pointing at the profile link. Want to sell less often, add
more hooks between the offers.

## Flow

Schedule, Config, fetch last posts, split them, fetch insights, build prompt,
Claude, clean the text, create container, wait, publish, fetch permalink.

Publishing is two calls: you create a container, then publish it by its
`creation_id`. The 10 second wait between them isn't optional, the container
isn't ready immediately. Insights and permalink are both set to continue on
error, so a missing metric can't block a post.

## Setup

Import `workflow.json`, then:

1. Query Auth credential into the five `graph.threads.com` nodes
2. Header Auth credential into `Generate post`
3. Timezone under the three dots, Settings, Timezone. Import doesn't carry it
4. Rewrite `playbook` and `postTypes` in `Config`
5. Run `Extract text` with Execute step a few times and read what comes out
   before you activate anything

Edit the playbook inside the field rather than pasting the whole thing in. The
editor collapses newlines on paste and `postTypes` is parsed line by line, so a
paste quietly turns six post types into one very long one.

## Config fields

`playbook` and `postTypes` as above.

`model`, claude-opus-5-5.

`historyPosts`, 12, how much history goes into the prompt.

`maxPostChars`, 280. Threads allows 500. The model reads this as a target
rather than a ceiling, so lowering it on its own won't shorten posts by much;
the post shape section of the playbook has to say the same thing.

`replyControl`, one of everyone, accounts_you_follow, mentioned_only.

## API

Meta app with the Threads use case and the scopes `threads_basic`,
`threads_content_publish`, `threads_manage_insights`. Long lived tokens last 60
days and are renewed with
`GET /refresh_access_token?grant_type=th_refresh_token`.

250 posts per 24h, 500 characters each.
