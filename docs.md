# preparation

```
sudo pacman -S podman fuse-overlayfs crun qemu-base slirp4netns
```

# Configuration

```
sudo nvim /etc/sysctl.d/00-local.conf
```
add value below

```
kernel.unprivileged_userns_clone=1
```
