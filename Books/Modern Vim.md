---
title: Modern Vim
description: Hints & tips on using modern Vim features like plugins
---

## Opening Files

Remember that fuzzy finding (like telescope does) just needs the letters to be in the string and in that order, not near each other. So `app/routes/settings/user-management` could be found by `arsu-`. Letters at the start of a word will be given priority over those within a word.

## Plugins

'Packages' are folders which contain one or more plugins. By convention you install them in `~/.config/nvim/pack`. Plugins in the `start` subdirectory of the package should run when Neovim starts. Just putting a plugin inside a package installs it, so if I need to get rid of Lazy at some point I can just clone the plugin repos into `pack`.

Plugins can be added at runtime, until restarted, with `:packadd plugin-name`, which also adds them to Neovim's `runtimepath`.

You then generate the `help` docs for the plugin with `:helptags ~/.config/nvim/pack/*/doc`. Needs to be re-run after updates as it's not automatically updated when files change.

Incidentally, seems you can follow hyperlinks in the docs with `<C-]>`.

## Terminal

`:!{cmd}` is handy for quickly running one off commands from Neovim's command line.

You can `Ctrl-z` to suspend Neovim and go back to the terminal, then `fg` to resume. But running the terminal in Neovim gives you access to stuff like search, yank and all your other keybinds.

When typing in the built-in terminal emulator, you enter `Terminal` mode which can have its own keybinds.

The terminal starts in `Normal` mode, and any of `i`, `I`, `a` or `A` puts you in `Terminal` mode. To get back to `Normal` the default keybind is `Ctrl-\+Ctrln`.

`:read !{cmd}` can be used to run a command and read the output into the current buffer, while `:term {cmd}` reads the output into a new 'terminal buffer' which is updated every second or so if the command keeps running.

Opening a terminal in a buffer with `:term` does so in an 'alternate file', which allows you to switch between the terminal and the underlying buffer with `Ctrl-^`. `gf` to open the filepath under the cursor works the same way.

Terminal buffers suspend with Neovim, and will be restored from sessions.

## Autocommands

Defined with the syntax `autocmd {event} {pattern} {command}` where `event` is like `BufReadPost` or `BufWritePre`, `pattern` is a regex applied to the filename and `command` is, funnily enough, a command to run when the `event` & `pattern` are matched.

You can define multiple comma-separated events, and the current file is available in `command` as `<afile>` (so to get the filename you can use `expand('<afile>')`).

For the `FileType` event `pattern` is matched against the filetype, not the filename.

`autocmd!` removes all autocommands for the current group.
