# Linux Package Management & Shell I/O — Lab Notes

Part of my ongoing SOC analyst prep (Google Cybersecurity Certificate). This lab covered two things: managing software with APT on a Debian-based system, and getting comfortable with basic input/output in the Bash shell.

## Part 1: Package management with APT

Started by confirming APT was actually available by just running `apt` with no arguments. It came back with version info and usage instructions, which confirmed it was installed — no surprise since this is a Debian-based box and APT is the default there. On a different distro (Red Hat, CentOS, etc.) this would've been YUM or DNF instead.

From there I installed Suricata, a network intrusion detection/analysis tool:

```
sudo apt install suricata
```

Needed `sudo` here since installing software requires elevated privileges. Took a couple minutes, and the install output listed out dependencies being pulled in along with Suricata itself. Confirmed the install worked by just running `suricata`, which printed version and usage info (Suricata 4.1.2 in this case).

Then removed it:

```
sudo apt remove suricata
```

Ran `suricata` again afterward and got `-bash: /usr/bin/suricata: No such file or directory`, confirming the removal actually worked and didn't just look like it worked.

Next installed tcpdump, a packet capture tool:

```
sudo apt install tcpdump
```

To verify what was actually installed on the system, I used:

```
apt list --installed
```

This dumps every installed package, which is a lot since Linux ships with plenty of software by default. Had to scroll/search through it to confirm tcpdump showed up (it did, flagged `[installed]`) and Suricata didn't, since I'd already removed it.

Last step was reinstalling Suricata and re-checking the installed list to make sure both Suricata and tcpdump showed up together. They did.

**Takeaway:** APT install/remove/list is basic stuff, but the point of the exercise was really about verification — not just running a command and assuming it worked, but checking the actual state of the system afterward. That habit matters a lot more once you're doing this on production systems or during an incident.

## Part 2: Shell input/output basics

Second half was lower-level: how the shell handles input and returns output, using `echo` and `expr`.

`echo` just prints back whatever string you give it:

```
echo hello
echo "hello"
```

Both return `hello` — quotes don't change the output here, they just tell the shell to treat everything inside as one grouped string, which matters more once you're passing in special characters.

`expr` handles basic integer math. Framed this in a security-relevant way: say you've got 32 alerts total and only 8 needed real action — how many were false positives?

```
expr 32 - 8
```
→ `24`

Also ran a quick projection — averaging 3500 login attempts a month, what's the yearly total?

```
expr 3500 * 12
```
→ `42000`

Worth noting `expr` only does integer math — no decimals, everything gets rounded down. And every operator/term needs to be space-separated (`expr 32 - 8`, not `expr 32-8`) or it'll error out.

Wrapped up with `clear` to reset the terminal view.

**Takeaway:** Nothing here is advanced, but this is the muscle memory layer — knowing how the shell interprets input and returns output (or throws an error) before moving on to anything more complex like scripting or log parsing. Also a good reminder that command-line math like this is genuinely useful for quick triage calculations (false positive rates, volume projections) instead of pulling out a calculator or spreadsheet mid-shift.

---

# Linux File Navigation, grep, and File Management — Lab Notes

Continuing my SOC analyst prep (Google Cybersecurity Certificate). This batch of labs covered navigating a Linux directory structure, reading file contents, filtering with grep and piping, and basic file/directory management (create, move, remove, edit).

## Lab 1: Navigating directories and reading files

Scenario was investigating files under `/home/analyst` — the kind of thing you'd actually do when pulling a user access report during an investigation.

Started with the basics of orienting myself:

```
pwd
ls
```

`pwd` confirms the working directory, `ls` lists what's in it. From `/home/analyst` I moved into the reports directory:

```
cd /home/analyst/reports
ls
```

That showed a `users` subdirectory (along with the other report folders). Moved into it and read a user file:

```
cd /home/analyst/reports/users
ls
cat Q1_added_users.txt
```

`cat` dumps the whole file to the terminal — useful for quickly checking a user list for a specific employee's department or ID without opening anything in a GUI.

Then navigated over to the logs directory and checked the start of a log file instead of the whole thing:

```
cd /home/analyst/logs
ls
head server_logs.txt
```

