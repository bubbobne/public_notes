---
title: EndeavourOS — Setup & Cheatsheet
tags: [endeavouros, linux, arch, setup, system, configuration]
updated: 2025-11-06
---

# 🧩 EndeavourOS — Setup & Cheatsheet

Notes and useful commands for my **EndeavourOS (Arch-based)** workstation  
with **Ryzen 7 / 32 GB RAM** and **KDE Plasma** desktop environment.

---

## ⚙️ System Info

- **Distribution:** EndeavourOS (Arch Linux base, rolling release)
- **Desktop Environment:** KDE Plasma
- **Kernel:** `uname -r`
- **Installed kernels:** `pacman -Qs '^linux'`
- **Bootloader:** systemd-boot (default) or GRUB
- **Microcode update:**
  ```bash
  sudo pacman -S amd-ucode
  sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

---

## 📦 Package Management

### Pacman
```bash
sudo pacman -Syu             # full system upgrade
sudo pacman -S <pkg>         # install a package
sudo pacman -Rns <pkg>       # remove package + unused dependencies
sudo paccache -r             # clean package cache (pacman-contrib)
sudo pacman -Qdt             # list orphaned packages
```

### Yay (AUR helper)
```bash
yay -Syu                     # update system + AUR
yay -Ss <pkg>                # search package
yay -S <pkg>                 # install from AUR
yay -Rns <pkg>               # remove AUR package
yay -Yc                      # clean AUR cache
```

---

## 🧠 Systemd and Logs

```bash
systemctl status <service>
sudo systemctl enable --now <service>
sudo systemctl disable --now <service>
sudo systemctl restart <service>
```

### Boot and performance analysis
```bash
systemd-analyze blame                # list services by boot time
systemd-analyze critical-chain       # dependency tree
journalctl -b                        # logs from current boot
journalctl -u <service> -f           # follow logs for specific service
```

---

## 🧰 Zsh / Oh My Zsh Configuration

### Aliases in `~/.zshrc`
```zsh
# Pacman / Yay
alias pSyu='sudo pacman -Syu'
alias ySyu='yay -Syu'
alias pR='sudo pacman -Rns'
alias pQdt='sudo pacman -Qdt'

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -lh --group-directories-first --color=auto'

# Git
alias ga='git add -A'
alias gc='git commit -m'
alias gp='git push'
alias gl='git pull'
alias gs='git status'

# System
alias jbf='journalctl -b -f'
alias jser='journalctl -u'

# Cleanup
alias clean-cache='sudo paccache -rk2 && yay -Yc'
```

Reload after editing:
```bash
source ~/.zshrc
```

---

## 💾 Maintenance and Cleanup

```bash
sudo pacman -Rns $(pacman -Qdtq)    # remove orphan packages
sudo paccache -rk2                   # keep last 2 versions of each package
sudo journalctl --vacuum-time=7d     # delete logs older than 7 days
sudo fstrim -v /                     # TRIM SSD
```

### Remove old kernels
```bash
uname -r                             # check current kernel
sudo pacman -Rns linux-XYZ linux-XYZ-headers
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 🧩 Git & Vault Sync

```bash
cd ~/Vaults/SystemDocs
git add .
git commit -m "Update EndeavourOS setup"
git pull --rebase origin main
git push origin main
```

**Optional alias:**
```zsh
alias vault-update='cd ~/Vaults/SystemDocs && git add . && git commit -m "sync" && git pull --rebase && git push'
```

---

## 🧭 Troubleshooting Quick Reference

| Issue | Command |
|--------|----------|
| Pacman lock error | `sudo rm /var/lib/pacman/db.lck` |
| Cache full | `sudo paccache -r` |
| Slow boot | `systemd-analyze blame` |
| Regenerate GRUB config | `sudo grub-mkconfig -o /boot/grub/grub.cfg` |
| Service failed | `systemctl status <service>` / `journalctl -u <service>` |
