# Libvirt / QEMU Fix: Dependency Job Failed

If you encounter the error: `A dependency job for libvirtd.service failed` when installing or starting QEMU, it is likely due to a bug in `systemd-creds` (systemd 260+) failing to initialize encrypted secrets via the TPM.

## The Solution: Switch to Modular Daemons
Modern Arch Linux installations should use modular daemons instead of the legacy monolithic `libvirtd`. This bypasses the failing encryption service.

### 1. Disable Legacy Services
```bash
sudo systemctl disable --now libvirtd.service libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket
```

### 2. Enable Modular Services
```bash
sudo systemctl enable --now virtqemud.socket virtnetworkd.socket virtstoraged.socket
```

### 3. Verify the Fix
```bash
virsh -c qemu:///system uri
# Should return: qemu:///system
```

## Alternative: Fix the Systemd Host Key
If you MUST use the legacy `libvirtd.service`, you can try to fix the underlying crash by manually initializing the systemd host key:

```bash
sudo systemd-creds setup
sudo systemctl restart libvirtd.service
```

## Post-Fix Configuration
Ensure your user is in the `libvirt` group to avoid using `sudo` for every command:
```bash
sudo usermod -aG libvirt $(whoami)
# Log out and log back in for changes to take effect
```
