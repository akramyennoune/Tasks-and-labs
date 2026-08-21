# Linux File Permissions — Hands-On Lab

## Project Overview

This project walks through investigating and hardening file/directory permissions on a Linux system. The scenario: a research team's shared project directory has grown a little too permissive, and it's my job to audit what's there, understand exactly what each permission set means, and lock things down to a least-privilege model — without breaking legitimate access.

## Environment

Working directory: `/home/researcher2/projects`, owned by user `researcher2`, group `research_team`.

```bash
researcher2@97f03f8bf26b:~$ pwd
/home/researcher2
researcher2@97f03f8bf26b:~$ cd projects
researcher2@97f03f8bf26b:~/projects$ ls -la
total 32
drwxr-xr-x 3 researcher2 research_team 4096 Aug 21 15:39 .
drwxr-xr-x 3 researcher2 research_team 4096 Aug 21 15:56 ..
-rw--w---- 1 researcher2 research_team   46 Aug 21 15:39 .project_x.txt
drwx--x--- 2 researcher2 research_team 4096 Aug 21 15:39 drafts
-rw-rw-rw- 1 researcher2 research_team   46 Aug 21 15:39 project_k.txt
-rw-r----- 1 researcher2 research_team   46 Aug 21 15:39 project_m.txt
-rw-rw-r-- 1 researcher2 research_team   46 Aug 21 15:39 project_r.txt
-rw-rw-r-- 1 researcher2 research_team   46 Aug 21 15:39 project_t.txt
```

Using `ls -la` instead of a plain `ls` was the key move here — it surfaces hidden files (anything starting with a dot), which `ls` alone would skip right past. That's how `.project_x.txt` showed up at all.

### Starting permissions

| File | User | Group | Other |
|---|---|---|---|
| project_k.txt | read, write | read, write | read, write |
| project_m.txt | read, write | read | none |
| project_r.txt | read, write | read, write | read |
| project_t.txt | read, write | read, write | read |
| .project_x.txt (hidden) | read, write | write | none |

## Reading a permissions string

I picked the hidden file, `.project_x.txt`, to break down: `-rw--w----`

The full pattern is `[type][owner][group][other]`, ten characters total:

- **Position 1** — file type. `-` means a regular file; a directory would show `d` instead.
- **Positions 2–4** — owner permissions.
- **Positions 5–7** — group permissions.
- **Positions 8–10** — permissions for everyone else.

`r` = read, `w` = write, `x` = execute. A dash in any slot means that permission isn't granted.

For `.project_x.txt` specifically: the owner has read and write but not execute, the group only has write (no read, which is a little unusual and worth flagging), and everyone else has nothing at all.

## Step 1 — Tightening `project_k.txt`

Org policy is that outside users shouldn't be able to write to project files. `project_k.txt` was sitting at `-rw-rw-rw-`, meaning literally anyone on the system could edit it. That's the first thing I fixed:

```bash
researcher2@97f03f8bf26b:~/projects$ chmod o-w project_k.txt
researcher2@97f03f8bf26b:~/projects$ ls -la
-rw-rw-r-- 1 researcher2 research_team 46 Aug 21 15:39 project_k.txt
```

`chmod o-w` removes the write bit specifically from the "other" category, leaving owner and group access untouched. Confirmed with a follow-up `ls -la` — other now shows `r--` instead of `rw-`.

## Step 2 — Locking down the hidden file

Next target: `.project_x.txt`. The goal was to strip it down to owner-read-only — no one else should be able to touch it, and even the owner shouldn't be able to write to it anymore.

```bash
researcher2@97f03f8bf26b:~/projects$ chmod u-w,g-w .project_x.txt
researcher2@97f03f8bf26b:~/projects$ ls -la
-r-------- 1 researcher2 research_team 46 Aug 21 15:39 .project_x.txt
```

`chmod u-w,g-w` removes write access from both the owner (`u`) and group (`g`) in a single command — no need to run `chmod` twice. End result: `-r--------`, meaning only the owner can read it, and nobody (including the owner) can write to it.

## Step 3 — Restricting the `drafts` directory

Last piece: the `drafts` directory needed to be scoped down so that `researcher2` could still get in, but every other permission was stripped away — group and other included.

```bash
researcher2@97f03f8bf26b:~/projects$ chmod g-r,g-w,g-x,o-r,o-w,o-x drafts
researcher2@97f03f8bf26b:~/projects$ ls -la
drwx------ 2 researcher2 research_team 4096 Aug 21 15:39 drafts
```

That's six flags removed in one command — read, write, and execute for both group and other. What's left is `drwx------`: the owner can read, write, and enter the directory, and everyone else is fully locked out.

## Summary

Final state of the directory:

```
drwxr-xr-x 3 researcher2 research_team 4096 Aug 21 15:56 ..
-r-------- 1 researcher2 research_team   46 Aug 21 15:39 .project_x.txt
drwx------ 2 researcher2 research_team 4096 Aug 21 15:39 drafts
-rw-rw-r-- 1 researcher2 research_team   46 Aug 21 15:39 project_k.txt
-rw-r----- 1 researcher2 research_team   46 Aug 21 15:39 project_m.txt
-rw-rw-r-- 1 researcher2 research_team   46 Aug 21 15:39 project_r.txt
-rw-rw-r-- 1 researcher2 research_team   46 Aug 21 15:39 project_t.txt
```

This got the directory to a least-privilege state — nobody has more access than they actually need. It also ties directly into the **integrity** pillar of the CIA triad: by controlling who can *modify* files (not just who can see them), the risk of unauthorized or accidental changes drops significantly.

**Takeaway:** permission auditing is a small habit with a big payoff. `ls -la` + reading the permission string carefully catches misconfigurations that are easy to miss, and `chmod`'s symbolic mode (`u`/`g`/`o` with `+`/`-`) makes targeted fixes fast without having to recalculate octal values by hand. For a SOC-analyst-style workflow, being fluent in Linux permissions matters beyond just hardening — it's also how you reason about what a compromised account could and couldn't have touched.

## Tools Used

- Linux shell (`ls -la`, `chmod`)
- `chmod` symbolic mode: `u` (user/owner), `g` (group), `o` (other), `+`/`-` to add or remove

