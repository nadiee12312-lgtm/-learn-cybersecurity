# 🐧 LINUX FROM SCRATCH FOR HACKERS
> If you come from Windows and never touched a terminal, this book is for you.
> From "what is this?" to moving with confidence. Version 2.0

---

## 🤔 Wait, why Linux?

You come from Windows and wonder why the whole security world uses Linux. Real reasons:

- **It's transparent:** you can see and control EVERYTHING. Nothing is hidden.
- **It's the server OS:** most of what you'll audit/protect runs Linux.
- **The tools live here:** almost every security tool was born on Linux.
- **The terminal is power:** what takes 10 clicks in Windows is one command here that you can automate.
- **It's free and open.**

You don't have to abandon Windows. You can have Linux **inside** Windows (with WSL) or in a virtual machine. The best of both worlds.

---

## ⌨️ The terminal: your new superpower

What scares people most at first is the **terminal** (that black screen with text). But think of it this way:

> In Windows you tell the computer what to do with **clicks**. In Linux you tell it with **words**. And words are more precise, faster, and can be **automated**.

The terminal isn't hard. It's **honest**: it does exactly what you tell it. Once you lose the fear, you won't want to go back to clicks.

When you open the terminal, you see something like:
```
user@machine:~$
```
That's called "the prompt". The `$` means "I'm ready, give me a command". The `~` means you're in your personal folder (your "home").

---

## 🚶 Your first steps (navigation commands)

Let's start moving. Type these and press Enter:

```bash
pwd          # "print working directory" → where am I now?
ls           # "list" → what's in this folder?
ls -la       # lists EVERYTHING, even hidden files, with details
cd folder    # "change directory" → enter a folder
cd ..        # go up one level
cd ~         # back to your home folder
clear        # clear the screen
```

**Practice now:** open your terminal, type `pwd`, then `ls`, then enter some folder with `cd`. Move around. Lose the fear. You won't break anything just by looking.

---

## 📂 How Linux is organized

In Windows you have `C:\`. In Linux everything hangs from a single root: `/`. It's like a tree:

```
/           → the root, the origin of everything
├── /home   → users' folders (yours: /home/yourname)
├── /etc    → system configuration (important in security)
├── /var    → changing data, like LOGS (/var/log) ← key in forensics
├── /bin    → programs/commands
├── /tmp    → temporary files
└── /root   → the superuser's folder (the system's boss)
```

Don't memorize it now. Just understand that **everything is a single folder structure**, and that `/home/yourname` is where you live.

---

## 📄 Working with files

```bash
cat file.txt      # show a file's contents
less file.txt     # show it paginated (exit with q)
nano file.txt     # open a simple editor to write (Ctrl+X to exit)
touch new.txt     # create an empty file
mkdir myfolder    # create a folder
cp file dest      # copy
mv file dest      # move or rename
rm file           # delete (CAREFUL, no recycle bin!)
```

⚠️ **Careful with `rm`:** in Linux there's no recycle bin. What you delete is gone forever. And **NEVER** type `rm -rf /` (that wipes the whole system — it's a cruel joke that circulates, don't fall for it).

---

## 🔍 Finding things (super useful)

```bash
find / -name "file.txt" 2>/dev/null   # find a file by name
grep "word" file.txt                  # find a word INSIDE a file
which nmap                            # where is a program installed?
history                              # see the commands you already typed
```

`grep` is one of your best friends — it finds text inside files. You'll use it all the time (in logs, in results, in everything).

---

## 🔐 Permissions (Linux's key concept)

In Linux, every file has **permissions**: who can read, write, or execute it. When you do `ls -l` you see something like:

```
-rwxr-xr--  user  file
```

Don't get overwhelmed. The simple idea:
- **r** = read
- **w** = write
- **x** = execute

And to make a file executable (for example, a script):
```bash
chmod +x script.sh    # now it can be executed
./script.sh           # this is how you run it
```

You'll understand permissions deeply with practice. For now, keep this: **each file has rules for who can do what.**

---

## 👑 sudo: asking for boss permission

Some things need administrator permissions (the "root", the system's boss). For that you use `sudo`:

```bash
sudo command      # runs that command as administrator (it'll ask your password)
```

**Golden rule:** don't use `sudo` for everything recklessly. Use it only when you truly need it. It's powerful, and with power comes responsibility (and the chance to break something).

---

## 📦 Installing programs

In Windows you download a `.exe`. In Linux you install from the terminal, easy and safe:

```bash
# On Kali / Ubuntu / Debian (they use apt):
sudo apt update              # update the program list
sudo apt install name        # install a program

