---
published: true
title: "Windows Productivity Max"
date: 2018-11-26T00:00:00.000Z
categories: ["article"]
tags: ["windows", "cmder", "ricing"]
---
## Setup Windows 10 for Productivity Maxx.

_I use Arch btw_
But, When I am not doing shenanigans on my Arch (AntergOS) with KDE setup (which i absolutely love), I use Windows 10. Which is very useable (atleast, for me) contrary to popular linux chad opinion.

### Some Points

- Windows 10 Home came preinstalled on my laptop (Dell 7567).
- I like using Windows 10 (not as much as the KDE setup, though).
- My HDMI Monitor works without problems.
- My NVIDIA GPU works, both for PyTorch and Gaming.
- good battery life, though, using bumblebee on linux makes battery life just as good, if not better, on linux too.

### My Setup

I use Windows 10 Home Edition, which came preinstalled on my Dell 7567 dual booting with AntergOS runnning KDE Plasma 5. A Monitor (Dell S2240L) connected to my laptop and keep it on power most of the time, which imo I should stop doing and make sure its getting proper charge and discharge cycles. Connecting it to an external keyboard and mouse essentially makes it a `Dual Monitor Gaming PC B|` setup. Instead of having my Laptop's screen as the primary monitor, switching to the bigger 21inch monitor as the primary screen makes it very useful, for both working and playing games.

<center><img src='https://i.imgur.com/aQGGbBW.jpg' width='80%' alt='my precious'></center>

My setup on a random night.

### Improve Windows!

These are some things I believe would help improve your experience on Windows.

- **Enable Dark Mode.**

Not really the dark mode you wanted, still, better than nothing. 

- Go to `Settings->Personalization->Colors`

- Scroll to the botom and select dark mode.

This does not the turn the windows explorer to dark, but atleast makes the system settings dark and windows store dark (and maybe more apps i dont know of).

- **Decrease scaling.**

On a 1080p on both my laptop and the external monitor, the default 125% scaling option is just harsh, and well, very big. I believe decreasing it to 100% makes it much more bearable and makes the system look less cluttered.

- Right click on the desktop and Select `Display Settings` at the end of the list.

<center><img src='https://i.imgur.com/YYUjE1C.png' width='80%' height="30%" alt='Less Eye Gouging Options.'></center>

- Change the scaling to 100%.

<img src='https://i.imgur.com/WYeVBhY.png' width='80%' alt='100% will do again.'>

- **Smaller Taskbar Buttons**

The taskbar is uuuge, by default. Change this to smaller taskbar buttons and enjoy that space in your taskbar.

- Go to `Settings->Personalization->Taskbar`

- Swipe the button titled `Use Small Taskbar Buttons`

<img src='https://i.imgur.com/a6jXPI9.png' width='80%' alt='100% will do again.'>

- **Get bash back.**

Install the magical Windows Subsytem for Linux on your Windows 10 machine right now! Get most of what a Linux distribution can do right on your Windows machine without a heavy VM.

- Go to the Microsoft Store. (Search `Store` in the Search bar)

- Search for `WSL` to get all your option of OS'es listed, which include vanilla Debian, Kali Linux and Ubuntu. yep, the famous kali lincox right on windows.

- Install any of them and follow the simple process to setup your user on it. With the included terminal you can access the beautiful subsystem, but its meh, really. 

<img src='https://i.imgur.com/4U68KvT.png' width='80%' alt='lincoxes, lincoxes everywhere'>

It will feel like the perfect system, you can even run GUI apps using an XServer, it is still limited, but for running most command line tools, it works great.

- **Better Command Line and terminals with Cmder!**

Everyone loves their terminals, but using the default CMD prompt, Powershell or even the WSL Shell on Windows, is, _unpleasant_.

