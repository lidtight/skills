# LidTight skills

This repository holds one Agent Skill, `lidtight-check`, for AI coding agents such as Claude Code, Codex, Cursor and Lovable. With it, the agent runs the free check from [LidTight](https://lidtight.com) on your own live web app and explains the result in plain words. If you clearly say yes, it also runs the free deep checks on your own app. LidTight checks apps built with AI tools such as Lovable, Bolt, Base44 and Replit. The free check reads only what any visitor's browser can see. It needs no key and no sign-up, and it takes from a few seconds to about a minute. The skill follows the open [Agent Skills format](https://agentskills.io/specification).

```text
.
├── .claude-plugin/
│   ├── marketplace.json      # Claude Code marketplace: lists the plugin below
│   └── plugin.json           # Claude Code plugin manifest
├── skills/
│   └── lidtight-check/
│       └── SKILL.md          # the skill
├── LICENSE
└── README.md
```

## Install

### Claude Code

**From the plugin marketplace.** In a Claude Code session, add this repository as a marketplace, then install the plugin:

```text
/plugin marketplace add lidtight/skills
/plugin install lidtight@lidtight
```

`/plugin install` opens the plugin's details. Choose a scope there. The skill then shows as `/lidtight:lidtight-check`. From your shell, the same steps are `claude plugin marketplace add lidtight/skills` and `claude plugin install lidtight@lidtight`.

Docs: [Install and manage plugins](https://code.claude.com/docs/en/plugins/install), [Create a marketplace](https://code.claude.com/docs/en/plugin-marketplaces).

**Or copy the skill folder.** For all your projects, copy it to `~/.claude/skills/`. For one project, copy it to `.claude/skills/` in that project.

```bash
git clone https://github.com/lidtight/skills.git lidtight-skills
mkdir -p ~/.claude/skills
cp -R lidtight-skills/skills/lidtight-check ~/.claude/skills/
```

The skill then shows as `/lidtight-check`.

Docs: [Extend Claude with skills](https://code.claude.com/docs/en/skills).

### Codex

Copy the skill folder to `~/.agents/skills/` for all your projects, or to `.agents/skills/` in a repository.

```bash
git clone https://github.com/lidtight/skills.git lidtight-skills
mkdir -p ~/.agents/skills
cp -R lidtight-skills/skills/lidtight-check ~/.agents/skills/
```

Codex finds new skills by itself. If the skill does not appear, restart Codex. To call it by name, type `$` or run `/skills`. You can also ask `$skill-installer` to install the skill from `https://github.com/lidtight/skills/tree/main/skills/lidtight-check`.

The skill runs one `curl` command that needs the network. Codex's default sandbox keeps network access off, and Codex asks for your approval before a command that needs the network.

Docs: [Agent Skills in Codex](https://learn.chatgpt.com/docs/build-skills), [Agent approvals and security](https://learn.chatgpt.com/docs/agent-approvals-security).

### Cursor

Copy the skill folder to `~/.cursor/skills/` for all your projects, or to `.cursor/skills/` in a project.

```bash
git clone https://github.com/lidtight/skills.git lidtight-skills
mkdir -p ~/.cursor/skills
cp -R lidtight-skills/skills/lidtight-check ~/.cursor/skills/
```

Cursor also loads skills from `.agents/skills/` and `.claude/skills/`, so a copy you made for Codex or Claude Code works too. Cursor imports skills from a GitHub repository only as part of a Cursor plugin, and this repository does not have one, so copy the folder.

Docs: [Agent Skills in Cursor](https://cursor.com/docs/context/skills).

### Lovable

Workspace owners and admins can import the skill:

1. Open **Workspace settings → Skills → Add**.
2. Choose **Import from GitHub**.
3. Paste `https://github.com/lidtight/skills/tree/main/skills/lidtight-check`.

Lovable's docs do not say whether a skill can run a shell command or reach the network. If Lovable cannot run the check, the skill asks you to run it at [lidtight.com](https://lidtight.com) and then explains the result.

Docs: [Skills in Lovable](https://docs.lovable.dev/features/skills).

## What the free check looks at

- Can someone spend your money with your own keys? (secret keys in the public code)
- Can a stranger download your customer list? (Supabase tables, Base44 entities, Firebase collections and the app's own endpoints open to anyone)
- Can bots sign up without limits? (sign-up without email confirmation)
- Did secret files ship with your app? (`.env`, `.git`, database backups, settings files)
- Is your source code public? Are browser and email protections on? (security headers, https, CORS, fake email in your name)

The free check counts records and never keeps customer data. For a serious problem, it shows what is wrong and how serious, not where.

## The deep checks

The deep checks are free too, and only for your own app. They test the app as two signed-in test users: Can one customer see another's data? Can someone get paid features, credits or admin rights for free? Can one customer change another's orders? Can anyone upload files at your cost? Can a bot run up your AI bill?

After you confirm that the app is yours, the skill asks one more question: "Also run the deep checks? They sign up 2 test users in your app and change only our own test data. Only for your own app." It shows the consent line with it. Only on a clear yes, it adds `deep=1` to the same command. The deep checks sign up 2 test users and create one test order in your app, and change only that test data. They upload one small text file to each storage bucket, then ask to delete it, and call your AI helper up to 5 times. The test users stay in your app. The test order stays too if your app does not let users delete it. On [lidtight.com](https://lidtight.com), the same choice is a box under the app link.

## Safety rules

The skill tells the agent to:

- check only your own app, or an app you control, and to stop without a clear yes;
- send LidTight only the app's public link, never passwords, secret keys, tokens or user data;
- never attack the app, and never try to exploit or confirm a finding itself;
- run the check once per change, wait for the result, and never run it in a loop: the limit is 5 checks per visitor per hour;
- run the deep checks only on your own app, and only after you clearly say yes;
- never pay, enter card details or create accounts for you;
- change your code only when you ask.

## How it works

The skill calls `https://lidtight.com/api/scan?url=<your app's live link, URL-encoded>`, with `&deep=1` only after your yes to the deep checks. The answer is a stream of `data: {json}` lines with the event types `step`, `finding`, `done` and `error`. Lines that start with `:` only keep the connection open, and the agent ignores them. The agent reads the stream until the `done` event, then explains each finding, most serious first, and links the matching guide. If the agent has no shell or no network, it sends you to [lidtight.com](https://lidtight.com) instead.

## Links

- [lidtight.com](https://lidtight.com): run the check in your browser.
- [lidtight.com/guides](https://lidtight.com/guides): plain answers to the security questions founders ask.
- [lidtight.com/llms.txt](https://lidtight.com/llms.txt): what LidTight does, for AI assistants.

## License

MIT. See [LICENSE](LICENSE).
</content>
</invoke>
