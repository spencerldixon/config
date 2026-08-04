# 💻 config

Ansible playbooks to configure my macbook.

Development environment + daily tooling + security hardening — split into two independently runnable playbooks, chained together by `site.yml`.

## Structure

```
config/
  ansible.cfg           - tells ansible where inventory lives, local settings
  inventory.ini          - list of machines to run against (just localhost, this is a laptop setup)
  requirements.yml        - external ansible collections needed (community.general, for homebrew/git_config modules)
  site.yml               - runs development.yml + security.yml in order
  development.yml         - dev environment + daily tooling playbook
  security.yml             - hardening playbook (macOS + Firefox)
  files/
    gitignore_global        - global gitignore, copied to ~/.gitignore_global
    firefox-policies.json    - Firefox enterprise policy (hardening + forced extensions), copied into Firefox.app
  tasks/
    xcode.yml               - installs Xcode command line tools, accepts license
    homebrew.yml             - installs Homebrew + core CLI packages (mise, git, neovim, stow, tmux, openssl, wget, ghostty, docker, gh)
    apps.yml                  - installs GUI apps via Homebrew cask (firefox, slack, spotify, rectangle, transmission, vlc, tor-browser, notion, anki)
    git.yml                    - sets global git config (user, email, editor, push default, global gitignore)
    dotfiles.yml                - clones dotfiles repo, stows packages, bootstraps neovim plugins via lazy.nvim
    zsh.yml                      - installs zsh + oh-my-zsh, sets as default shell
    scm_breeze.yml                 - clones + installs scm_breeze
    security.yml                    - macOS hardening (FileVault, firewall, Gatekeeper, SSH, guest account, etc), based on drduh's OS X Security and Privacy Guide
    firefox.yml                      - installs policies.json (telemetry off, tracking protection, forced extensions: uBlock Origin, Privacy Badger, LastPass, Instapaper, FoxyProxy, Facebook/Google Container)
    osx.yml                           - cosmetic macOS defaults (dock size, key repeat, dark mode) — not wired into either playbook yet
```

`development.yml` and `security.yml` are each a single play (`hosts: local`) that pulls in task files via `include_tasks`, tagged to match the filename. `site.yml` chains both via `import_playbook`.

## Setup (one time)

Install ansible itself, then pull in the collection the homebrew task needs:

```sh
brew install ansible
cd config
ansible-galaxy collection install -r requirements.yml
```

## Run everything

```sh
cd config
ansible-playbook site.yml
```

## Run one playbook only

```sh
cd config
ansible-playbook development.yml
ansible-playbook security.yml
```

## Run one task only

Use the tag matching the task file name, against whichever playbook contains it:

```sh
cd config
ansible-playbook development.yml --tags xcode
ansible-playbook development.yml --tags homebrew
ansible-playbook development.yml --tags dotfiles
ansible-playbook security.yml --tags security
ansible-playbook security.yml --tags firefox
```

## Notes

- Everything targets `localhost` (`ansible_connection=local` in `inventory.ini`) — no remote hosts, no SSH.
- Tasks are idempotent: safe to re-run, already-installed stuff gets skipped.
- Add a new task file → drop it in `tasks/`, wire it into `development.yml` or `security.yml` (whichever concern it fits) with its own tag.
- `xcode.yml` accepts the Xcode license, needs sudo. Run with `--ask-become-pass` if you don't have passwordless sudo:
  ```sh
  ansible-playbook development.yml --tags xcode --ask-become-pass
  ```
- `firefox-policies.json` lives inside the `Firefox.app` bundle once copied there. Firefox self-updates replace the whole bundle and wipe it — re-run `ansible-playbook security.yml --tags firefox` after a Firefox update.
