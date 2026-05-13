# preparation

```
git clone https://aur.archlinux.org/gvproxy-bin.git
```
```
cd gvproxy-bin
```
```
makepkg -rsi
```
```
sudo pacman -S podman fuse-overlayfs crun qemu-base 
```

# Configuration

```
sudo nvim /etc/sysctl.d/00-local.conf
```
add value below

```
kernel.unprivileged_userns_clone=1
```
```
sudo mv /usr/bin/gvproxy /usr/lib/podman/gvproxy 2>/dev/null || sudo cp $(which gvproxy) /usr/lib/podman/gvproxy
```
```
nvim ~/.config/containers/containers.conf
```
```
[engine]
helper_binaries_dir = ["/usr/lib/podman", "/usr/libexec/podman"]
```
