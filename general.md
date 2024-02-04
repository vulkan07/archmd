# General Maintenance stuff for Arch
## 1. Disk Usage
- Use `df -H` to check the available space left on the partitions 
- Use `ncdu` cli prgram to check what folders and files are the main culprits
### Yay
- `yay -Scc` to clean package cache
- `yay -Rcns $(yay -Qtdq)` to remove old unused packages
### Journalctl
- `sudo journalctl --vacuum-size=<...>` to limit system logs sizes (i.e. `50M` or `2weeks`)
- To make cleaning automatic, set `SystemMaxUse=` in `/etc/systemd/journald.conf`
