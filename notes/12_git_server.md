## Git Server

Running your own Git server is about owning your source of truth. Your repos live where you decide, under rules you set, at a pace you control. That means you decide who can read and write, how code moves to production, and how the system grows as your team and projects grow. It’s pure Git under the hood—no mystery platform layers—so you get predictability, auditability, and hooks you can bend into real-world workflows like auto-deploys, mirroring, and backups. If you’ve ever wanted a quiet, dependable place for code that you can lock down, wire up, and scale out, a bare Git server is exactly that.

```
+-------------------+              +-------------------+
|   Git Client 1    |              |   Git Client 2    |
| - Clone           |              | - Push            |
| - Pull            |              | - Pull Requests   |
| - Push            |              | - Merge           |
+--------+----------+              +----------+--------+
         |                                   |
         |                                   |
         +----+----------------------+-------+
              |                      |
              |    Git Operations    |
              |                      |
         +----v----------------------v----+
         |                                |
         |       Git Server (Bare)        |
         | - Central Repository Storage   |
         | - Access Control               |
         | - Version History Management   |
         | - Branches & Tags Handling     |
         |                                |
         +--------------------------------+
```

### Prerequisites 

Pick a Debian/Ubuntu box you trust. You’ll connect over SSH because it’s simple, secure, and scriptable. You’ll need admin rights once to set things up; after that, day-to-day work happens with regular user access. Think of the server as a quiet filesafe that only speaks Git over SSH—no fancy web UI required unless you want one later.

#### Commands

Update packages and install Git:

```bash
sudo apt update
sudo apt install git
```

Show the installed version:

```bash
git --version
# output (example):
# git version 2.43.0
```

### Create a place for repos

A bare repository is the central hub with no working directory—no editable files, just Git’s object store and refs. It’s perfect as a shared remote: everyone clones from it, pushes to it, and pulls from it. Since it’s not a checkout, nothing on the server gets “dirty,” and hooks can react to pushes cleanly.

Create a home for repos and initialize one:

```bash
sudo mkdir -p /opt/git/myrepo.git
sudo git init --bare /opt/git/myrepo.git
sudo tree -L 1 /opt/git/myrepo.git 2>/dev/null || ls -1 /opt/git/myrepo.git
# output (typical top-level contents):
# HEAD
# config
# description
# hooks
# info
# objects
# refs
```

Set ownership so the dedicated Git user (we’ll make it next) can write:

```bash
sudo chown -R git:git /opt/git
```

A quick mental map of a bare repo:

```
/opt/git/myrepo.git
├─ config        (repo settings)
├─ objects/      (all commits, trees, blobs)
├─ refs/         (branches, tags)
├─ hooks/        (scripts on push/receive)
└─ HEAD          (default branch pointer)
```

### Create a dedicated “git” user and lock it down

A separate user keeps permissions simple and tidy. You’ll log in as `git` using SSH keys, not passwords. You can go further and restrict the shell so this user can only run Git—handy on shared servers.

Create the user:

```bash
sudo adduser --disabled-password --gecos "" git
# set a password only if you really need one; keys are better
# to fully lock password logins:
sudo passwd -l git
```

Set up SSH keys for the git user:

```bash
sudo -u git mkdir -p ~git/.ssh
sudo -u git chmod 700 ~git/.ssh
sudo -u git touch ~git/.ssh/authorized_keys
sudo -u git chmod 600 ~git/.ssh/authorized_keys
```

Authorize a developer (paste their public key):

```bash
# Example key, replace with a real developer's public key:
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFAKEKEYEXAMPLEONLY1234567890abcdefg dev@laptop" | sudo tee -a ~git/.ssh/authorized_keys
# output:
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFAKEKEYEXAMPLEONLY1234567890abcdefg dev@laptop
```

Optional: restrict the git user to git-only commands:

```bash
which git-shell
# output (example):
# /usr/bin/git-shell
sudo chsh -s /usr/bin/git-shell git
```

ASCII view of the handshake:

```
[Developer Laptop] --(SSH key)--> [Server: user=git] --(authorized_keys)--> [myrepo.git]
```

### Use the repo from a developer machine

Developers treat the server like any other remote. Clone once, make changes locally, push/pull when needed. The remote URL points at the bare repo path.

Clone:

```bash
git clone git@yourserver:/opt/git/myrepo.git
# or: git clone ssh://git@yourserver/opt/git/myrepo.git
# output:
# Cloning into 'myrepo'...
# warning: You appear to have cloned an empty repository.
```

Make a first commit and push:

