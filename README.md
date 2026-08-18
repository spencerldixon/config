# 💻 config

Ansible playbooks to configure my macbook with preferred settings and tooling.

## Install

Install ansible itself, then pull in the collection the homebrew task needs:

```sh
brew install ansible
cd config
ansible-galaxy collection install -r requirements.yml
```

## Run playbook

This will install everything

```sh
ansible-playbook site.yml --ask-become-pass
```

## Run a single task

```sh
ansible-playbook site.yml --tags xcode
```

## Structure

```
config/
  ansible.cfg
  inventory.ini
  requirements.yml
  site.yml                  - Main playbook for all tasks 
  files/
    gitignore_global        - Global gitignore, copied to ~/.gitignore_global
    firefox-policies.json   - Firefox settings file (hardening, minimal Home screen, GPC, and managed extensions)
  tasks/
    xcode.yml               - Installs Xcode command line tools, accepts license
    homebrew.yml            - Installs Homebrew packages (mise, git, neovim, stow, tmux, openssl, wget, ghostty, docker, gh)
    apps.yml                - Installs favourite mac apps via Homebrew cask (firefox, slack, spotify, rectangle, transmission, vlc, tor-browser, notion, anki)
    git.yml                 - Sets global git config (user, email, editor, push/default branch behaviour, global gitignore)
    dotfiles.yml            - Clones dotfiles repo, stows packages, bootstraps neovim plugins via lazy.nvim
    zsh.yml                 - Installs zsh and oh-my-zsh, sets as default shell
    scm_breeze.yml          - Installs scm_breeze for git shortcuts
    security.yml            - MacOS hardening (FileVault, firewall, Gatekeeper, SSH, guest account, etc), based on drduh's OS X Security and Privacy Guide
    firefox.yml             - Configures Firefox settings with hardening and extensions (survives Firefox self-updates)
    osx.yml                 - MacOS defaults (dock size, key repeat, dark mode)
```

## Notes

- `xcode.yml` accepts the Xcode license, needs sudo. Run with `--ask-become-pass` if you don't have passwordless sudo:
  ```sh
  ansible-playbook development.yml --tags xcode --ask-become-pass
- `firefox.yml` merges `files/firefox-policies.json` into the `org.mozilla.firefox` preference domain, preserving unrelated values. It only runs the import when managed policy values differ and survives Firefox self-updates.
