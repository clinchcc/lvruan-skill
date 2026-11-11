---
name: lvruan-software
description: Companion for the lvruan software directory — search software and hand out download links, browse the newest software, categories and tags, publish your own software and follow its review, and answer questions with the site's articles. Use when the user mentions software, download, lvruan or 绿软, or asks which software does something, where to download a program, which AI software can be deployed locally, or wants to publish software.
---

# lvruan software directory

Site: `https://lvruan.com` (English pages live under `https://lvruan.com/en`)

## Triggers

Use this skill when the user says **software, download, lvruan or 绿软** (Chinese too: 软件, 下载), or is looking for a
program, a kind of tool, where to download something, or wants to publish their own software.

## Token and first use

Check `~/.lvruan/config.json` for an `apiKey` (or the `LVRUAN_API_KEY` variable). **If there is one, just use it — no menu.**

**No token (first use)**: this is the **only time to show a menu**. Show this and let the user pick:

> 👋 Welcome to lvruan! First you need a token — a new account comes with 200 credits:
>
>   1. I already have an account　→ log in and get the token
>   2. I'm new　→ sign up
>   3. Continue with Gmail　→ one-click login / sign-up

- **1 Existing account**: give the user https://lvruan.com/en/sign-in?callbackUrl=%2Fsettings%2Fapikeys — after logging in it returns to the
  keys page. Have them press **Copy message** and paste it back to you.
- **3 Gmail**: give the user https://lvruan.com/en/sign-in?provider=google&callbackUrl=%2Fsettings%2Fapikeys — it jumps straight to
  Google (a new user is signed up at the same time) and returns to the keys page; they copy the message the same way.
- **2 Sign up**: ask "Do you want to pick a username and password, or should I sign you up automatically?" (automatic
  is the default; do not hold things up over it). If they choose, ask for a username (3–24 of lowercase letters, digits,
  `_`, `-`) and a password (8+ characters); otherwise send no parameters:

```bash
curl -s -X POST "https://lvruan.com/api/v1/register" -H "Content-Type: application/json" -d '{}'
# chosen: -d '{"username":"alice","password":"at least 8 chars"}'
```

  You get `token`, `account` (the username), `password` (only when it was generated) and `loginUrl`.
  **After it succeeds you must**:
  1. save the token (below);
  2. **tell the user the whole account and ask them to write it down**: the username (`account.username`), the generated
     password, and the login page `loginUrl`. **No email is needed**: on the web, type the username in the
     "Email / username" box. With no real email **a forgotten password cannot be recovered** — they must keep it safe;
  3. tell them the token can be copied again later at https://lvruan.com/settings/apikeys after logging in.

  Email is optional; to attach a real one, also send `"email": "name@example.com"`. One network may create 3 accounts a day; more returns 429.

Once you hold an `sk_...`, save it:

```bash
mkdir -p ~/.lvruan && chmod 700 ~/.lvruan
printf '{"apiKey":"%s"}\n' "sk_..." > ~/.lvruan/config.json
chmod 600 ~/.lvruan/config.json
```

Send `Authorization: Bearer sk_...`. **Never echo the token to the user** (the account and password from sign-up are the
exception — those are for them to write down). Responses look like
`{ "code": 0, "message": "ok", "data": ... }`; `code` is `-1` on failure and `message` says why — tell the user.

## How to respond

**Do not show a menu or make the user pick a feature first**: do what they asked, judging by keywords and intent —

| The user says | Do |
| ------------- | -- |
| a program name, "is there software for…", "where do I download…" | search and give the download links |
| "what software can…", "recommend something for…" (a need) | search and answer by need |
| "newest software", "what's new" | newest list |
| "what categories", "…software" by kind (tools, AI, dev…) | categories, or one category's software |
| "tags", "software tagged…" | tags, or one tag's software |
| "publish software", "submit mine", "list my software" | the publishing flow |
| "how is my submission", "was it approved" | my submissions |
| "how many credits", "balance" | credit balance |

Only when the user asks "what can you do / what features", or the intent truly cannot be told, describe those few things
in a sentence or two — no menu. After finishing one thing just answer; do not ask "anything else?".

