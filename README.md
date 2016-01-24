# wemohue 

This is a set of two daemons and a utility script for controlling Hue lights
with a WeMo light switch.

You must wire the switch to not interrupt power to the light fixture, the switch
and the fixture must get continuous power. This is described in [this WEMO
Community
thread](http://community.wemothat.com/t5/WEMO-Application/Phillips-Hue-WeMo-LIght-Switch-and-IFTTT/td-p/6888
"WEMO Community Thread About Hue Lights").

## Config File

Before installing, make a config file as `~/.wemohue/config.yml` that looks like this:

```yaml
hue_bridge: rafael_living_hue
wemo_switch: rafael_living_wemo
```

`hue_bridge` should be the hostname or IP address of your Hue bridge. You
should create a static lease for it in your router interface.

`wemo_switch` is the name you gave your WeMo switch in the WeMo app, it is not
necessarily the hostname, it will be found by ouimeaux using UPnP discovery.

## Linux Installation

```bash
sudo apt-get update
sudo apt-get install apache2-utils
pip install --user --upgrade phue
pip install --user --upgrade ouimeaux
mkdir ~/bin ~/run ~/log ~/src
cd ~/src
git clone https://github.com/rkitover/wemohue.git wemohue
cd wemohue
cp toggle_hue* wemo* ~/bin
chmod +x ~/bin/toggle_hue ~/bin/toggle_hue_daemon ~/bin/wemo_switch_daemon
```

Run `crontab -e` to add the following entries, if you don't know how to use vim
run `EDITOR=nano crontab -e` instead:

```crontab
@reboot     (/usr/bin/python ~/bin/wemo_switch_daemon 2>&1 | rotatelogs -n 5 ~/logs/wemo_switch_daemon.log 86400) &
@reboot     (/usr/bin/python ~/bin/toggle_hue_daemon  2>&1 | rotatelogs -n 5 ~/logs/toggle_hue_daemon.log  86400) &
```

Make sure `~/bin` is in your PATH, which you can set in `~/.bashrc`.

## Mac Installation

Make sure [Homebrew](http://brew.sh "Mac Homebrew") and the Homebrew python are
both installed and your PATH is correctly set up to use them.

```bash
pip install --user --upgrade phue
pip install --user --upgrade ouimeaux
mkdir ~/bin ~/run ~/log ~/src
cd ~/src
git clone https://github.com/rkitover/wemohue.git wemohue
cd wemohue
cp toggle_hue* wemo* ~/bin
chmod +x ~/bin/toggle_hue ~/bin/toggle_hue_daemon ~/bin/wemo_switch_daemon
cp launchctl/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/com.magneto.wemoswitchdaemon.plist
launchctl load ~/Library/LaunchAgents/com.magneto.togglehuedaemon.plist
```

*NOTE:* you cannot run launchctl commands in tmux unless you are using
`reattach-to-user-namespace`.

Make sure `~/bin` is in your PATH, which you can set in `~/.bashrc`.

## Toggle Script

The `toggle_hue` script will toggle the lights from on to off and from off to
on, and make the switch reflect the stgate of the lights. If you need to
explicitly set the lights to on or off from another script, program or
whatever, you can pass an `on` or `off` parameter, e.g. `toggle_hue on` or
`toggle_hue off`.

It also takes a `toggle` parameter which is the same as no parameter at all.