`head` defaults to the first 10 lines, which is exactly what you want when you're skimming a log for warning/error entries without scrolling through the entire file.

**Takeaway:** `cd`, `pwd`, `ls`, `cat`, and `head` cover most of what you need to move through a filesystem and pull relevant content out of files quickly — which matters a lot when you're working entirely through a remote shell with no GUI.

## Lab 2: Filtering with grep and piping

Same file structure, but this time the goal was searching rather than just reading.

First, filtering a log file down to just the error lines:

```
cd /home/analyst/logs
grep error server_logs.txt
```

`grep` searches for a string and only prints matching lines — a lot faster than reading the whole log to manually pick out errors.

Next, finding files by name pattern using piping:

```
cd /home/analyst/reports/users
ls | grep Q1
```

This pipes the output of `ls` into `grep`, so instead of grep searching file contents, it's filtering the list of filenames itself — in this case, anything with "Q1" in the name. Did the same thing again for "access":

```
ls | grep access
```

Then searched inside specific files for specific data:

```
grep jhill Q2_deleted_users.txt
grep "Human Resources" Q4_added_users.txt
```

The second one needed quotes around "Human Resources" since it's two words — without quotes, grep would treat "Human" and "Resources" as separate arguments instead of one search string.

**Takeaway:** grep plus piping is the difference between manually scanning files and actually querying them. Being able to pull "every line with X" or "every filename containing Y" out of a directory full of logs and reports is a core part of triage — you're rarely reading everything start to finish, you're searching for what matters.

## Lab 3: File and directory management

This one was less about reading data and more about basic housekeeping — creating, moving, and removing files and directories, plus editing a file directly in the shell.

Created a new subdirectory for future log storage:

```
mkdir logs
ls
```

Removed a directory that was no longer needed:

```
rm -r temp
ls
```

(`-r` for recursive, since removing a directory means removing everything inside it too.)

Moved a file from one directory to another:

```
cd notes
mv Q3patches.txt ../reports/
```

`mv` handles both moving and renaming in Linux — there's no separate "move" vs "rename" command, it's the same operation depending on whether the destination path changes the name or not.

Removed a single file:

```
rm tempnotes.txt
```

Created a new empty file:

```
touch tasks.txt
```

`touch` is normally used to update a file's timestamp, but if the file doesn't exist yet it creates it empty — handy for quickly scaffolding a file before editing it.

Then opened it in nano to actually add content:

```
nano tasks.txt
```

Typed in a couple of lines, then saved and exited with `CTRL+X`, confirmed the save with `Y`, and confirmed the filename with `ENTER`. (Normally you'd use `CTRL+O` to save and `CTRL+X` to exit, but in a browser-based shell `CTRL+O` gets intercepted by the browser itself, so `CTRL+X` → `Y` → `ENTER` is the workaround.) Cleared the screen afterward since nano can leave some leftover text artifacts in a web terminal, then confirmed the file contents with `cat`.

**Takeaway:** `mkdir`, `rm`, `mv`, `touch`, and basic nano usage round out the file management side of working in a shell. Combined with navigation and grep from the earlier labs, this covers a decent chunk of what you'd actually be doing day to day investigating or organizing files on a Linux system with no GUI to fall back on.

---
#Terms and definitions:

Absolute file path: The full file path, which starts from the root

Argument (Linux): Specific information needed by a command

Authentication: The process of verifying who someone is

Authorization: The concept of granting access to specific resources in a system

Bash: The default shell in most Linux distributions

Command: An instruction telling the computer to do something

File path: The location of a file or directory

Filesystem Hierarchy Standard (FHS): The component of the Linux OS that organizes data

Filtering: Selecting data that match a certain condition

nano: A command-line file editor that is available by default in many Linux distributions

Options: Input that modifies the behavior of a command

Permissions: The type of access granted for a file or directory

Principle of least privilege: The concept of granting only the minimal access and authorization required to complete a task or function

Relative file path: A file path that starts from the user's current directory

Root directory: The highest-level directory in Linux

Root user (or superuser): A user with elevated privileges to modify the system

Standard input: Information received by the OS via the command line

Standard output: Information returned by the OS through the shell

*Environment: Debian-based Linux, Bash shell*