```bash
cd myrepo
echo "hello" > README.md
git add README.md
git commit -m "Initial commit"
# output (commit):
# [main (root-commit) 1a2b3c4] Initial commit
#  1 file changed, 1 insertion(+)
#  create mode 100644 README.md

git push origin main
# output:
# Enumerating objects: 3, done.
# Counting objects: 100% (3/3), done.
# Writing objects: 100% (3/3), 233 bytes | 233.00 KiB/s, done.
# Total 3 (delta 0), reused 0 (delta 0)
# To yourserver:/opt/git/myrepo.git
#  * [new branch]      main -> main
```

Pull later:

```bash
git pull origin main
# output:
# Already up to date.
```

A tiny commit graph, just to visualize branches:

```
A---B---C  main
 \       
  D---E   feature/login
```

### Practical use case 1: small team sharing a central remote

You’ve got two or more devs, each with their own laptops. Everyone clones the central bare repo, works on branches, and pushes to `origin`. Merges and reviews can happen locally or via lightweight conventions (e.g., “feature branches reviewed by pair buddy before merging”). No web app is required; it’s Git basics, fast and quiet.

Add a teammate’s remote (from their side it’s the same clone step). Typical day-to-day:

```bash
# create a feature branch
git switch -c feature/api

# work, commit
git add .
git commit -m "Add API endpoint"

# push branch to share
git push -u origin feature/api
# output:
# To yourserver:/opt/git/myrepo.git
#  * [new branch]      feature/api -> feature/api

# open a review however your team likes (chat, issue tracker, etc.)
# merge locally when approved
git switch main
git pull --ff-only
git merge --no-ff feature/api -m "Merge feature/api"
git push
```

### Practical use case 2: auto-deploy on push (post-receive hook)

When code lands on a specific branch, you can kick off a deployment script. With a bare repo, the server always sees pushes, so hooks are the perfect trigger.

Create a `post-receive` hook:

```bash
sudo -u git tee /opt/git/myrepo.git/hooks/post-receive >/dev/null <<'SH'
#!/usr/bin/env bash
set -euo pipefail

read oldrev newrev refname
branch=${refname#refs/heads/}

if [ "$branch" = "main" ]; then
  workdir=/var/www/myapp
  if [ ! -d "$workdir/.git" ]; then
    git --work-tree="$workdir" --git-dir=/opt/git/myrepo.git checkout -f main
  else
    git --work-tree="$workdir" --git-dir=/opt/git/myrepo.git fetch origin main
    git --work-tree="$workdir" --git-dir=/opt/git/myrepo.git checkout -f main
  fi
  # place build/restart steps here:
  # systemctl restart myapp.service
  echo "Deployed $(git --git-dir=/opt/git/myrepo.git rev-parse --short main) to $workdir"
fi
SH
sudo chmod +x /opt/git/myrepo.git/hooks/post-receive
```

Push to trigger:

```bash
git push origin main
# output (server hook echoes back):
# Deployed 9f3a1c2 to /var/www/myapp
```

Small mental picture of the flow:

```
dev -> push main ---> [bare repo hook] ---> sync files to /var/www/myapp ---> restart service
```

### Practical use case 3: mirror to GitHub/GitLab while keeping on-prem as source of truth

Keep private control but still use hosted services for CI or collaboration. The server can push to an external remote after each update.

Add a mirror remote on any clone (or via a server-side hook):

```bash
git remote add mirror git@github.com:org/myrepo.git
git push --mirror mirror
# output:
# * [new branch] main -> main
# * [new tag] v1.0 -> v1.0
```

Optional server-side hook snippet (append to `post-receive`):

```bash
# inside the if [ "$branch" = "main" ]; then block
git --git-dir=/opt/git/myrepo.git push --mirror mirror || echo "Mirror push failed"
```

### Security and hygiene that actually helps

Keep SSH key-only access. Keep the `git` user owning `/opt/git` and nothing else. Back up the bare repos like regular files; they’re self-contained. If you expose SSH to the internet, firewall it, fail2ban it, or tuck it behind a VPN. Updates for the OS and Git are boring and necessary; set a rhythm and stick to it.

Basic firewall (UFW) with SSH allowed:

```bash
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
# output (example):
# Status: active
# To                         Action      From
# --                         ------      ----
# OpenSSH                    ALLOW       Anywhere
# OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Back up your repos (rsync and tar both work great):

```bash
# rsync to another disk or host
sudo rsync -a --delete /opt/git/ /backup/git/