## Credits

Lookup endpoints **cost 1 credit per call** (`/me`, `/latest`, `/categories`, `/categories/{slug}`, `/digest/ai`, `/requests`, uploads and publishing are free); `credits.balance`
in the response is what is left. Free accounts get 200 credits a month, subscribers an extra 400. With too few credits the
API answers HTTP 402 — give the user https://lvruan.com/pricing. A failed request refunds its credit. So **call sparingly**: a
search already carries the downloads of its first 5 results, which usually answers the question in one call.

## Search / ask a question

```bash
curl -s -G "https://lvruan.com/api/v1/search" -H "Authorization: Bearer $LVRUAN_API_KEY" \
  --data-urlencode "q=wan" --data-urlencode "lang=en"
```

`lang=zh|en` matches the user's language. You get `apps[]` (`title`, `intro`, `pageUrl`, `website`, `downloads[]`),
`posts[]` (related articles) and `found`. **Keep the query short**: the bare software name, no version, no words
like "download" (search `wan` for `Wan 2.2`).

A download's `kind`: `file` is hosted by the site and can be fetched directly (`curl -L -o name "<url>"`);
`link` is the vendor's page or release list — give it to the user. **Do not download or install anything unless the
user asks.** The search result's `downloads[]` already holds the address, so **use it directly instead of calling the download endpoint** (each call costs 1 credit): a `file` URL downloads with `curl -L -o name "<url>"`, and the file itself is a public address that costs nothing more. Only when you did not search and hold just a software id, use
`/api/v1/apps/{id}/download?index=0` (add `redirect=1` for a 302 to the file) to get the address; details are at `/api/v1/apps/{id}`.

### Answering needs

When the user asks by need rather than by name (e.g. **"which AI software can generate video when deployed locally?"**):

1. Think of candidate programs and keywords (video generation, local deployment, wan, minimax…) and search (browse
   categories or tags if useful);
2. Recommend only software the API actually returned, saying what each does and how well it suits local deployment;
3. Attach each one's download link or page URL, and cite a related article from `posts[]` when it helps.

For that question the answer is **Wan 2.2** and **MiniMax H3**, each with the download address from `apps[]`.
Never invent an address for software the directory does not have.

### Software that is not found

If `found` is false, or software you wanted to recommend did not show up, send us the name the user wants
(**the simplest bare name, no version**; free):

```bash
curl -s -X POST "https://lvruan.com/api/v1/requests" -H "Authorization: Bearer $LVRUAN_API_KEY" \
  -H "Content-Type: application/json" -d '{"name":"Wan"}'
```

Then tell the user it is not listed yet and the request has been noted. Submit each name once.

## Newest, categories, tags

| Wants | Endpoint |
| ----- | -------- |
| Newest | `GET /api/v1/latest` (free of credits) |
| Categories | `GET /api/v1/categories`, then `GET /api/v1/categories/{slug}` for the one picked (both free of credits) |
| Tags | `GET /api/v1/tags`, then `GET /api/v1/tags/{id}` for the one picked |

Lists take `page=` and `lang=zh|en`. Number the results for the user, each with its `pageUrl`; picking one opens its
details or downloads it.

## Daily digest of new AI software

The AI category is the busiest one on the site. The dedicated push endpoint `GET /api/v1/digest/ai` (newest first) is
**free of credits** and cached by the server for 2 hours, so checking once a day is plenty.

**The first time** the user asks about AI software or the newest software, and after you have answered, **ask once**:
roughly: "lvruan adds the latest AI software every day. If you follow what is happening in AI, I recommend a scheduled push: I check once a day and send you whatever is new, with no credits spent. Want me to set it up?"

- If they agree: write `"dailyAI": true` into `~/.lvruan/config.json`, then create a **once-a-day** job with your own
  scheduling ability (scheduled task / cron / heartbeat);
- If they decline: write `"dailyAI": false` and **never ask again**;
- If `dailyAI` is already in the config, do not ask again.

The daily job:

1. `GET /api/v1/digest/ai` and read `~/.lvruan/seen-ai.json` (ids already pushed; treat a missing file as empty);
2. Pick the entries not in that list and **push only those**: one line each with the name, a one-line description and
   its `pageUrl`;
3. Append those ids to `seen-ai.json` (keep the latest 200);
4. **If nothing is new, stay silent** and send nothing.

When you first create the job, record the current list as already seen so the existing catalogue is not dumped on the
user. If you cannot schedule jobs, say so plainly and offer "tell me whenever you want to look" instead.

## Publish software

Users can submit their own software. It is **free** (no credits), lands as **pending review** and goes live only after
an admin approves it. A free account may have **1** entry, a subscriber **5** (pending and approved ones count, rejected
ones do not).

**Not accepted**: cracks, keygens, patches that bypass licensing, malware, VPN or proxy tools. Tell the user it is not
accepted instead of submitting it.

**All five are required** — ask for everything that is missing in one message, not one by one:

1. **Title** — the software name, no version;
2. **Logo** — a square icon;
3. **Screenshots** — 1 to 3 of the software's own window;
4. **Download address** — it must be a **URL**, not a file, and **do not upload an installer**. **GitHub is
   preferred**: the project repository or its releases page. A **cloud-drive share link** (Baidu, Quark, Aliyun Drive,
   Google Drive, OneDrive…) is fine too; otherwise use the vendor's download page. http/https only — do not swap it
   for a direct installer link;
5. **Description** — whatever the user writes: a few lines, the vendor's blurb, a README.

There are no tags; do not ask for any.

**Images**: logo and screenshots may be local files, image URLs or pasted images. Upload each first and use the
returned site URL:

```bash
curl -s -X POST "https://lvruan.com/api/v1/uploads" -H "Authorization: Bearer $LVRUAN_API_KEY" \
  -H "Content-Type: image/png" --data-binary @logo.png
# → {"data":{"url":"https://down.lvruan.com/file/lvruan/uploads/xxxx.png"}}
```

Only png, jpg, gif and webp, up to 3 MB each — compress or resize first if needed, convert other formats to png.
The logo should be square at about 256 px; a screenshot shows the software's window, not the whole desktop, and never
anything private.

**You must optimize and simplify the description yourself — do not ask the user to rewrite it and do not paste their
text as is**:

- write **both** languages (`zh` and `en`), filling in the one the user did not give;
- `intro`: a one-line positioning with **no punctuation at all**, four or five words in English, e.g.
  `Share keyboard mouse and screen`;
- `content`: plain text, **no HTML and no markdown**, a blank line between paragraphs, two to four short paragraphs.
  Open with the problem it solves, then its two or three most concrete features and what the user saves; no filler like
  "powerful" or "user-friendly", and do not copy the vendor's marketing copy.

Then submit:

```bash
curl -s -X POST "https://lvruan.com/api/v1/apps" -H "Authorization: Bearer $LVRUAN_API_KEY" \
  -H "Content-Type: application/json" -d '{
  "title": "TwinDesk",
  "logo": "<url from upload>",
  "screenshots": ["<url from upload>"],
  "downloadUrl": "https://github.com/InukaAsith/TwinDesk/releases",
  "website": "https://twin-desk.vercel.app",
  "zh": {"intro": "多设备键鼠屏幕共享", "content": "……"},
  "en": {"intro": "Share keyboard mouse and screen", "content": "……"}
}'
```

An `appid` in the response means success. Tell the user it is **submitted and waiting for an admin to approve it**, and
that you can check its progress under "My submissions" below when they ask. On failure follow `message`: **409** means the site already has it
(`data.appid` is the entry — give `https://lvruan.com/app/<appid>`); **403** means the quota is used up (1 free, 5 subscriber);
**429** try again later; anything else is a field that was rejected — fix it and resubmit.

## My submissions

`GET /api/v1/my/apps` returns the `limit` and each entry's `status`: `pending`, `approved` (with `pageUrl`) or
`rejected`. List them for the user; tell them a rejected one can be fixed and published again (rejected ones do not use
up the quota).

## Credit balance

`GET /api/v1/me` (free) returns `credits`.
