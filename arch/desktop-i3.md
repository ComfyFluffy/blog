# Desktop with i3

This article tells about configurating i3 with [example images](https://github.com/someoneinjd/dotfiles).

## Installation

### Desktop Install

Install WM, fonts, terminal, browser and utils:

```
# pacman -S i3-gaps polybar xorg xorg-xinit alsa-utils kitty ttf-cascadia-code noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra xorg-xset firefox
```

Modify `/etc/X11/xinit/xinitrc`:

```properties
...
# twm &
# xclock -geometry 50x50-1+1 &
# xterm -geometry 80x50+494+51 &
# xterm -geometry 80x20+494-0 &
# exec xterm -geometry 80x66+0+0 -name login
exec i3
```

Exit `root` and log into `user` to create `~/.config/fontconfig/fonts.conf`:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<alias>
   <family>monospace</family>
   <prefer>
     <family>Cascadia Code</family>
   </prefer>
 </alias>
</fontconfig>
```

Create `~/.xinitrc`:

```bash
xrdb-merge ~/.Xresources

# HiDPI configuration
export QT_AUTO_SCREEN_SCALE_FACTOR=1
export GDK_DPI_SCALE=1.5

exec i3
```

Create `~/.Xresources`:

```properties
# HiDPI configuration
Xft.dpi: 144
```

Now we can start the desktop:

```
$ startx
```