<!-- <img src='https://i.imgur.com/ZPbprPZ.png' width='80%' alt='All The Shells are Mine.'> -->
![All The Shells Are Mine](https://i.imgur.com/ZPbprPZ.png)

There exists many terminal apps on Windows, Hyper, ConEmu, Cmder, are some I tried.

[Cmder](https://github.com/cmderdev/cmder) turned out to be the most beautiful piece of all them. Its a fork of ConEmu, but with more niceties.

- Download Cmder's latest release of `cmder.zip` from [https://github.com/cmderdev/cmder/releases/](https://github.com/cmderdev/cmder/releases/).

- Extract it into somewhere where it won't get deleted.

- Pin the Cmder application to your taskbar for super fast access.

- Run it and enjoy the CMD Prompt goodness.

<img src='https://i.imgur.com/ZeHFXwq.png' width='80%' alt='Good Cmder.'>

You can also minimize/maximize your active Cmder Windows Quake/Guake style using `Ctrl-~`.

- **Setup Terminals and Shortcuts in Cmder**

These are some shortcuts I have setup which help me go around Cmder faster.

**Terminals**

- Open Cmder

- Open the settings either from the bottom right hamburger or by press `Win-Alt-P`

- Select `Startup->Tasks`.
<img src='https://i.imgur.com/Kc6H0sI.png' width='80%' alt='Cmder bby'>

- Order the terminals (to your liking or as i have set them up),
	- Bash
    - Cmd prompt as Admin
    - Powershell as Admin
- Select each item in the list and change the hotkeys (to your liking or what i prefer)
	- Bash := `Ctrl-Alt-B`
    - Cmd := `Ctrl-Alt-C`
    - Powershell := `Ctrl-Alt-V`
 (Other smooth combinations could be `Ctrl-Shift-1/2/3`, or `Ctrl-1/2/3`)
 
 **Killing processes**
 
 - Select `Keys And Macros` in the `Settings` Window.
 
 -  Search `Terminate` in the Search bar, select `Terminate (kill) all but shell processes in the current console: Close(10,1)`
 
 - Change the hotkey (to your liking or what I prefer)
 	- Terminate Window := `Win-Shift-Delete`
    
**Splitting Panes**

This duplicates the current terminal into either a right split or a bottom split. Super useful, use it on linux all the time.

- Search `Split: duplicate` in the `Keys and Macros` search bar.
- Set the hotkey for bottom and right splits (to your liking or what I prefer).
	- Down Split := `Ctrl-Shift-D' (for this you'll have to clear the existing shortcut for it)
    - Right Split := `Ctrl-Shift-R`
        
With this setup you can use splitted Powershells, Cmd prompts, WSL Shells, all together at the same time.

<img src='https://i.imgur.com/FUz7tJX.png' width='80%' alt='oh yea bby'>

- **The Text Editor: Visual Studio Code, my opinion**

just my choice of text editor, please don't kill me.

Visual Studio Code is a beautiful text editor, Powered by JalebiScript, its not the fastest editor around but it gets my work done. It has tons of extensions, themes, customizable shortcuts, portable configs, debugging features. You can even program your Arduino's right out of Visual Studio Code.

- Download Visual Studio Code and install it from here [https://code.visualstudio.com/](https://code.visualstudio.com/)

Apart from installing Extensions for languages of your choice, here are some customizations I like.

**Theme of Choice**
The `Cobalt 2` theme.

- Open the Extensions tab using either the big column on the left or pressing `Ctrl-Shift-X`.

- Search for `Cobalt 2` in the search bar and install it.

- Enable the theme, Press `Ctrl-Shift-P` to open the command omnibox, search `Theme`, select it, and select your choice of theme (or Cobalt 2) from the list of themes.

**Gitlens**
VSCode already has a great git interface, GitLens is an extension which makes it even better allowing you to checkout remote branches and commits, git blame, and more.

- In the `Extensions` tab, Search `GitLens`.

- Install it and enable it.

- ??
 
- Profit.

PS. Add `.vscode` to your `.gitignore` to prevent your folder config moving to your git repo.

Hope you end up writing beautiful code and tests for them in this shiny editor this Christams. 

- **Workspaces Good.**

Windows 10 has workspaces or virtual desktops, not 2x2 workspaces, but linear workspaces, with all the software you have installed and keep open, workspaces will help unclutter it up ^_^.

- `Win+Tab` opens up the workspaces menu.

- There is a single workspace, by default. Extend it by clicking on the Big Plus button.

- I prefer 4 Workspaces, these added desktops are preserved over sessions, +1.

- If you have a Windows Precision Touchpad, which most new laptops do, you can use the default 4 finger left/right swipe to move between workspaces (similar to how Mac does it). or configure the gestures from the system settings.

- To move between workspaces using the keyboard:
	- `Win+Ctrl+Right` and `Win+Ctrl+Left`

## What's missing?

- KDE Connect

:( its just so good. I am yet to find something similar on windows. I use google drive for transferring small files on windows, on linux, it could be just shared over KDE Connect.

- Can't move windows between workspaces using keyboard shortcuts

You have to drag and drop the specific windows using the mouse/touchpad from the workspaces menu.
