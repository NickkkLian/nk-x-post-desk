# X Post Desk

![The page built from the bundled example, partly used: "3 posts · 1 posted", the first card closed and marked Posted, the second card open with its post text, "168 / 280" and an "Open in X with this text" button, then the first reply with a post link pasted into its field, the line "Got it. This reply will go under post 1234567890123456789." and a live "Open the reply in X" button](docs/desk.png)

Turn the posts and threads your agent drafted into one page of prefilled X composer links: replies chained in order,
every part counted the way X counts, and you press Post. No account to connect, no key.

It also tells you what X will do to your text before you open X: `main.py` becomes a link there and counts 23
characters, not 7, because `.py` is a top-level domain (Paraguay's).

An agent skill (`nk-x-post-desk`) with one Python script. The picture above is `demo-desk.html`, built by the `--demo`
command below from the example in this repository, after the first post was marked as posted and a post link was
pasted into the second post's reply field. The example's tool and posts are invented, and so is the pasted link.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills): 14 agent skills that work with numbers, check an agent's work, guard a repository, and write and present.

## File names X turns into links, and the ones it leaves

| You type | In X's composer | It counts as | Why, as far as it is known |
|---|---|---|---|
| `main.py` | **a link** | 23, not 7 | `.py` is a top-level domain (Paraguay's) |
| `install.sh` | **a link** | 23, not 10 | `.sh` is Saint Helena's |
| `lib.rs` | **a link** | 23, not 6 | `.rs` is Serbia's |
| `main.tf` | **a link** | 23, not 7 | `.tf` is the French Southern Territories' |
| `script.pl` | **a link** | 23, not 9 | `.pl` is Poland's |
| `Main.java` | **a link** | 23, not 9 | `.java` is a brand domain (Oracle's) |
| `backup.zip` | **a link** | 23, not 10 | `.zip` is a generic domain, like `.com` |
| `index.js` | plain text | 8 | `.js` is not a top-level domain |
| `package.json` | plain text | 12 | `.json` is not one |
| `main.go` | plain text | 7 | `.go` is not one |
| `my_script.py` | plain text | 12 | the underscore: X does not link a name that has one before the ending (one name seen; the rule is the library's) |
| `CLAUDE.md` | plain text | 9 | `.md` is a top-level domain (Moldova's) that X has left as plain text since spring 2026 |
| `README.md` | plain text | 9 | the same |

`scripts/build_desk.py` gives the same answer as X's web composer for all 13 names, and its self-test holds this
table. The second column is what the composer showed on 2026-10-10, when these 13 names were pasted as one line and
nothing was posted: seven turned into links and six stayed text. The third column is the script's count. That a link counts 23 whatever
its length is X's published rule; the composer's own number was not seen, because its counter is a ring without a
figure. The country and brand names are from the public list of top-level domains.

On `.md`, X's site and X's published rule differ. X's open-source
[twitter-text](https://github.com/twitter/twitter-text) library still lists `.md` as a top-level domain, so by the
library a bare `.md` name would be a link. In 247 public posts read on 2026-10-08, a bare `.md` name was a link in
the 4 posts that had one up to 2026-02-20, and plain text in the 13 posts that had one from 2026-04-06 on (in one
of the 13, a Japanese post, the names run into the text around them). X's library has not been changed to match, and
the author has seen no statement from X; it can change again. Not seen either way: an address written with
`https://` that ends in `.md`; the script still counts that one as a link, as the library does.

To check again in a minute: paste a line of file names into X's composer, look at which ones turn blue, and close it
without posting.

## Try it

Clone and run the example (Python 3.9 or newer, standard library only; nothing is installed, no account, no key):

```bash
git clone https://github.com/NickkkLian/nk-x-post-desk && cd nk-x-post-desk
python3 scripts/build_desk.py --demo
open demo-desk.html
```

(`open` is macOS; elsewhere, open the file in any browser.) The demo prints:

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

A desk of your own is a folder per post. This root post mentions two file names:

```bash
mkdir -p posts/2026-11-03-hello
printf 'run main.py first, the notes are in CLAUDE.md\n=====\nthis reply is fine\n' > posts/2026-11-03-hello/post.md
python3 scripts/build_desk.py posts
```

```text
✘ P02 2026-11-03-hello/post.md root: X turns 'main.py' into a link (a name that ends in a real top-level domain; it counts 23). Reword it or move it to a reply, or allow root links: --allow-root-link, or "allowRootLink": true in desk.json
1 post, 2 parts, 1 problem
nothing written
```

`CLAUDE.md` is not named in the
refusal: X leaves it as text (the table above). The build exits 1 and writes no page until the folder is clean.

A link in the root post is refused by default because many people keep the link in a reply. If you want it in the
root, run the build with `--allow-root-link` or put `"allowRootLink": true` in `desk.json`. In a reply nothing is
refused: a file name there gets a one-line notice, and the page is built.

## What it does

- **Reads a folder an agent can write without help.** One folder per post, named `YYYY-MM-DD-name`; inside it
  `post.md` (the root post, then each reply, separated by a line of `=====`), at most one picture, and an optional
  `alt.txt`. The whole format and every refusal: [references/format.md](references/format.md).
- **Counts the way X counts.** 280 per part; any link or bare domain counts 23 (the script carries X's list of 1,419
  top-level domains); Chinese, Japanese and Korean characters, arrows and emoji count 2. The rule was read from X's
  open-source [twitter-text](https://github.com/twitter/twitter-text) library, with one exception taken from X's
  site itself: a bare name ending in `.md` is counted as plain text (the table above).
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
and tells you where `desk.html` is. [SKILL.md](SKILL.md) is what it reads. How it went in a Claude Code and a Codex session is under [Verify](#verify).

## Next to Buffer and Typefully

Buffer and Typefully are schedulers: you connect your X account, and they publish your posts for you at the time you
choose. If you want posts to go out while you are away, use one of them.

Typefully also has an [MCP server](https://support.typefully.com/en/articles/13128440-typefully-mcp-server): its
help page says an AI tool such as Claude Code can "create, edit, schedule, and manage drafts" in your Typefully
account once you approve the connection. If you are fine connecting an account, that route already exists.

X Post Desk is for when an agent has drafted a thread into files and you want to post it yourself. It connects nothing, stores nothing outside your browser,
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

The self-test ends on `build_desk selftest · 42/42 passed`. Its 42 checks: 21 sample texts against the count by X's published rule (not a number X showed), a planted mistake for
every refusal, 16 pasted links of which only the real post links may become a reply, a desk whose text tries to be
markup, 10 planted outside references that the build's own audit must see, the two ways to allow a root link, the
notice for a file name in a reply, and three added in 0.1.1: the 13 names as X's composer showed them on 2026-10-10,
the `.md` rule and the underscore rule. Before the first release, 54 deliberate breaks were made to the script one
at a time in a sandbox copy (a part over 280 let through, the root's link check off, the link pattern without its
anchors, the text written into the page unescaped, the audit switched off, the notice turned into a refusal, and so
on). Each one turned the self-test red on the check that guards it, none by crashing; the unbroken copy stayed
green. For 0.1.1 the two file-name fixes were each switched off in a copy, and the three new checks went red for
them.

Tested so far: the script, in a fresh clone with an empty home folder (Python 3.13), and its self-test on Python 3.9
as well. The skill was run once from start to finish inside a Claude Code session, on 2026-10-01, in a project
that held one notes file, with no other skills or settings: asked for a launch post with the link in a reply and a
second post, the agent loaded the skill, wrote two post folders, was refused once for a reply that counted 290,
shortened it, and built `desk.html`. On 2026-10-01 the same request was run once in an OpenAI Codex session
(CLI 0.159.2, a clean environment): Codex loaded the skill, wrote two post folders and built the page with 0 problems;
no refusal came up in that run, so the refusal path was not seen under Codex.

## License

MIT. Not affiliated with X. Read a script before letting it run in your environment.
