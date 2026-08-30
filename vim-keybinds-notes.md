# Vim Keybinds Cheat Sheet

## Modes
| Key | Action |
|---|---|
| `Esc` | Return to Normal mode |
| `i` | Insert before cursor |
| `I` | Insert at start of line |
| `a` | Insert after cursor |
| `A` | Insert at end of line |
| `o` | New line below, insert mode |
| `O` | New line above, insert mode |
| `v` | Visual mode (character) |
| `V` | Visual mode (line) |
| `Ctrl+v` | Visual mode (block) |
| `R` | Replace mode |
| `:` | Command mode |

## Movement (Normal mode)
| Key | Action |
|---|---|
| `h j k l` | Left, down, up, right |
| `w` | Next word start |
| `W` | Next WORD start (ignores punctuation) |
| `b` | Previous word start |
| `B` | Previous WORD start |
| `e` | End of word |
| `E` | End of WORD |
| `0` | Start of line |
| `^` | First non-blank character |
| `$` | End of line |
| `gg` | First line of file |
| `G` | Last line of file |
| `{n}G` or `:{n}` | Go to line n |
| `{` | Previous paragraph/block |
| `}` | Next paragraph/block |
| `%` | Jump to matching bracket |
| `Ctrl+f` | Page down |
| `Ctrl+b` | Page up |
| `Ctrl+d` | Half page down |
| `Ctrl+u` | Half page up |
| `H` | Top of screen |
| `M` | Middle of screen |
| `L` | Bottom of screen |

## Editing
| Key | Action |
|---|---|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dd` | Delete (cut) line |
| `dw` | Delete word |
| `d$` | Delete to end of line |
| `d0` | Delete to start of line |
| `D` | Delete to end of line (same as `d$`) |
| `yy` or `Y` | Yank (copy) line |
| `yw` | Yank word |
| `p` | Paste after cursor/line |
| `P` | Paste before cursor/line |
| `r{char}` | Replace single character |
| `cc` | Change (delete + insert) whole line |
| `cw` | Change word |
| `C` | Change to end of line |
| `u` | Undo |
| `Ctrl+r` | Redo |
| `.` | Repeat last change |
| `J` | Join line below to current line |
| `~` | Toggle case of character |
| `>>` / `<<` | Indent / unindent line |

## Search & Replace
| Key | Action |
|---|---|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Repeat search (same direction) |
| `N` | Repeat search (opposite direction) |
| `*` | Search word under cursor (forward) |
| `#` | Search word under cursor (backward) |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | Replace all with confirmation |
| `:s/old/new/g` | Replace all in current line |

## Visual Mode
| Key | Action |
|---|---|
| `v` then move | Select characters |
| `V` then move | Select lines |
| `Ctrl+v` then move | Select block |
| `d` | Delete selection |
| `y` | Yank selection |
| `c` | Change selection |
| `>` / `<` | Indent / unindent selection |
| `=` | Auto-indent selection |

## Copy/Paste Registers
| Key | Action |
|---|---|
| `"ayy` | Yank line into register `a` |
| `"ap` | Paste from register `a` |
| `"+y` | Yank into system clipboard |
| `"+p` | Paste from system clipboard |

## Windows & Tabs
| Key | Action |
|---|---|
| `:split` or `Ctrl+w s` | Horizontal split |
| `:vsplit` or `Ctrl+w v` | Vertical split |
| `Ctrl+w w` | Switch between windows |
| `Ctrl+w q` | Close current window |
| `Ctrl+w =` | Equalize window sizes |
| `:tabnew` | New tab |
| `gt` | Next tab |
| `gT` | Previous tab |

## File & Session
| Key | Action |
|---|---|
| `:w` | Save |
| `:q` | Quit |
| `:wq` or `:x` | Save and quit |
| `:q!` | Quit without saving |
| `:wa` | Save all open files |
| `:e {file}` | Open a file |
| `ZZ` | Save and quit (normal mode shortcut) |
| `ZQ` | Quit without saving (normal mode shortcut) |

## Marks & Jumps
| Key | Action |
|---|---|
| `m{a-z}` | Set mark |
| `` `{a-z} `` | Jump to mark (exact position) |
| `'{a-z}` | Jump to mark (start of line) |
| `Ctrl+o` | Jump back |
| `Ctrl+i` | Jump forward |

## Useful Combos
| Key | Action |
|---|---|
| `dip` | Delete inner paragraph |
| `di"` | Delete inside quotes |
| `di(` | Delete inside parentheses |
| `ci{` | Change inside braces |
| `yi[` | Yank inside brackets |
| `vap` | Select a paragraph (with spacing) |
| `dat` | Delete a tag (HTML/XML) |

---
*Tip: prefix most commands with a number to repeat, e.g. `3dd` deletes 3 lines, `5j` moves down 5 lines.*
