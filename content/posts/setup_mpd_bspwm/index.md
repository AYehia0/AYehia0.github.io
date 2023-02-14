---
title: "Full MPD setup on BSPWM + Polybar"
date: 2022-11-23T21:24:34+02:00
draft: true
toc: 
    enable: true 
    auto: false
tags: [mpd, polybar, bspwm, linux]
categories: [Linux]
featuredImagePreview: "featured-image.png"
featuredImage: "featured-image.png"
---

## What is mpd ?
MPD (Music Player Daemon) is an audio player, used to play audio tracks, organize playlists with some database for musics. In order to interface with it, a client is needed (in our case (ncmpcpp)[https://rybczak.net/ncmpcpp/] is used).

## Installation
- Install mpd and ncmpcpp on ArchLinux : `sudo pacman -S mpd ncmpcpp`
- Install mpDris2 (provides MPRIS 2 support to mpd): `yay -S mpdris2`
## Setup
### Autostart
There are multiple ways to auto-run `mpd and mpdris2` :
- Autostart with systemd : `systemctl enable/start --user mpd/mpDris2`
- Add to your `.xinitrc`
## Configuration
### MPD
In order for MPD to be able to playback audio, ALSA, optionally with PulseAudio, must be set up and working, be default 
Since we're going with per-user setup, the default path for the config file is : `~/.config/mpd/mpd.conf`
These are some of the most commonly used configuration options (we're not going to use them all):

- `pid_file` - The file where MPD stores its process ID
- `db_file` - The music database
- `state_file` - MPD's current state is noted here
- `playlist_directory` - The directory where playlists are saved into
- `music_directory` - The directory that MPD scans for music
- `sticker_file` - The sticker database
- `auto_update_depth` - The depth of the directories it should scan, "0" means all.

To make the config work, a `~/.mpd/` must be created and `~/.mpd/playlists/`.

```bash
music_directory             "~/Music"
playlist_directory          "~/.mpd/playlists"
db_file                     "~/.mpd/database"
log_file                    "~/.mpd/log"
pid_file                    "~/.mpd/pid"
state_file                  "~/.mpd/state"
sticker_file                "~/.mpd/sticker.sql"
restore_paused "yes"
auto_update	"yes"
auto_update_depth "0"
audio_output {
    type                    "pulse"
    name                    "pulse audio"
}
audio_output { 
    type "fifo" 
    name "my_fifo" 
    path "/tmp/mpd.fifo" 
    format "44100:16:2" 
}
```
### ncmpcpp

{{< image src="ncmpcpp.gif" alt="stats" position="center" style="border-radius: 8px;" >}}

ncmpcpp is a very cool TUI with some features like regular expressions for library searches, extended song format, items filtering, the ability to sort playlists, and a local filesystem browser.
To use it, a functional mpd must be present on the system since ncmpcpp/mpd work together in a client/server relationship (that's why it should run on the same port as mpd).

```
mpd_host = "127.0.0.1"
mpd_port = 6600
mpd_connection_timeout = "5"  
mpd_crossfade_time = "5"
mpd_music_dir = ~/Music

# General
colors_enabled          = "yes"
enable_window_title     = "yes"
main_window_color       = "default"
execute_on_song_change  = notify-send "Playing.." "$(mpc --format '%title% \n%artist%' current)"
autocenter_mode         = "yes"
centered_cursor         = "yes"
user_interface          = "classic"

# Progess Bar
progressbar_look = "━━╸"
progressbar_color = "white"
progressbar_elapsed_color="green"

# UI Visibility
header_visibility = "no"
statusbar_visibility = "no"
titles_visibility = "no"
startup_screen = "playlist"
#startup_slave_screen = "visualizer"
locked_screen_width_part = 50
ask_for_locked_screen_width_part = no

# UI Appearance
now_playing_prefix = "$b$3"
now_playing_suffix = "$/b$9"
song_status_format = "$7%t"
song_list_format = "$8%a - %t$R  $5%l"
song_columns_list_format = "(3f)[green]{} (60)[magenta]{t|f:Title} (1)[]{}"
song_library_format = {{ %a - %t } (%b)}|{%f} 
song_window_title_format = "Music"

# Visualizer
visualizer_data_source = "/tmp/mpd.fifo"
visualizer_output_name = "my_fifo"
visualizer_in_stereo = "yes"
visualizer_type = "ellipse"
visualizer_look = ●●
visualizer_color = "33,39,63,75,81,99,117,153,189"
visualizer_fps = "60"
```

Notes: 
- On `execute_on_song_change`: `mpc` is used to get some metadata about the running track.

### Polybar

To show the current music and get some basic music controlling on the polybar, we need to customize the polybar to add the new changes:

1. Create a module to open up ncmpcpp when clicked:

```bash
[module/music-player]
type = custom/text
content = ""
content-foreground = ${color.purple}
click-right = termite -t musicplayer -e "ncmpcpp"
```

2. Create a module to sync with mpd and show current playing song.

```bash
[module/mpd]
type                    = internal/mpd
host                    = 127.0.0.1
port                    = 6600
interval                = 2
format-offline          = ""
label-song              = "%artist% - %title%"
label-song-maxlen       = 40
icon-repeat             = ""

toggle-on-foreground    = ${color.green}
toggle-off-foreground   = ${color.red}
```

3. Create a module to add basic music control.

```bash
[module/mpd_control]
type                    = internal/mpd
host                    = 127.0.0.1
port                    = 6600
interval                = 1
format-online           = <icon-prev><toggle><icon-next>
format-offline          = <label-offline>
label-offline           = "󰝛 Offline"
icon-play               = " %{T2} "
icon-pause              = " %{T2} "
icon-stop               = " %{T2} "
icon-prev               = "%{T2} "
icon-next               = " %{T2}"

format-offline-foreground = ${color.grey}
icon-play-foreground    = ${color.lime}
icon-pause-foreground   = ${color.lime}
icon-stop-foreground    = ${color.lime}
icon-prev-foreground    = ${color.blue}
icon-next-foreground    = ${color.blue}
toggle-on-foreground    = ${color.green}
toggle-off-foreground   = ${color.red}
```

4. Putting all together.

{{< image src="polybar.png" alt="stats" position="center" style="border-radius: 8px;" >}}

It's up to you on how to add to the bar, but for me I am having it as a separate bar section.
```bash
[bar/section2]
monitor-strict = false
override-redirect = false
bottom = false
fixed-center = true
width = 21.5%
height = 40
offset-x = 43.5%
offset-y = 10
background = ${color.bg-alt}
foreground = ${color.fg}
radius = 6
line-size = 2
line-color = ${color.blue}
border-size = 2
border-color = ${color.bg}
padding = 2
module-margin-left = 0
module-margin-right = 0
font-0 = "banana:size=6;2"
font-1 = "Font Awesome 6 Free Solid:size=10;3"
modules-left = music-player
modules-center = mpd
modules-right = mpd_control
separator =
dim-value = 1.0
cursor-click = pointer 
cursor-scroll = ns-resize
```

### mpDris2

To test if mpdris2 works try to stop the current track using playerctl as : `playerctl stop -p "mpd"`, if everything works fine you can create some keybindings to control mpd.

In my case I am using sxhkd:
```bash
# pause everything
super + m
    pactl set-sink-mute 0 toggle

super + shift + p
    playerctl -p "mpd" play-pause
```
