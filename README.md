# X Post Desk

![The page built from the bundled example, partly used: "3 posts · 1 posted", the first card closed and marked Posted, the second card open with its post text, "168 / 280" and an "Open in X with this text" button, then the first reply with a post link pasted into its field, the line "Got it. This reply will go under post 1234567890123456789." and a live "Open the reply in X" button](docs/desk.png)

Buffer and Typefully post for you through a connected account; X Post Desk connects nothing: it turns the drafts your
agent wrote into a page of prefilled X composer links, and you press Post.

An agent skill (`nk-x-post-desk`) with one Python script. The picture above is `demo-desk.html`, built by the `--demo`
command below from the example in this repository, after the first post was marked as posted and a post link was
pasted into the second post's reply field. The example's tool and posts are invented, and so is the pasted link.

## Try it

Nothing is installed, no account, no key. Python 3.9 or newer, standard library only.

```bash
git clone https://github.com/NickkkLian/nk-x-post-desk && cd nk-x-post-desk
python3 scripts/build_desk.py --selftest
python3 scripts/build_desk.py --demo
open demo-desk.html
```

(`open` is macOS; elsewhere, open the file in any browser.) The self-test ends on `build_desk selftest · 39/39 passed`,
and the demo prints:

```text
✔ 2026-11-03-walkposter-launch · root 201 · reply 1 171 · picture poster.png
✔ 2026-11-06-climb-numbers · root 168 · reply 1 152 · reply 2 62
✔ 2026-11-10-print-sizes · root 164 · reply 1 62
3 posts, 7 parts, 0 problems
written demo-desk.html · 3 posts · 1 picture embedded · 97 KB · 0 references outside the page
```

The numbers after `root` and `reply` are each part's length as X counts it. On the page:

1. Save the picture from the card (a link cannot attach one).
2. Press **Open in X with this text**. X's composer opens with the post typed in. Add the picture, press Post.
3. Paste the link of the post you just published into the reply's field. **Open the reply in X** now opens the next
   part as a reply to it.
4. Mark the card as posted. The mark stays in that browser.

### A mistake it catches

A desk of your own is a folder per post. This root post mentions a file name:

```bash
mkdir -p posts/2026-11-03-hello
printf 'read the notes in CLAUDE.md first\n=====\nthis reply is fine\n' > posts/2026-11-03-hello/post.md
python3 scripts/build_desk.py posts
```

```text
✘ P02 2026-11-03-hello/post.md root: X turns 'CLAUDE.md' into a link (a name that ends in a real top-level domain; it counts 23). Reword it or move it to a reply, or allow root links: --allow-root-link, or "allowRootLink": true in desk.json
1 post, 2 parts, 1 problem
nothing written
```

`.md` is Moldova's top-level domain, so on X the name `CLAUDE.md` becomes a live link, and it counts as 23
characters whatever its length. The same goes for `main.py` and `run.sh`. The build exits 1 and writes no page until
the folder is clean.

A link in the root post is refused by default because many people keep the link in a reply. If you want it in the
root, run the build with `--allow-root-link` or put `"allowRootLink": true` in `desk.json`. In a reply nothing is
refused: a file name there gets a one-line notice, and the page is built.

## What it does

- **Reads a folder an agent can write without help.** One folder per post, named `YYYY-MM-DD-name`; inside it
  `post.md` (the root post, then each reply, separated by a line of `=====`), at most one picture, and an optional
  `alt.txt`. The whole format and every refusal: [references/format.md](references/format.md).
