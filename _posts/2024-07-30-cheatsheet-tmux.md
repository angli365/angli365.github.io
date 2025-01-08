---
layout: post
title: Cheatsheet Tmux
date: 2024-07-30 14:19 +0200
author: me
categories: [cheatsheet, tmux]
tags: [cheatsheet, tmux]
---

# 🖥️ Tmux Cheatsheet

## Table of Contents
- [Sessions](#sessions)
- [Windows](#windows)
- [Panes](#panes)
- [Copy Mode](#copy-mode)
- [Misc](#misc)
- [Help](#help)

## Sessions

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| Start a new session | `tmux`, `tmux new`, `tmux new-session` | `new` | |
| Start a new session named *mysession* | `tmux new -s mysession` | `new -s mysession` | |
| Rename session | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>$</kbd> |
| Detach from session | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>d</kbd> |
| Show all sessions | `tmux ls`, `tmux list-sessions` | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>s</kbd> |
| Attach to last session | `tmux a`, `tmux at`, `tmux attach`, `tmux attach-session` | | |
| Attach to session *mysession* | `tmux a -t mysession`, `tmux at -t mysession`, `tmux attach -t mysession`, `tmux attach-session -t mysession` | | |
| Kill/delete session *mysession* | `tmux kill-ses -t mysession`, `tmux kill-session -t mysession` | | |
| Kill all sessions but the current | `tmux kill-session -a` | | |
| Kill all sessions but *mysession* | `tmux kill-session -a -t mysession` | | |
| Move to previous session | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>(</kbd> |
| Move to next session | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>)</kbd> |

## Windows

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| Create window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>c</kbd> |
| Rename current window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>,</kbd> |
| Close current window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>&</kbd> |
| List windows | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>w</kbd> |
| Previous window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>p</kbd> |
| Next window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>n</kbd> |
| Switch/select window by number | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>0</kbd>...<kbd>9</kbd> |
| Toggle last active window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>l</kbd> |
| Reorder window, swap window 2 and 1 | | `swap-window -s 2 -t 1` | |
| Move current window to the left | | `swap-window -t -1` | |

## Panes

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| Split pane horizontally | | `split-window -h` | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>%</kbd> |
| Split pane vertically | | `split-window -v` | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>"</kbd> |
| Switch to pane in direction | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>↑</kbd>/<kbd>↓</kbd>/<kbd>←</kbd>/<kbd>→</kbd> |
| Toggle between pane layouts | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>Space</kbd> |
| Show pane numbers | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>q</kbd> |
| Switch/select pane by number | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>q</kbd> <kbd>0</kbd>...<kbd>9</kbd> |
| Toggle pane zoom | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>z</kbd> |
| Convert pane into a window | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>!</kbd> |
| Resize pane height | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>↑</kbd>/<kbd>↓</kbd> (hold) |
| Resize pane width | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>←</kbd>/<kbd>→</kbd> (hold) |
| Close current pane | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>x</kbd> |

## Copy Mode

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| Enter copy mode | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>[</kbd> |
| Enter copy mode and scroll up | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>PgUp</kbd> |
| Quit mode | | | <kbd>q</kbd> |
| Go to top line | | | <kbd>g</kbd> |
| Go to bottom line | | | <kbd>G</kbd> |
| Move cursor | | | <kbd>h</kbd>/<kbd>j</kbd>/<kbd>k</kbd>/<kbd>l</kbd> |
| Start selection | | | <kbd>Space</kbd> |
| Clear selection | | | <kbd>Esc</kbd> |
| Copy selection | | | <kbd>Enter</kbd> |
| Paste contents of buffer_0 | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>]</kbd> |

## Misc

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| Enter command mode | | | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>:</kbd> |
| Enable mouse mode | | `set mouse on` | |

## Help

| Description | Shell Command | Tmux Command | Shortcut |
|-------------|---------------|--------------|----------|
| List key bindings | `tmux list-keys` | `list-keys` | <kbd>Ctrl</kbd> + <kbd>b</kbd> <kbd>?</kbd> |
| Show every session, window, pane, etc. | `tmux info` | | |

----
<div align="center" style="background: linear-gradient(135deg, #ffffff, #f5f5f5); padding: 40px; border-radius: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.12); margin: 40px 0; backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);">
    <p style="font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Text', sans-serif; font-size: 1.1em; color: #1d1d1f; line-height: 1.5; margin-bottom: 25px; font-weight: 400;">
        <i>Is this blog helpful? Help fuel more in-depth technical content by treating me to a coffee! Your support keeps the knowledge flowing ☕✨</i>
    </p>
    <a href="https://www.buymeacoffee.com/angli"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=angli&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a coffee button"></a>
</div>
