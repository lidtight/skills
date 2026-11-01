---
name: lidtight-check
description: Use when the user is about to launch, publish or share a web app, asks whether their app is secure or safe to go live, has added sign-in, payments, a database, file uploads or an AI feature, or wants a security review of a deployed web app, including apps built with Lovable, Bolt, Base44, Replit, Supabase or Firebase. Runs LidTight's free check on the user's own live app link. The check reads only what any visitor's browser can see and needs no key and no sign-up. Then explains each finding in plain words, most serious first, with what it means for the founder's money and customers. Do not use for an app the user does not own or control, or for code that is not live yet.
license: MIT
compatibility: Needs a shell with curl and internet access to reach lidtight.com. Without them, the skill sends the user to https://lidtight.com instead.
metadata:
  author: LidTight
  homepage: https://lidtight.com
  version: "1.0.0"
---

# LidTight check

LidTight's free check looks at a live web app the way a stranger does. It reads only what any visitor's browser can see. It needs no key and no sign-up. It takes from a few seconds to about a minute. It counts records and never keeps customer data. It answers five questions:

- Can someone spend your money with your own keys? (secret keys in the public code)
- Can a stranger download your customer list? (database tables and the app's own endpoints open to anyone)
- Can bots sign up without limits? (sign-up without email confirmation)
- Did secret files ship with your app? (`.env`, `.git`, database backups, settings files)
- Is your source code public? Are browser and email protections on?

## Rules

Follow these rules every time. They come before every step below.

1. Check only the user's own app, or an app the user controls. If you are not sure, ask. Without a clear yes, stop. If the user asks you to check someone else's app, say no and say why.
2. Send LidTight only the app's public link. Never send passwords, secret keys, tokens or user data. LidTight does not need them.
3. Do not attack the app. Do not try to exploit a finding or confirm it yourself. Do not read its database, sign up, upload files, call its AI feature or send it test requests. The LidTight check is the only thing that touches the live app. You may read the user's own source code.
4. Run the check once per change. Wait for the `done` event. Never start a second run while one is running. Never run the check in a loop. The limit is 5 checks per visitor per hour.
5. You cannot run the deep checks. This address never runs them. Do not ask for them on the user's behalf. Do not pay, enter card details or create accounts for the user.
6. Change the user's code only when the user asks.

## When to offer the check

Offer the check when the user:

- is about to launch, publish or share the app;
- asks whether the app is secure or safe to go live;
- has added sign-in, payments, a database, file uploads or an AI feature, and the new version is live;
- asks for a security review of a deployed web app.

The app must be live on the public internet. LidTight refuses private addresses such as `localhost`. If the app is not live yet, say that the check needs the live link, and wait.

## Step 1: Find the live link

Look for the app's public address before you ask the user. Check:

- the README;
- `homepage` in `package.json`;
- `index.html`: the `<link rel="canonical">` tag or the `og:url` tag;
- `public/robots.txt` (the `Sitemap:` line), `sitemap.xml` or a `CNAME` file;
- deploy settings that can name the domain, such as `vercel.json`, `netlify.toml`, `wrangler.toml`, `fly.toml` or `.replit`;
- `site_url` in `supabase/config.toml`, if it is not a local address.

Do not open `.env` files or other files that hold secrets just to find the link. Ask the user instead.

Use the address that visitors use, with `https://`. If you find nothing, ask: "What is the live link of your app?"

## Step 2: Confirm with the user

Show the link and ask one question. For example:

> I found `https://your-app.lovable.app`. Is this your app, or an app you control? If yes, I will run LidTight's free check on it. The check reads only what any visitor's browser can see.

If the user already said in this conversation that the app is theirs and asked for the check, do not ask again. Without a clear yes, stop.

## Step 3: Run the check

Run this one command. Put the user's link in place of `https://your-app.lovable.app`. Change nothing else.

```bash
curl -sN --max-time 180 -G "https://lidtight.com/api/scan" --data-urlencode "url=https://your-app.lovable.app"
```

This calls `https://lidtight.com/api/scan?url=<the link, URL-encoded>`. `curl` does the encoding. Do not add other parameters.

The answer streams while the check runs. Wait until the command ends. Do not start the command again while it runs.

If you have no shell, if `curl` is missing, or if the command cannot reach the internet, do not guess a result. Your tool may block network access. Then ask the user to approve this one command, or tell the user to open https://lidtight.com, paste the link and click "Check my app". Offer to explain what they see.

## Step 4: Read the events

The answer is a list of events. Each event is one line: `data: ` and then one JSON object. An empty line follows each event. Read every line in order. There are four types.

### `step`: progress

```text
data: {"type":"step","id":"secrets","label":"Looking for secret keys","status":"done","note":"none found"}
```

- `status` is `running`, `done` or `skipped`.
- `note` is a short result. Use it for context. For example, the `backend` step names the database: `"note":"Lovable Cloud (Supabase) · 2 tables · 1 server function"`.

### `finding`: one problem or fact

```text
data: {"type":"finding","finding":{"id":"locked-3","severity":"critical","title":"Anyone can read 12 records, with names, emails and phone numbers","detail":"Where it is, the proof and the fix are in the report.","evidence":["We counted them without downloading any."],"locked":true}}
```

- `severity` is `critical`, `high`, `medium`, `low` or `info`. A serious problem is a `critical` or `high` finding.
- `title` says what is wrong, in plain words.
- `detail` says what it means. For a locked finding, it says only that the details are in the report.
- `evidence` is optional: short lines of proof. LidTight masks secrets and personal data.
- `locked: true` is set on every `critical`, `high` and `medium` finding. LidTight shows what is wrong and how serious, not where. Where it is, the proof and the fix are in LidTight's paid report. The `id` of a locked finding (`locked-3`) does not name the place.

`low` and `info` findings come in full. One `info` finding, "N tips for your app", has an empty `detail` and lists the tips in `evidence`.

### `done`: the last event

```text
data: {"type":"done","counts":{"critical":1,"high":1,"medium":2,"low":0,"info":3},"ms":16073,"deep":false,"scanId":"<scan id>"}
```

- The check is complete only when you see this event.
- `counts` gives the number of findings for each severity.
- `ms` is the run time in milliseconds.
- `deep` is always `false` here.
- You do not need `scanId`.

### `error`: the check stopped

```text
data: {"type":"error","message":"Too many requests. Try again in an hour."}
```

Tell the user the message in plain words. Do not run the check again on your own.

| Message | What to do |
|---|---|
| "That link does not look right." | Check the link with the user. |
| "We could not find <host>. Check the spelling of the link." | Check the spelling with the user. |
| "We can only check apps that are live on the public internet." | Ask for the public link. |
| "We could not open that link. Check that the app is live." | Ask the user to check that the app is live. |
| "The app answered with HTTP <status>. Check that the app is live and public." | Ask the user to check that the app is live and public. |
| "Many checks are running right now. Try again in a minute." | Tell the user. Run again only if the user asks. |
| "Too many requests. Try again in an hour." | The hourly limit is reached. Wait an hour. |
| "The check stopped unexpectedly. Try again." | Tell the user. Run again only if the user asks. |

If the answer has no `done` event and no `error` event (for example, it is an HTML page or it is empty), the check did not finish. Tell the user. Run again only if the user asks.

## Step 5: Explain the result

Write for a founder who is not a security expert. Use short sentences and common words.

1. First line: the number of serious problems (`critical` plus `high`). If there are none, say: "The free check found no serious problem."
2. Then each finding, most serious first: `critical`, `high`, `medium`, `low`, `info`. For each one, say:
   - what is wrong, from the title;
   - what a stranger could do;
   - what it means for the founder's money and customers;
   - the matching guide, from the table below.
3. Use only what the events say. A locked finding does not say where the problem is. Do not guess the table, the key or the file.
4. Give the tips from the tips finding as a short list.
5. Add the run time and the counts from the `done` event. For example: "Done in 16.1 s: 1 critical, 1 high, 2 medium, 3 info."
6. Say what the free check cannot see (see below).
7. Offer next steps: a fix in the user's own code (Step 6), the guides, and the deep checks.

### What each kind of finding means

Match the finding by its title. All guides are at https://lidtight.com/guides.

| The title says | What it means for the founder | Guide |
|---|---|---|
| "Anyone can read … records", "Anyone can read your whole database", "Anyone can read records in one of your collections", "Anyone can read personal data from one of your endpoints" | A stranger can download customer data without signing in. | https://lidtight.com/guides/what-strangers-see. For Supabase or Lovable Cloud, also https://lidtight.com/guides/supabase-public-key. For Base44, https://lidtight.com/guides/base44-app-security |
| "Anyone can list … files your users uploaded" | A stranger can list the files that the app's users uploaded. | https://lidtight.com/guides/what-strangers-see |
| "A secret key is in your public code", "Something that looks like a secret key is in your public code", "A settings file with secret keys is public", "A file with your secret settings is public" | Someone can spend the founder's money with the founder's own keys, or reach data the key opens. | https://lidtight.com/guides/launch-security-checklist |
| "Your full source code is public" | Anyone can read the app's code as it was written. | https://lidtight.com/guides/what-strangers-see |
| "Your code history can be downloaded", "A copy of your database can be downloaded" | A stranger can download the app's code history or a copy of its database. | No guide yet: https://lidtight.com/guides |
| "Bots can create unlimited fake accounts in your app" | A bot can sign up thousands of fake accounts with made-up emails. Each one can use the app, and its free limits, as a real user. | https://lidtight.com/guides/signup-without-email-confirmation |
| "Any website can read your users' data from your app" | Another website can read a signed-in user's data from the app. | https://lidtight.com/guides/security-headers-cors |
| "Some browser protections are switched off" | Browsers do not block common tricks against the app's users. These settings are quick to turn on. | https://lidtight.com/guides/security-headers-cors |
| "Visitors can open your app without encryption" | Visitors can open the app over plain `http://`. | https://lidtight.com/guides/security-headers-cors |
| "Anyone can send email in your name", "Fake email from … is reported, not blocked" | Someone can send email that looks like it comes from the founder's domain. | No guide yet: https://lidtight.com/guides |
| "Your app may work out prices in the browser", "Your app sends prices or amounts from the browser", "Your app sends roles or permissions from the browser" | A hint from the code, not tested. If it is true, a user can change a price or a role and get paid features for free. | https://lidtight.com/guides/paid-features-for-free |
| "Your app tells the server which user owns the data" | A hint from the code, not tested. If it is true, one user can reach another user's data. | https://lidtight.com/guides/one-customer-sees-another |
| "We can see inside your app: N tables" (info) | Anyone can see the table and column names. The access rule on each table decides what a stranger can read. | https://lidtight.com/guides/supabase-public-key |
| "An admin page is listed in your public code" (info) | Anyone can see that the page exists. What matters is that the server refuses its data to normal users. The deep checks test this. | https://lidtight.com/guides/launch-security-checklist |
| Any other title | Explain it from the title and the detail. | https://lidtight.com/guides |

Two more guides help often:

- Before a launch: https://lidtight.com/guides/launch-security-checklist
- When the user says the builder's own scan already passed: https://lidtight.com/guides/is-builder-scan-enough

### What the free check cannot see

Say this every time, in short words:

- The free check acts as a stranger. It does not sign in. The worst problems often hide behind sign-in.
- It cannot see whether one customer can see or change another customer's data, whether a user can get paid features, credits or admin rights for free, whether anyone can upload files at the founder's cost, or whether a bot can run up the AI bill.
- It reads only what the browser gets. It does not read the server code. A hint about logic in the browser is not a test.
- Its database checks cover Supabase (also Lovable Cloud), Base44 and Firebase. Other databases are not checked yet. Its sign-up check covers only Supabase (also Lovable Cloud) so far.
- No serious finding does not prove that the app is safe.

Then tell the user about the deep checks:

- The deep checks are free, for the app's owner. They test the app as two new signed-in test users.
- They answer: Can one customer see another's data? Can someone get paid features, credits or admin rights for free? Can one customer change another's orders? Can anyone upload files at your cost? Can a bot run up your AI bill?
- They create one test order, change only their own test data and change it back, and call the AI helper up to 5 times. The two test users stay in the app. If the app does not let users delete orders, the test order stays too, and LidTight says which one.
- So far, the deep checks work only with Supabase (also Lovable Cloud).
- To ask for them, the user opens https://lidtight.com, pastes the link, clicks "Check my app", then leaves an email under the result and confirms it. LidTight looks at each request before the deep checks run. The results come by email.

Matching guides: https://lidtight.com/guides/one-customer-sees-another, https://lidtight.com/guides/paid-features-for-free, https://lidtight.com/guides/file-upload-abuse, https://lidtight.com/guides/ai-bill-abuse.

## Step 6: Fix, only when the user asks

If the user asks you to fix a finding:

- Work only in the user's own code and settings.
- Read the matching guide for the usual cause and fix.
- Common fixes: turn on row level security (the database's access rules) for every Supabase table, with a rule per table for who may read and who may change; move secret keys to server code; ask new users to confirm their email; check admin rights, plans and prices on the server, never only in the page; turn on the missing browser protections.
- If a secret key was ever public, tell the user to rotate it: make a new key in the provider's dashboard and turn off the old one. Moving the key out of the code is not enough.
- Do not test the fix against the live app yourself. After the user publishes the fix, offer to run the check once more.

## Example

A real run on Tidybook, LidTight's own demo app, on 1 November 2026, ended with:

```text
data: {"type":"done","counts":{"critical":1,"high":1,"medium":2,"low":0,"info":3},"ms":16073,"deep":false,"scanId":"<scan id>"}
```

A good answer starts like this:

> The free check found 2 serious problems in tidybook-sweet-spot.lovable.app: 1 critical and 1 high. It also found 2 medium problems. Done in 16.1 s.
>
> 1. **Critical: anyone can read 12 records, with names, emails and phone numbers.** No sign-in is needed. A stranger can download your customers' contact details. LidTight counted the records and read a few values only to check them. It kept none. Guide: https://lidtight.com/guides/what-strangers-see
> 2. **High: bots can create unlimited fake accounts.** A bot can sign up thousands of fake accounts. Each one can use your app, and your free limits, as a real user. Guide: https://lidtight.com/guides/signup-without-email-confirmation
> 3. **Medium: some browser protections are switched off.** They are quick to turn on. Guide: https://lidtight.com/guides/security-headers-cors
> 4. **Medium: your app may work out prices in the browser.** This is a hint from your code, not a test. If it is true, a user can pay less than your price. Guide: https://lidtight.com/guides/paid-features-for-free
>
> LidTight's page shows what is wrong, not where. Where each problem is, the proof and the fix are in its paid report. I can also look for the cause in your own code if you want.
>
> What the free check cannot see: it does not sign in. ...
</content>
</invoke>
