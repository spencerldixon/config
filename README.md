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
    homebrew.yml            - Installs CLI tools, Treesitter/Telescope and shell prerequisites, Ruby build dependencies, Ghostty, Docker Desktop, AI CLIs, and a Nerd Font
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

## Dotfiles prerequisites

Run Xcode setup before Homebrew on a new machine. Treesitter compiles parsers
with the C compiler from the Xcode command line tools:

```sh
ansible-playbook site.yml --tags xcode,homebrew --ask-become-pass
```

`homebrew.yml` installs `tree-sitter-cli` (not just the Tree-sitter library),
`ripgrep` and `fd` for Telescope, `fzf`, `zsh-autosuggestions`, `direnv`, and
`thefuck` for the shell, `jq` for Claude's status line, and Node/npm for Pi.
macOS already supplies `curl`, `tar`, `pbcopy`, `osascript`, and the utilities
used by the tmux status scripts. Oh My Zsh and SCM Breeze are installed by
their own tasks; Spotify is installed by `apps.yml` for the tmux music display.

After the dotfiles are linked, install their mise-managed Ruby, Python, uv,
Terraform, and Yarn versions, then sync Neovim plugins and wait for parser
installation to finish:

```sh
mise install --yes
nvim --headless "+Lazy! sync" +qa
nvim --headless "+lua require('nvim-treesitter').install({ 'bash', 'c', 'css', 'diff', 'gitignore', 'html', 'htmldjango', 'javascript', 'json', 'lua', 'luadoc', 'make', 'markdown', 'markdown_inline', 'python', 'ruby', 'terraform', 'toml', 'typescript', 'vim', 'vimdoc' }):wait(300000)" +qa
```

The shell must activate the Homebrew `mise` executable via `mise activate zsh`;
`~/.local/bin/mise` only exists when mise was installed separately there.
Ensure Homebrew's `bin` directory is on the shell PATH using `brew shellenv`.

Install Pi separately using its installer; Node is its runtime prerequisite.
Pi's declared skills package installs at startup. Authenticate inside Pi with
`/login` and run `pi mcp login atlassian` for its remote MCP server.
Claude and Codex also require their own login; Codex's statusline TOML is merged
into its local configuration rather than Stowed.

Install tmux plugins with prefix + Shift-I. Select `MesloLGS Nerd Font Mono` in
your terminal if its existing font does not include the Neovim/tmux glyphs.
Start Docker Desktop once to initialize its engine and CLI integrations.
