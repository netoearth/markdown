# Screen sharing on Hyprland (Arch Linux)

This guide will go through the setup and troubleshooting steps required to get screen sharing working on OBS with the [Hyprland](https://github.com/vaxerski/Hyprland) compositor.

**WARNING:** This guide demands that the user has a basic understanding on installing packages, using git, compiling from source, and some patience.

## Installing PipeWire and friends
First, we need to install `pipewire` and `wireplumber`. To do that, run the following command:
```
sudo pacman -S pipewire wireplumber
```

## Installing xdg-desktop-portal-wlr
Now we need to install the `xdg-desktop-portal-wlr` package. Run the following command:
```
sudo pacman -S xdg-desktop-portal-wlr
```

Make sure you don't have any other `xdg-desktop-portal-*` packages installed, as this could cause problems later. To check if you have any, run the following command:
```
pacman -Q | grep xdg-desktop-portal-
```
then delete all of the listed packages except the `-wlr` one we just installed.

Also make sure to install the optional dependencies `grim` and `slurp`:
```
sudo pacman -S grim slurp
```

## Editing the configuration file
Open your Hyprland config file (usually located at `~/.config/hypr/hyprland.conf`) with a text editor, and add the following line at the end:
```
exec-once=dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
```
This will make sure that `xdg-desktop-portal-wlr` can get the required variables on startup.

## Restart your session
Now log out and then log back in, or reboot your system. Once you're back into Hyprland, try sharing in OBS with the "Screen Capture (PipeWire)" option.

If it works, congratulations! You may exit this guide now. If not, follow along:

## Troubleshooting PipeWire
Make sure both `pipewire` and `wireplumber` are running. Run the following command:
```
systemctl --user status pipewire wireplumber
```
This should return "active (running)" for both. If it doesn't, follow the [Arch Wiki article](https://wiki.archlinux.org/title/PipeWire) for troubleshooting.

## Troubleshooting xdg-desktop-portal-wlr
Run the following command to check for errors in `xdg-desktop-portal-wlr`:
```
systemctl --user status xdg-desktop-portal-wlr
```
This should return "active (running)". If it doesn't, check the errors it gives you. Here's some workarounds for common errors:

### `Portal service (wlroots implementation) was skipped because of a failed condition check`
Make sure you added the required line to the Hyprland configuration. Check for typos.

### `unsupported wl_shm format 0x34324742` (after trying to share the screen on NVIDIA)
We'll need to modify the `wlroots` submodule source code to make it choose a viable format.

To do this manually, clone the [Hyprland](https://github.com/vaxerski/Hyprland) repo if you haven't already:
```
git clone --recursive https://github.com/vaxerski/Hyprland
```
Then, install all required dependencies for building, as specified in the wiki:
```
sudo pacman -S gdb ninja gcc cmake libxcb xcb-proto xcb-util xcb-util-keysyms libxfixes libx11 libxcomposite xorg-xinput libxrender pixman wayland-protocols cairo pango --needed
```
Open the `wlroots/types/output/render.c` file, look for the `wlr_output_preferred_read_format`, and replace it with the following:
```
uint32_t wlr_output_preferred_read_format(struct wlr_output *output) {
    return DRM_FORMAT_XRGB8888;
}
```
Afterwards, compile and install by running `sudo make install`, then restart your session.

Alternatively, you can install the modified [hyprland-nvidia-git](https://aur.archlinux.org/packages/hyprland-nvidia-git) AUR package.

## Troubleshooting OBS
You can try the [wlrobs-hg](https://aur.archlinux.org/packages/wlrobs-hg) plugin.

## Troubleshooting your browser
If you're trying to share your screen in your browser, first make sure that it supports Wayland and screen sharing on it, else it will not work.

### Firefox
Make sure you're running on Wayland. To do this, go to `about:support`, look for "Window Protocol", and make sure it is set to "wayland".
If it isn't, set the environment variable `MOZ_ENABLE_WAYLAND=1` in an environment file, like `/etc/environment`, `/etc/profile`, `~/.bash_profile`, `~/.zshenv`, etc.

If it still doesn't work, open `about:config` and make sure the `media.peerconnection.enabled` flag is set to true.

### Chromium
To run on Wayland, set the `--ozone-platform-hint=auto` flag in the chromium flags file (`~/.config/chromium-flags.conf` for the default package, `~/.config/chrome-flags.conf` for Chrome).

Also ensure the "WebRTC PipeWire support" flag is enabled by opening `chrome://flags/#enable-webrtc-pipewire-capturer` and enabling it.

## Troubleshooting your app
Make sure the app you're trying to share the screen with supports Wayland and sharing the screen with it.

Notable programs that still don't have full support for this are Discord and Zoom.

## It still doesn't work!
Follow the [official xdg-desktop-portal-wlr troubleshooting guide](https://github.com/emersion/xdg-desktop-portal-wlr/wiki/"It-doesn't-work"-Troubleshooting-Checklist).

If it still doesn't work, consider opening an issue.

## References
* <https://wiki.archlinux.org/title/PipeWire>
* <https://github.com/emersion/xdg-desktop-portal-wlr/issues/216>
* <https://github.com/emersion/xdg-desktop-portal-wlr/issues/190#issuecomment-1144287165>
* <https://github.com/emersion/xdg-desktop-portal-wlr/wiki>
* <https://wiki.archlinux.org/title/Firefox#Wayland>
* <https://wiki.archlinux.org/title/chromium#Native_Wayland_support>
