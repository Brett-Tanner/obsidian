# Practical Vim

## Key Notation

- `<>` denotes combinations of keys to be pressed together.
- `{}` indicates a list of valid keystrokes to follow the preceding command
- `-` indicates keys to be pressed together

## Commands

- `@:` repeats the last command
- `:s/target/replacement` replaces the next instance of `target` with `replacement` and is repeatable with `&` (but only on the current line)

## Dot Command

Repeats the last change. But what is a 'change'?

Everything from when you enter insert mode to when you exit it is a single change, unless you interrupt the action by moving around with the arrow keys. Each time you press an arrow key it creates a new undo chunk.

Using motions which drop you into insert mode like `A` or `dw` includes them in the change, making them replay-able along with any changes you make in insert mode.

Generally the pattern you want to aim for is one key to move, then one key to execute.

## LSP

- `]d` and `[d` go to the next and previous diagnostics respectively
- `K` in normal mode brings up the definition of whatever's under the cursor

## Registers

### Expression Register

Accessed with `=`, it evaluates the result of some vimscript code and returns the result. Access it in insert mode with `Ctrl-r=`, then type an expression in the prompt and hit enter to insert it.

## Insert Mode

- `Ctrl-r{register}` pastes at the cursor in insert mode
- `Ctrl-v{code}` allows you to insert an arbitrary character by code (ASCII) or `Ctrl-vu{code}` for unicode
  - `ga` gives you the code for the character under your cursor
- `Ctrl-k{char1}{char2}` inserts the digraph made by the two chars. For example `Ctrl-k>>` would produce »

## Normal Mode

## Operators

Can all be combined with a motion to specify the range they apply to.

- `c` change
- `d` delete
- `y` yank to register
- `gc` comments
- `g~` swap case
- `gu` make lowercase
- `gU` make uppercase
- `<` indent left or `>` right
- `!` filter through an external program

### Motions

- `w` & `b` move forward/backward to the start of the next/previous word, while `e` and `ge` do the same to the end
  - Their capitalized variants move to the start/end of the next/previous WORD, skipping over slashes/hyphens etc.
  - WORD motions also work with change and delete motions
- `gj` & `gk` move down/up one 'display' line, motions like `$` and `^` can also be prefixed with `g` to do the same
- `Ctrl-a` increments the number under the cursor or the next number on the line, while `Ctrl-x` decrements it.
  - Both can prefixed with a number to increment/decrement that many times

### Copying

- `:reg` shows you all registers and their contents
- `yt{char}` yanks from the cursor to `char`

### Deleting

- `daw` uses a text object to delete the entire word the cursor is in and its whitespace; while `caw` does the same and drops you into insert mode
- `db` deletes from cursor to word start; `dw` from cursor to word end
- `S` deletes to the end of a line from the cursor

### Formatting

- `>{num lines}` increases indentation one step from the current position for the number of lines given by the next input
- `gc{motion}` comments the lines covered by `motion`. `gcc` comments the current line and `gc` in visual mode comments the selected lines
- `gu{motion}` or `gU{motion}` downcase or up case the selection covered by `motion`
- `O` adds the newline above the current line rather than below. Useful for avoiding `ko`

### Search

- `*` searches for the word under the cursor and can be incremented/decremented like `/`
- `?` Is a backward search, inverse of `/`, still `n` for next occurrence, `N` for previous occurrence
- `f{char}` searches the line for the next instance of `{char}`, `;` for next occurrence, `,` for previous occurrence, `F{char}` searches backwards
- `c` appended to a `:%s` search will ask you for confirmation before making each change

## Settings

- `hlsearch` toggles search match highlighting
  - In my config it's turned on, and you can hit `esc` in normal mode to dismiss it

## Text Objects

## Visual Mode

Repeating actions taken in visual mode with the dot command repeats them on a selection of the same length as the initial one.

Visual block mode is triggered with `ctrl-v`, and works much the same but in columns rather than lines

### Selection

- `gv` reselects the previous selection
- `o` toggles the 'free end' of the selection
- `vit` selects within tags

### Actions

- `r` replaces the entire selection
- `U` uppercases the selection, `u` downcases it
