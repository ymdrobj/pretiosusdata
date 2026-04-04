# Managing Multiple GitHub Accounts on One Machine

## How It Works

Each GitHub account needs its own SSH key. An SSH config file routes
Git to the right key based on a host alias you use in the remote URL.
A per-directory gitconfig ensures the right email shows on commits.

```
┌─────────────────────┐
│ ~/.ssh/config        │
│                      │
│ github-personal ───► id_personal   ──► GitHub (personal account)
│ github-evedge  ───► id_evedge     ──► GitHub (EV Edge account)
│ github-clientX ───► id_clientX    ──► GitHub (client X account)
└─────────────────────┘

┌─────────────────────┐
│ ~/.gitconfig         │
│                      │
│ ~/Developer/pretiosusdata/ ──► yaniv@pretiosusdata.com
│ ~/Developer/ev-edge/       ──► yaniv@evedge-email.com
│ (default)                  ──► personal@gmail.com
└─────────────────────┘
```

---

## Initial Setup (One-Time)

### 1. Create SSH directory

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### 2. Generate SSH keys

One key per GitHub account:

```bash
# Personal
ssh-keygen -t ed25519 -C "your-gmail@gmail.com" -f ~/.ssh/id_personal

# EV Edge
ssh-keygen -t ed25519 -C "your-evedge-email" -f ~/.ssh/id_evedge
```

Press enter twice to skip passphrase (or set one if you prefer).

### 3. Create SSH config

Create/edit `~/.ssh/config`:

```
# Personal GitHub
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_personal

# EV Edge GitHub
Host github-evedge
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_evedge
```

### 4. Add public keys to GitHub

For each account, copy the public key and paste it into
GitHub → Settings → SSH and GPG keys → New SSH key:

```bash
# Copy personal key to clipboard
cat ~/.ssh/id_personal.pub | pbcopy

# Copy EV Edge key to clipboard
cat ~/.ssh/id_evedge.pub | pbcopy
```

Log into each GitHub account separately to add the corresponding key.

### 5. Test connections

```bash
ssh -T git@github-personal
# Should respond: Hi <personal-username>! ...

ssh -T git@github-evedge
# Should respond: Hi <evedge-username>! ...
```

### 6. Set up per-directory Git identity

Edit `~/.gitconfig`:

```gitconfig
[user]
    name = Yaniv Mor
    email = your-gmail@gmail.com

[includeIf "gitdir:~/Developer/pretiosusdata/"]
    path = ~/.gitconfig-personal

[includeIf "gitdir:~/Developer/ev-edge/"]
    path = ~/.gitconfig-evedge
```

Create the per-directory config files:

```bash
# ~/.gitconfig-personal
[user]
    email = yaniv@pretiosusdata.com
```

```bash
# ~/.gitconfig-evedge
[user]
    email = your-evedge-email
```

---

## Daily Usage

### Cloning a repo

Always use the host alias instead of `github.com`:

```bash
# Personal repos
git clone git@github-personal:yourhandle/pretiosusdata.git

# EV Edge repos
git clone git@github-evedge:evedge-org/repo-name.git
```

### Initializing a new repo

```bash
cd ~/Developer/pretiosusdata
git init
git remote add origin git@github-personal:yourhandle/pretiosusdata.git
```

### Updating an existing repo from HTTPS to SSH

```bash
cd ~/Developer/ev-edge
git remote set-url origin git@github-evedge:evedge-org/repo-name.git
```

### Verifying which identity a repo uses

```bash
cd ~/Developer/pretiosusdata
git config user.email
# Should show: yaniv@pretiosusdata.com

git remote -v
# Should show: git@github-personal:...
```

---

## Adding a New Client Account

When you get a new client with their own GitHub org:

### 1. Generate a new key

```bash
ssh-keygen -t ed25519 -C "your-client-email" -f ~/.ssh/id_clientname
```

### 2. Add a block to `~/.ssh/config`

```
# Client Name GitHub
Host github-clientname
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_clientname
```

### 3. Add the public key to the client's GitHub account

```bash
cat ~/.ssh/id_clientname.pub | pbcopy
```

Log into the client's GitHub account → Settings → SSH and GPG keys → paste.

### 4. Add a per-directory gitconfig entry

In `~/.gitconfig`, add:

```gitconfig
[includeIf "gitdir:~/Developer/clientname/"]
    path = ~/.gitconfig-clientname
```

Create `~/.gitconfig-clientname`:

```gitconfig
[user]
    email = your-client-email
```

### 5. Test

```bash
ssh -T git@github-clientname
```

### 6. Clone repos using the alias

```bash
git clone git@github-clientname:client-org/repo-name.git ~/Developer/clientname/repo-name
```

---

## Troubleshooting

**"Permission denied (publickey)"**
- Wrong host alias in remote URL
- Public key not added to the right GitHub account
- Test with: `ssh -T git@github-youralias`

**Commits showing wrong email**
- Check: `git config user.email` inside the repo
- The `includeIf` path must end with `/` and match your folder structure exactly

**"Could not read from remote repository"**
- Verify remote: `git remote -v`
- Make sure it uses `git@github-youralias:` not `git@github.com:`

---

## File Reference

| File | Purpose |
|------|---------|
| `~/.ssh/id_<name>` | Private key (never share) |
| `~/.ssh/id_<name>.pub` | Public key (add to GitHub) |
| `~/.ssh/config` | Routes host aliases to keys |
| `~/.gitconfig` | Global git config with includeIf rules |
| `~/.gitconfig-<name>` | Per-directory email override |
