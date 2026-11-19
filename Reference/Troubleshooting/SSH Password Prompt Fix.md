---
tags:
  - ssh
  - ssh-add
  - password
  - obsidian
---
# Fix Obsidian Git SSH Passphrase Prompts

*(Replace `id_ed25519` with `id_rsa` or your custom key filename if needed).*

## macOS

```bash
# 1. Add key to Apple Keychain
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# 2. Add config to ~/.ssh/config
cat << 'EOF' >> ~/.ssh/config

Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF
```

## Windows (PowerShell)

```powershell
# 1. Enable and start the SSH Agent service (Run as Administrator)
Set-Service -Name 'ssh-agent' -StartupType Automatic
Start-Service ssh-agent

# 2. Add key to the agent (Regular PowerShell)
ssh-add \$env:USERPROFILE\.ssh\id_ed25519

# 3. Point Git to Windows OpenSSH
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

## Linux

```bash
# 1. Start agent and add key for current session
eval "\$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 2. Auto-load key on login (add to ~/.bashrc or ~/.zshrc)
cat << 'EOF' >> ~/.bashrc

if [ -z "\$SSH_AUTH_SOCK" ]; then
   eval "\$(ssh-agent -s)" > /dev/null
   ssh-add ~/.ssh/id_ed25519 2>/dev/null
fi
EOF
```