# tar snapshot (quick & simple)
sudo tar -C /opt -czf /backup/myrepos-$(date +%F).tar.gz git
```

Update the box:

```bash
sudo apt update && sudo apt upgrade -y
```

### Troubleshooting

Most hiccups are permission or key issues. If pushes fail with “permission denied (publickey)”, the server didn’t accept your key. If pushes fail with “non-fast-forward”, your branch is behind—pull or rebase, then push. If you accidentally `git init` inside the bare repo directory from a client, you’re in the wrong place—work in your clone, not on the server.

Check SSH key is being used:

```bash
ssh -v git@yourserver 'echo ok'
# output (look for):
# Offering public key: /home/dev/.ssh/id_ed25519
# Authenticated to yourserver (...)
# ok
```

Fix repo permissions:

```bash
sudo chown -R git:git /opt/git
sudo find /opt/git -type d -exec chmod 755 {} \;
sudo find /opt/git -type f -exec chmod 644 {} \;
sudo chmod -R 700 ~git/.ssh
sudo chmod 600 ~git/.ssh/authorized_keys
```

Resolve non-fast-forward:

```bash
git pull --rebase origin main
git push origin main
# output:
# Successfully rebased and updated refs/heads/main.
```

Verify the remote URL:

```bash
git remote -v
# output:
# origin  git@yourserver:/opt/git/myrepo.git (fetch)
# origin  git@yourserver:/opt/git/myrepo.git (push)
```

### Optional lightweight web UI

Sometimes you just want a simple window into your repos without adopting a whole platform. That’s what cgit and gitweb give you: a fast, read-only catalog of your bare repos with clickable history, diffs, blame, and tarball downloads. They don’t change your Git workflow at all; pushes still happen over SSH, and the web UI only reads from the same `/opt/git/*.git` directories. You can put it behind HTTPS and basic auth for internal eyes, expose it read-only to CI and stakeholders, or leave it on a private network. Think of it as a static-feeling dashboard backed by your live repos—zero database, tiny surface area, and easy to back up or blow away.

```
Browser
   |
   v
[ Nginx/Apache ] --(CGI/FastCGI)--> [ cgit/gitweb ] --(read-only)--> /opt/git/*.git
                                                         |
                                                         +-- renders commits, diffs, trees
```

cgit quick path on Debian/Ubuntu. You install cgit and a tiny CGI wrapper, point it at your repo directory, and let your web server hand off requests. The result is a snappy index page with per-repo views and RSS/Atom feeds you can wire into chat or dashboards.

```bash
# Install cgit (UI), fcgiwrap (runs CGI), and a web server (pick one)
sudo apt update
sudo apt install -y cgit fcgiwrap nginx apache2-utils

# Make sure your repos are where you expect
sudo ls /opt/git | head
# output (example):
# myrepo.git
# infra.git
# website.git
```

Tell cgit where to look and how to present things. The config is plain text; the important bit is `scan-path`, which walks a directory and treats `*.git` as repos. You can group by path segments, enable blame/diff links, and set a friendly title.

```bash
# /etc/cgitrc
sudo tee /etc/cgitrc >/dev/null <<'CFG'
# where to find bare repos
scan-path=/opt/git

# top-level branding
root-title=Company Git
root-desc=Fast, read-only views of our repositories

# organize list by folder names (e.g., libs/foo.git -> section "libs")
section-from-path=1

# useful features
enable-git-config=1
enable-index-owner=1
enable-index-links=1
enable-log-filecount=1
enable-log-linecount=1
enable-commit-graph=1
enable-blame=1
enable-diff=1

# cache to speed up large repos (tune for your box)
cache-size=512
CFG
```

Wire cgit into Nginx. The `fcgiwrap` service runs the CGI binary; Nginx forwards requests to it. You can mount it at `/cgit/` to keep things tidy. While you’re here, you can also enable smart HTTP cloning so read-only clones work over HTTPS in addition to SSH.

```bash
# Nginx site: /etc/nginx/sites-available/cgit
sudo tee /etc/nginx/sites-available/cgit >/dev/null <<'NGX'
server {
    listen 80;
    server_name _;

    # cgit UI at /cgit/
    location /cgit/ {
        root /usr/lib;
        fastcgi_param SCRIPT_FILENAME /usr/lib/cgit/cgit.cgi;
        fastcgi_param PATH_INFO $uri;
        include fastcgi_params;
        fastcgi_pass unix:/run/fcgiwrap.socket;
    }

    # static assets shipped with cgit (CSS, logo)
    location /cgit.css {
        root /usr/share/cgit;
    }
    location /cgit.png {
        root /usr/share/cgit;
    }

    # optional: smart HTTP for read-only clones via /git/
    # e.g., git clone http://server/git/myrepo.git
    location /git/ {
        root /opt;
        gzip off;
        include fastcgi_params;
        fastcgi_param GIT_HTTP_EXPORT_ALL "";
        fastcgi_param GIT_PROJECT_ROOT /opt/git;
        fastcgi_param PATH_INFO $uri;
        fastcgi_pass unix:/run/fcgiwrap.socket;
        # Make sure repos have git-daemon-export-ok or rely on GIT_HTTP_EXPORT_ALL above
    }
}
NGX

# Enable site and start services
sudo ln -s /etc/nginx/sites-available/cgit /etc/nginx/sites-enabled/cgit
sudo systemctl enable --now fcgiwrap
sudo systemctl reload nginx

# Quick smoke test
curl -I http://localhost/cgit/
# output:
# HTTP/1.1 200 OK
# Content-Type: text/html
```

Lock it down or open it up, your call. For internal-only, keep it on a private network or put simple basic auth in front; for public read-only, just serve it over HTTPS. Basic auth is enough for “only the team can see this,” while write access still flows over SSH keys to the bare repos.

```bash
# Create an htpasswd file for Nginx basic auth
sudo htpasswd -c /etc/nginx/.cgit.htpasswd alice
# (enter a password)
# Then add to the /cgit/ location block:
#   auth_basic "Restricted";
#   auth_basic_user_file /etc/nginx/.cgit.htpasswd;
sudo nginx -t && sudo systemctl reload nginx
```

GitWeb alternative if you prefer the classic look. Same idea—scan a directory of bare repos and render HTML—but with a slightly different aesthetic and defaults. It plays well with Apache out of the box and is perfectly fine for small to mid-sized setups.

```bash
sudo apt install -y gitweb apache2
# Point GitWeb at /opt/git
sudo sed -i 's|^\$projectroot =.*|$projectroot = "/opt/git";|' /etc/gitweb.conf

# Enable Apache config that ships with GitWeb
sudo a2enconf gitweb
sudo systemctl reload apache2

curl -I http://localhost/gitweb/
# output:
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=UTF-8
```

A couple of sanity checks and gotchas that save time. Make sure the web server user can read the repos but not write to them; world-readable is fine for public read-only, or keep it group-readable and add the web user to the `git` group for internal sites. If the page is blank, it’s usually a CGI miswire—verify `fcgiwrap` is running and the `SCRIPT_FILENAME` path points to `cgit.cgi`. If cloning over HTTP 404s, check the `/git/` location, `GIT_PROJECT_ROOT`, and that your repo is actually under `/opt/git` and ends with `.git`.

```bash
# Permissions for internal read-only (web server is in group 'git')
sudo chgrp -R git /opt/git
sudo find /opt/git -type d -exec chmod 755 {} \;
sudo find /opt/git -type f -exec chmod 644 {} \;
sudo usermod -aG git www-data
```

Once it’s up, it becomes a pleasant habit: paste a link to a commit in chat, skim a blame line during code review, let PMs browse the changelog without needing Git installed, and hook the feed URL into your release channel. It’s the low-maintenance way to make your on-prem Git feel visible without turning it into a whole new platform.

### Quick reference (do these in order)

I. Install Git, make a bare repo, ensure `git:git` owns it.

```bash
sudo apt install git
sudo mkdir -p /opt/git/myrepo.git
sudo git init --bare /opt/git/myrepo.git
sudo chown -R git:git /opt/git
```

II. Create the `git` user and load developer keys.

```bash
sudo adduser --disabled-password --gecos "" git
sudo -u git mkdir -p ~git/.ssh && sudo -u git chmod 700 ~git/.ssh
sudo -u git touch ~git/.ssh/authorized_keys && sudo -u git chmod 600 ~git/.ssh/authorized_keys
echo "ssh-ed25519 AAAA... dev@laptop" | sudo tee -a ~git/.ssh/authorized_keys
```

III. Clone and push from a dev machine.

```bash
git clone git@yourserver:/opt/git/myrepo.git
cd myrepo
echo "ok" > README.md && git add README.md && git commit -m "init" && git push -u origin main
```

A final mental model to carry around:

```
[devs & CI] <--> ssh://git@server/opt/git/*.git  (bare repos)
                    |
                    +--> hooks (deploy, mirror, notify)
                    |
                    +--> backups (rsync/tar)
```

That’s it. You’ve got a clean, self-hosted Git setup that’s easy to reason about, easy to secure, and easy to extend when you need more.
