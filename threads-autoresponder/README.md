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

