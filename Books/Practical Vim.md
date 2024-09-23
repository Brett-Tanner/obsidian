# Practical Vim

- `<>` denotes combinations of keys to be pressed together.
- `{}` indicates a list of valid keystrokes to follow the preceding command
- `-` indicates keys to be pressed together

## Commands

- `@:` repeats the last command
- `:s/target/replacement` replaces the next instance of `target` with `replacement` and is repeatable with `&` (but only on the current line)

### Global

`:g/{pattern}/{command}` executes `command` on all lines that match `pattern`. Can be inverted with `:g!` (or just `:v`, which executes `command` on all lines that don't match `pattern`.

Remember commands typically take a range as well, so you can do something like `:g/{start}/.,{end}/{cmd}` to execute `cmd` on all lines between `start` and `end`.

For example, to delete html tags on all lines, you'd use `:g/\v<\/?\w+>/d`. If you wanted to delete all lines of a html file without links, you'd use `:v/href/d`.

You could also use it to yank all TODO/FIXME in a file to a register. First empty the register with `q{reg}q`, then `:g/\vTODO|FIXME/yank {capital_reg}`. It's important to use the capital version of a register so each line is appended, rather than overwriting the preceding line. You could also just use `t$` rather than the yank command to dump them all at the end of the file.

## Copying

- `:reg` (or for me, just hitting backtick) shows you all registers and their contents
- `Y` copies to the end of the current line

### Registers

Delete, yank and change all go to the unnamed register (`""`). Yanks also go to the yank register (`"0`) at the same time.

Addressing a register with a lowercase letter overwrites it, while an uppercase letter appends to the register.

Sending anything to the black hole register (`"_`) discards it completely.

- `"{register}{motion}` lets you delete/yank/change to or put from a register

## Deleting

- `daw` uses a text object to delete the entire word the cursor is in and its whitespace; while `caw` does the same and drops you into insert mode
- `db` deletes from cursor to word start; `dw` from cursor to word end
- `S` deletes to the end of a line from the cursor

## Dot Command

Repeats the last change. But what is a 'change'?

Everything from when you enter insert mode to when you exit it is a single change, unless you interrupt the action by moving around with the arrow keys. Each time you press an arrow key it creates a new undo chunk.

Using motions which drop you into insert mode like `A` or `dw` includes them in the change, making them replay-able along with any changes you make in insert mode.

Generally the pattern you want to aim for is one key to move, then one key to execute.

## Formatting

- `>{num lines}` increases indentation one step from the current position for the number of lines given by the next input
- `gc{motion}` comments the lines covered by `motion`. `gcc` comments the current line and `gc` in visual mode comments the selected lines
- `gu{motion}` or `gU{motion}` downcase or up case the selection covered by `motion`
- `O` adds the newline above the current line rather than below. Useful for avoiding `ko`

## Lists

### Change List

Let's you jump through the history of changes for the current buffer

- `:changes` shows the change list
- `g;` goes back, `g,` goes forward
- The `backtick.` mark always stores the last change as well
- `gi` takes you to the last insertion and switches to insert mode

### Jump List

- `:jumps` shows the jump list, which is kept per tab
- `Ctrl-o` jumps back, while `Ctrl-i` jumps forward

## LSP

- `]d` and `[d` go to the next and previous diagnostics respectively
- `K` in normal mode brings up the definition of whatever's under the cursor

## Macros

You can execute macros in parallel on a selection with `:normal @q`. This means if there's a line in the middle which the macro would usually fail on, that failure won't prevent the macro acting on valid lines after it.

If you record a macro using `qq` and realise you need to append something to it (like moving to the next line) you can append to it with `qQ`.

Since macros are just saved into a register, if you want to edit them you can put them into a buffer from that register, then yank the macro back in once edited. When yanking it back into a register, be sure not to yank the whole line so you don't get trailing newline characters in your macro.

Golden rule is 'when recording a macro, ensure every command is repeatable'.

- Start by normalizing the cursor position
- Macros fail if any of their motions do, consider this a feature rather than a bug
  - Lets you do something like `100@q` if your motion is targeted enough
- Macros can also capture find & dot commands, so repeating something like `;.` is valid

- `@@` invokes the most recently invoked macro

## Modes

### Insert Mode

- `Ctrl-r{register}` pastes at the cursor in insert mode
- `Ctrl-v{code}` allows you to insert an arbitrary character by code (ASCII) or `Ctrl-vu{code}` for unicode
  - `ga` gives you the code for the character under your cursor
- `Ctrl-k{char1}{char2}` inserts the digraph made by the two chars. For example `Ctrl-k>>` would produce »

### Visual Mode

Repeating actions taken in visual mode with the dot command repeats them on a selection of the same length as the initial one.

Visual block mode is triggered with `ctrl-v`, and works much the same but in columns rather than lines

## Motions

- `,` reverts an `f{char}` search, while `;` repeats it
  - `%` jumps to the end of the next matching parentheses, then toggles between ends of them
  - `t{char}` is a variant of `f{char}` which stops before `{char}` rather than on it. Remember it as `t` like 'till'.
- `gj` & `gk` move down/up one 'display' line, motions like `$` and `^` can also be prefixed with `g` to do the same
- `Ctrl-a` increments the number under the cursor or the next number on the line, while `Ctrl-x` decrements it.
  - Both can prefixed with a number to increment/decrement that many times
- `m{char}` sets a mark at the cursor position to `{char}`. You can then use `backtick{char}` to jump to that mark, or `'{char}` for the start if the line.
  - lowercase chars as marks are limited to the file they're created in, while uppercase chars are global
  - if you use an uppercase `{char}`, it persists through sessions so can always be used to jump to that file

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

## Patterns

Matched text can be referenced with `\{number}` after the regex. `\0` captures the whole match, which `\n` captures the nth capture group (in `()`). This can then be referenced in a substitute command like `%s/\1/replacement`.

Remember backslashes always need to be escaped to be treated as literals, even with `\V`. Same for `/` if searching forward and `?` for back.

- `<` and `>` are word boundary characters, so `/\vthe>` will not match 'there' or 'their'.
- `\w` matches any word character, while `\W` matches any non-word character
- `\zs` and `\ze` can be used to indicate only part if a pattern should be capturede as the match.
  - `/Practical \zsVim` will find all instances of 'Practical Vim', but only highlight the 'Vim' part
  - Useful when part of the pattern is critical for finding the match, but shouldn't actually be operated on. e.g. text in quotes

### Search

You can browse your search history with the up & down arrow keys.

While searching incrementally, `Ctrl-r Ctrl-w` autofills the full word the current incremental search result is hovering over.

Appending `/e` to a search places your cursor at the end of the match rather than the start. If you wanna add the offset after already searching, just run a blank search with `/e` appended like `//e`, as the last pattern will automatically be reused.

- `*` searches for the word under the cursor and can be incremented/decremented like `/`
- `?` Is a backward search, inverse of `/`
  - You can reverse search direction by starting a blank search in the opposite direction, as the previous pattern is used by default
- `n` for next occurrence, `N` for previous occurrence
  - `gn` is a motion to apply the change to the next match, `gN` for previous
- `f{char}` searches the line for the next instance of `{char}`, `;` for next occurrence, `,` for previous occurrence, `F{char}` searches backwards
- `c` appended to a `:%s` search will ask you for confirmation before making each change

### Substitution

The first arg to `:s`, which is usually `%` for me, is the (vertical) range to act on. Can be a visual selection if one is active when `:s` invoked.

When using the `/c` flag to confirm substitutions, `a` substitutes all remaining matches while `q` quits.

The replacement arg can be `\n` to substitute the nth match, or `\0` to substitute the whole match. You can use this to switch the order of words, for example `:s/\v(for) (example)/\2 \1` will switch 'for example' to 'example for'.

You can use the contents of a register as the replacement value, e.g. `:%s/pattern/\=@"/g` to replace with the contents of the default register.

`g&` repeats the last substitution, but with `%` as the range. `&&` repeats the last substitute command with the curent range (current line by default).

Using `\={vimscript}` in a substitution will execute the given vimscript, e.g. to increment a matched number `:%s/{pattern}/\=submatch(0)+1/g`. `submatch` is needed to refer to capture groups when evaluating vimscript, rather than the usual `\n`.

### Switches

- Adding a `\c` anywhere in your pattern will ignore case. `\C` will force case sensitivity.
- `\V` forces the search to be literal, so `a.k.a` will match literally that string, not a, k and a separated by any 2 chars. `\v` does the opposite.
  - The backslash (escape character) retains its function though.

## Settings

- `hlsearch` toggles search match highlighting
  - In my config it's turned on, and you can hit `esc` in normal mode to dismiss it

## Selection

- `gv` reselects the previous selection
- `o` toggles the 'free end' of the selection
- `vit` selects within tags

## Text Objects

- `v/d/ca` acts around the text object, replacing `i` doew the same within it
- `a{surround_char}` gives a text object around the nearest instances of `{surround_char}`, while `i{surround_char}` gives the same but inside the `{surround_char}` pair.
  - `t` selects around and within tags
  - works with paired brackets, quotes, etc. Not with random letters.
  - works when outside the specified text object, by jumping to the nearest one
- `w` & `b` move forward/backward to the start of the next/previous word, while `e` and `ge` do the same to the end
  - `s` acts on a sentence, `p` for paragraphs
  - Their capitalized variants move to the start/end of the next/previous WORD, skipping over slashes/hyphens etc.
  - WORD motions also work with change and delete motions