# Example: install nmap
sudo apt install nmap
```

This downloads and installs the program safely, no viruses or weird installers. It's one of the most comfortable things about Linux.

---

## 🛠️ Essential commands for security

Since you'll do cybersecurity, these will serve you from the start:

```bash
ip a                 # see your IP address and network cards
ping google.com      # test if you have a connection
ss -tunap            # see active network connections
ps aux               # see all running programs
top                  # live process monitor (exit with q)
nmap -sV localhost   # your first scan (on your own machine)
```

**Your first real exercise:** run `nmap -sV localhost`. That scans YOUR OWN machine and tells you what services it has open. It's legal (it's yours), and it's your first real step in reconnaissance.

---

## 🐉 About your Kali

If you installed **Kali Linux**, great! Kali is a version of Linux that comes with **hundreds of security tools** preinstalled. It's the "hacker's Linux".

But remember from the other book: **having Kali doesn't make you a hacker.** Kali is a toolbox. This book teaches you to use the box. Knowledge is what's worth it, not the tools.

For now, just **move around Kali, practice the commands above, and get used to it.** The tools come later, when you understand the basics.

---

## 💡 Tricks that make your life easier

- **Tab key:** autocompletes names. Type `cd Down` and press Tab → completes `Downloads`. Saves you a lot.
- **Up/down arrows:** browse the commands you already typed. Don't retype everything.
- **Ctrl + C:** stops a command that got stuck running.
- **`command --help`:** almost every command explains how to use it with `--help`.
- **`man command`:** the full manual of a command (exit with q).

---

## 🔗 The trick that changes everything: pipes and redirections

This is what makes Linux so powerful. You can **connect commands** to each other:

```bash
# The pipe | sends one command's output to another
ls -la | grep ".txt"          # lists files, but ONLY the .txt ones
cat file.txt | grep "error"   # shows only the lines with "error"
ps aux | grep firefox         # finds the firefox process

# Redirections: save output to a file
command > file.txt            # save (overwrite)
command >> file.txt           # save (append to the end)
nmap localhost > scan.txt     # save the scan result
```

**Why it matters:** in security you chain commands all the time. "Scan → filter what's interesting → save it". The pipe `|` is your best friend. Practice it.

---

## 📋 Your cheat sheet (save it)

```
NAVIGATE:  pwd · ls -la · cd · cd .. · cd ~ · clear
FILES:     cat · less · nano · touch · mkdir · cp · mv · rm
SEARCH:    find · grep · which · history
SYSTEM:    ip a · ps aux · top · ss -tunap · df -h · free -h
PERMS:     chmod +x · sudo · chmod 600
HELP:      command --help · man command
SHORTCUTS: Tab (autocomplete) · ↑↓ (history) · Ctrl+C (stop)
```

---

## 🎯 Your practice plan

1. **Today:** open the terminal and practice navigation commands (`pwd`, `ls`, `cd`). Just move around.
2. **Tomorrow:** create files and folders, move them, delete them (in your home folder). Play.
3. **This week:** run `nmap -sV localhost` and `ip a`. Start feeling comfortable.
4. **Every day:** one new command. In a month you'll move with confidence.

**The best free resource to practice Linux:** **OverTheWire: Bandit** (overthewire.org/wargames/bandit). It's a game that teaches you Linux level by level. Start TODAY, it's addictive.

---

> ⬛ *The terminal isn't hard, it's honest. It does exactly what you tell it. Master it and you master the most powerful tool in security. One command at a time.*
>
> **Next book:** `book-programmer-to-hacker.md` — to turn your programming skill into a superpower.
