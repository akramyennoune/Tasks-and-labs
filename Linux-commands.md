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
*Environment: Debian-based Linux, Bash shell*
