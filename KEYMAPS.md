# Vim Keymaps Quick Reference

Quick lookup table for all essential keybindings, organized by category.

## Diagnostics & Errors

| Keymap | Action | Description |
|--------|--------|-------------|
| `<leader>e` | Show diagnostic | Open floating window with full error message |
| `<leader>ce` | Copy diagnostic | Copy error/warning message to clipboard |
| `]d` or `]e` | Next diagnostic | Jump to next error/warning |
| `[d` or `[e` | Previous diagnostic | Jump to previous error/warning |

## File Navigation

| Keymap | Action | Description |
|--------|--------|-------------|
| `<leader>ff` | Find files | Search files in project (respects .gitignore) |
| `<leader>pf` | Find hidden files | Search hidden files (`.env`, `.gitignore`, etc.) |
| `<leader>fg` | Grep in files | Search text content across project |
| `<leader>fb` | Recent buffers | Show recently opened files |
| `<leader>fh` | Search help | Fuzzy search Vim help topics |
| `Ctrl-^` or `Ctrl-6` | Toggle buffer | Switch between current and last buffer |

## Harpoon (File Marking)

| Keymap | Action | Description |
|--------|--------|-------------|
| `<Tab>s` | Mark file | Add current file to Harpoon marks |
| `<Tab>` | Show marks | Toggle Harpoon quick menu |
| `<Tab>]` | Next mark | Navigate to next marked file |
| `<Tab>[` | Previous mark | Navigate to previous marked file |
| `m` | Mark line | Mark specific line in file |

## LSP Operations

| Keymap | Action | Description |
|--------|--------|-------------|
| `gd` | Go to definition | Jump to where symbol is defined |
| `gi` | Go to implementation | Jump to interface implementation |
| `gD` | Go to declaration | Jump to symbol declaration |
| `gr` | Find references | List all uses of symbol |
| `K` | Hover documentation | Show docs/signature in floating window |
| `<leader>rn` | Rename symbol | Rename across entire project |
| `<leader>ca` | Code actions | Show available quick fixes/refactorings |
| `fm` | List functions | Show all functions/methods in file |

## Text Editing & Manipulation

### Line Operations

| Keymap | Mode | Action | Description |
|--------|------|--------|-------------|
| `gcc` | Normal | Toggle line comment | Comment/uncomment current line |
| `gc{motion}` | Normal | Comment motion | Comment text object (e.g., `gcap` for paragraph) |
| `gc` | Visual | Comment selection | Comment/uncomment selected lines |
| `gb` | Visual | Block comment | Toggle block comment style |
| `gc2j` | Normal | Comment down | Comment current + 2 lines below |
| `gc3k` | Normal | Comment up | Comment current + 3 lines above |

### Visual Mode

| Keymap | Mode | Action | Description |
|--------|------|--------|-------------|
| `J` | Visual | Move down | Move selected lines down with auto-indent |
| `K` | Visual | Move up | Move selected lines up with auto-indent |

### Surround Operations

| Keymap | Mode | Action | Description |
|--------|------|--------|-------------|
| `S{char}` | Visual | Add surround | Surround selection with character |
| `cs{old}{new}` | Normal | Change surround | Change surrounding characters |
| `ds{char}` | Normal | Delete surround | Remove surrounding characters |
| `ysiw{char}` | Normal | Surround word | Surround inner word |
| `yss{char}` | Normal | Surround line | Surround entire line |

**Examples:**
- `cs"'` - Change double quotes to single quotes
- `cs"(` - Change quotes to parentheses  
- `ds"` - Delete surrounding quotes
- `ysiw"` - Surround word with quotes

## Terminal

| Keymap | Action | Description |
|--------|--------|-------------|
| `<leader>tt` | Toggle terminal | Open/close floating terminal window |
| `Ctrl-\` `Ctrl-n` | Normal mode | Exit terminal mode to normal mode |

## Window Navigation

| Keymap | Action | Description |
|--------|--------|-------------|
| `Ctrl-h` | Move left | Move to left window |
| `Ctrl-j` | Move down | Move to lower window |
| `Ctrl-k` | Move up | Move to upper window |
| `Ctrl-l` | Move right | Move to right window |

## Advanced Operations

### Project-wide Find/Replace

1. `<leader>fg` - Search for text
2. `Ctrl-q` - Send results to quickfix list
3. `:cdo s/old/new/gc | update` - Replace with confirmation

**Flags:**
- `g` - Replace all occurrences in line
- `c` - Confirm each replacement
- `i` - Case insensitive

### Common Motions

| Motion | Description |
|--------|-------------|
| `w` | Next word start |
| `b` | Previous word start |
| `e` | End of word |
| `0` | Start of line |
| `^` | First non-blank character |
| `$` | End of line |
| `gg` | Top of file |
| `G` | Bottom of file |
| `{` | Previous paragraph/block |
| `}` | Next paragraph/block |
| `%` | Matching bracket/paren |

### Text Objects

Use with operators like `d` (delete), `c` (change), `y` (yank), `v` (visual):

| Text Object | Description |
|-------------|-------------|
| `iw` | Inner word |
| `aw` | A word (with space) |
| `i"` | Inside quotes |
| `a"` | Around quotes (includes quotes) |
| `i(` or `ib` | Inside parentheses |
| `a(` or `ab` | Around parentheses |
| `i{` or `iB` | Inside braces |
| `a{` or `aB` | Around braces |
| `it` | Inside tag (HTML/XML) |
| `at` | Around tag |
| `ip` | Inner paragraph |
| `ap` | Around paragraph |

**Examples:**
- `diw` - Delete inner word
- `ci"` - Change inside quotes
- `ya{` - Yank around braces
- `vip` - Visually select paragraph

## Leader Key

Default leader key is typically `Space` or `,`. Check your config:
```lua
vim.g.mapleader = ' '  -- Space as leader
vim.g.maplocalleader = ' '
```

## Tips

1. **Repeat last command**: `.` (dot)
2. **Repeat last substitution**: `&` or `:s`
3. **Undo**: `u`
4. **Redo**: `Ctrl-r`
5. **Visual block mode**: `Ctrl-v`
6. **Search forward**: `/pattern`
7. **Search backward**: `?pattern`
8. **Next search result**: `n`
9. **Previous search result**: `N`
10. **Clear search highlight**: `:noh` or `<Esc><Esc>`

## Customizing Keymaps

Add custom keymaps in your `init.lua` or separate keymap file:

```lua
-- Basic format
vim.keymap.set('n', '<leader>w', ':w<CR>', { desc = 'Save file' })

-- With function
vim.keymap.set('n', '<leader>x', function()
  print('Custom action')
end, { desc = 'Custom action' })

-- Multiple modes
vim.keymap.set({'n', 'v'}, '<leader>y', '"+y', { desc = 'Yank to system clipboard' })
```

---

For detailed explanations and use cases, see [VIM_TIPS.md](./VIM_TIPS.md).