- **Counts the way X counts.** 280 per part; any link or bare domain counts 23 (the script carries X's list of 1,419
  top-level domains); Chinese, Japanese and Korean characters, arrows and emoji count 2. The rule was read from X's
  open-source [twitter-text](https://github.com/twitter/twitter-text) library.
- **Refuses before it builds.** A part over 280, a link in the root post (unless you allow it), a wrongly named
  folder, an empty part, two pictures, a picture that is not what its name says. Each refusal names the folder, the
  part and the fix.
- **Writes one file.** `desk.html` has the pictures inside it and works offline from a local file, in light and dark,
  down to a 390 px phone screen. Its Content-Security-Policy names its one script and one style sheet by sha256 and
  allows nothing else, so the browser itself blocks any request to another host. The build reads its own output back
  and will not write a page that refers to anything outside itself.
- **Uses X's documented link, and nothing else of X.** The buttons are plain links to
  `https://x.com/intent/tweet` with `text` and, for a reply, `in_reply_to`. A reply button has no link until you
  paste the link of the post it answers, and only a link shaped like `x.com/<name>/status/<digits>` is accepted.
- **Treats post text as text.** A draft that contains `<script>` is shown with the angle brackets, not run.

## With an agent

Install the folder as a skill (below), then ask in plain words, for example: *"Draft a launch post for this tool with
the link in a reply. I'll post it myself."* The agent drafts into the folder format, runs the check, builds the page
and tells you where `desk.html` is. [SKILL.md](SKILL.md) is what it reads.

Tested so far: the script, in a fresh clone with an empty home folder (Python 3.13), and its self-test on Python 3.9
as well. The skill was run once from start to finish inside a Claude Code session, on 2026-10-01, in a project
that held one notes file, with no other skills or settings: asked for a launch post with the link in a reply and a
second post, the agent loaded the skill, wrote two post folders, was refused once for a reply that counted 290,
shortened it, and built `desk.html`. It has not been run inside a Codex session.

## Next to Buffer and Typefully

Buffer and Typefully are schedulers: you connect your X account, and they publish your posts for you at the time you
choose. If you want posts to go out while you are away, use one of them. This does not compete with that.

Typefully also has an [MCP server](https://support.typefully.com/en/articles/13128440-typefully-mcp-server): its
help page says an AI tool such as Claude Code can "create, edit, schedule, and manage drafts" in your Typefully
account once you approve the connection. If you are fine connecting an account, that route already exists.

X Post Desk is for when nothing should be connected: an agent has drafted a thread into files, and you want to post
it yourself without connecting your account to anything. It connects nothing, stores nothing outside your browser,
and has no plan, cap or sign-up. What you give up is everything a scheduler does.

## What it does not do

- **Schedule.** The date in a folder's name orders the cards. Nothing happens on that date.
- **Post.** Every button opens X's composer; you press Post.
- **Attach the picture.** X's composer link has no parameter for media. The page shows the picture and you add it.
- **Know that a post went out.** "Posted" is a mark you set yourself.
- **Long posts.** The limit is 280 as X counts it. X Premium's longer posts are not supported: a part over 280 is
  refused.
- **Count emoji sequences exactly.** A flag, a family or a skin-tone emoji is counted per code point here and as 2
  on X, so the number shown can be higher than X's. It is never lower.
- **Sync between devices.** The page is a local file, and its marks live in one browser. To use it on a phone you
  have to get the file there yourself.

Also not known: whether the reply link opens as a reply inside X's phone app. The same link form was tried by the
author in a desktop browser, where it opened as a reply. If X opens a new post instead, the page says what to do:
open your post, press Reply, paste the copied text.

## Install

As a Claude Code skill:

```bash
git clone https://github.com/NickkkLian/nk-x-post-desk ~/.claude/skills/nk-x-post-desk
```

For OpenAI Codex (and other agents that read `~/.agents/skills`):

```bash
git clone https://github.com/NickkkLian/nk-x-post-desk ~/.agents/skills/nk-x-post-desk
```

Start a new session afterwards; skills load when a session starts. Or skip the agent and run
`scripts/build_desk.py` yourself: it needs nothing but Python.

## Verify

```bash
python3 scripts/build_desk.py --selftest
```

39 checks: 21 sample texts against X's count, a planted mistake for every refusal, 16 pasted links of which only the
real post links may become a reply, a desk whose text tries to be markup, 10 planted outside references that the
build's own audit must see, the two ways to allow a root link, and the notice for a file name in a reply. Before
release, 54 deliberate breaks were made to the script one at a time in a sandbox copy (a part over 280 let through,
the root's link check off, the link pattern without its anchors, the text written into the page unescaped, the
audit switched off, the notice turned into a refusal, and so on). Each one turned the self-test red on the check that
guards it, none by crashing; the unbroken copy stayed green.

## License

MIT. Not affiliated with X. Read a script before letting it run in your environment.
