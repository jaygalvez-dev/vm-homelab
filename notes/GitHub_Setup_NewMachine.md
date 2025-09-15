# GitHub Setup on a New Machine (Linux) — Things I Learned

**Date:** 2025-09-15  
**Machine:** Asus Q500A (Ubuntu Jammy / Linux)  
**Repo Example:** `vm-homelab`

---

## Goal
Set up this machine so it can **clone, pull, and push** to my GitHub repos (already used on another computer).

---

## 0) Prereqs
- A GitHub account (logged in in your browser).
- Either:
  - **SSH key** added to GitHub, or
  - **Personal Access Token (PAT)** for HTTPS pushes.

---

## 1) Install Git (if needed)
```bash
git --version
# If not installed:
sudo apt update && sudo apt install -y git
```

---

## 2) Identify yourself (per machine)
```bash
git config --global user.name "YOUR_GITHUB_USERNAME"
git config --global user.email "your_github_email@example.com"
git config --global init.defaultBranch main
```

> These settings are stored per machine (~/.gitconfig). You do this once on each new device.

---

## 3) Authenticate: SSH **(recommended)** or HTTPS

### Option A — SSH (recommended)
1. **Create a key** (if you don’t already have one):
   ```bash
   ssh-keygen -t ed25519 -C "your_github_email@example.com"
   # Press Enter to accept defaults; optionally set a passphrase.
   ```
2. **Start ssh-agent and add your key**:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```
3. **Copy the public key** and add it to GitHub → *Settings → SSH and GPG keys → New SSH key*:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
4. **Test**:
   ```bash
   ssh -T git@github.com
   # Expect something like: "Hi <username>! You've successfully authenticated..."
   ```

### Option B — HTTPS + Personal Access Token (PAT)
1. Generate a **token** on GitHub → *Settings → Developer settings → Personal access tokens → Tokens (classic)*.  
   - Scopes: at minimum, `repo`.
2. When pushing/pulling over HTTPS, use your username and **the token as your password**.  
   ```bash
   git remote set-url origin https://github.com/<USER>/<REPO>.git
   # Next push will prompt for credentials; use token for password.
   ```

---

## 4) Clone the repo on this machine
```bash
cd ~/Projects  # pick your workspace folder
git clone git@github.com:jaygalvez-dev/vm-homelab.git   # SSH
# or:
# git clone https://github.com/jaygalvez-dev/vm-homelab.git  # HTTPS
cd vm-homelab
```

**Verify remotes:**
```bash
git remote -v
```

---

## 5) Typical workflow (daily use)
```bash
# Get latest changes
git pull --ff-only

# Make changes, add files
git add <files>

# Commit with a clear message
git commit -m "docs(notes): add apt i386 architecture warning explanation"

# Push to GitHub
git push
```

**Create standard folders** (once per repo):
```bash
mkdir -p notes screenshots scripts
```

**Add the i386 note (example):**
```bash
mv ~/Downloads/Linux_i386_Warning.md notes/
git add notes/Linux_i386_Warning.md
git commit -m "docs(notes): add apt i386 architecture warning explanation"
git push
```

---

## 6) Working across multiple machines
- Each machine needs **its own** SSH key (or uses HTTPS + PAT).  
- Always `git pull --ff-only` before starting new work.  
- Keep commit messages consistent and descriptive.  
- If you do the same work on two machines, make sure you **push from machine A** and then **pull on machine B** before editing.

---

## 7) Troubleshooting

### `Permission denied (publickey)`
- Your SSH key isn’t loaded or not added to GitHub.
  ```bash
  ssh-add -l              # see if the key is loaded
  cat ~/.ssh/id_ed25519.pub  # add to GitHub if missing
  ssh -T git@github.com
  ```

### Authentication failed (HTTPS)
- You are using a password instead of a token. Create a PAT and use it as the password.  
- Update the remote to HTTPS (or switch to SSH):
  ```bash
  git remote set-url origin https://github.com/<USER>/<REPO>.git
  # or
  git remote set-url origin git@github.com:<USER>/<REPO>.git
  ```

### Divergent branches / merge commits you don’t want
- Prefer fast-forward only pulls:
  ```bash
  git pull --ff-only
  ```
- If you already created merge commits and want a cleaner history: consider a rebase (careful with shared branches).

### Detached HEAD
- You checked out a commit/Tag instead of a branch.
  ```bash
  git switch main
  ```

### Wrong email in commits
- Fix global config and amend:
  ```bash
  git config --global user.email "correct_email@example.com"
  git commit --amend --reset-author
  git push -f  # only if safe to force-push
  ```

---

## 8) Optional niceties

### `.gitignore` (repo root)
```
# OS cruft
.DS_Store
Thumbs.db

# Editors
.vscode/
.idea/

# Archives
*.zip
*.7z
*.tar
*.tar.gz

# VM / images
*.iso
*.vdi
*.vmdk
*.ova
*.img

# Logs
*.log
```

### MIT License
Create a `LICENSE` and pick MIT to make sharing clear.

### README link to notes
```md
## 🔧 Troubleshooting Logs
- [Ubuntu apt i386 architecture notice](notes/Linux_i386_Warning.md)
- [GitHub setup on a new machine](notes/GitHub_Setup_NewMachine.md)
```

---

## 9) Quick checklist for **any** new machine
- [ ] Install Git
- [ ] `git config --global user.name` / `user.email`
- [ ] Create SSH key **or** prepare HTTPS PAT
- [ ] Add SSH public key to GitHub (if using SSH)
- [ ] `git clone` the repo
- [ ] `git pull --ff-only` before editing
- [ ] Commit → Push → Verify on GitHub

---

**Author:** Jayjay  
**Purpose:** Repeatable onboarding for any new dev box (OptiPlex, Q500A, etc.).
