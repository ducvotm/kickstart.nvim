# Vim Productivity Tips for Java Developers

A comprehensive guide to essential Vim/Neovim productivity techniques focused on Java development workflows.

## Table of Contents

- [Diagnostic/Error Navigation](#diagnosticerror-navigation)
- [File Navigation](#file-navigation)
- [LSP Operations](#lsp-operations)
- [Code Manipulation](#code-manipulation)
- [Text Editing](#text-editing)
- [Terminal Integration](#terminal-integration)
- [Help System](#help-system)

---

## Diagnostic/Error Navigation

When working with long files containing multiple errors or warnings, these commands help you navigate and handle diagnostics efficiently.

### Show Full Diagnostic Message

**Keymap:** `<leader>e`

Shows the full diagnostic message in a floating window. Useful when error messages are truncated in the status line.

```lua
-- Custom keymap configuration
vim.keymap.set('n', '<leader>e', vim.diagnostic.open_float, { desc = 'Show diagnostic [E]rror messages' })
```

### Copy Diagnostic Message

**Keymap:** `<leader>ce`

Copies the current diagnostic message (error or warning) to the clipboard. Helpful for searching errors online or sharing with teammates.

```lua
-- Custom function to copy diagnostic to clipboard
local function copy_diagnostic()
  local diagnostics = vim.diagnostic.get(0, { lnum = vim.fn.line('.') - 1 })
  if #diagnostics > 0 then
    local message = diagnostics[1].message
    vim.fn.setreg('+', message)
    print('Diagnostic copied to clipboard')
  end
end

vim.keymap.set('n', '<leader>ce', copy_diagnostic, { desc = '[C]opy diagnostic [E]rror' })
```

### Navigate Between Diagnostics

**Next Error/Warning:** `]e` or `]d`  
**Previous Error/Warning:** `[e` or `[d`

Quickly jump between diagnostics in a file without scrolling.

```lua
vim.keymap.set('n', ']d', vim.diagnostic.goto_next, { desc = 'Go to next [D]iagnostic message' })
vim.keymap.set('n', '[d', vim.diagnostic.goto_prev, { desc = 'Go to previous [D]iagnostic message' })
```

**Pro Tip:** In large files (1000+ lines) with multiple errors, these keybindings are essential for efficient error handling.

---

## File Navigation

Efficient navigation between files is crucial for productivity. These techniques leverage Telescope for fuzzy finding.

### Find Files in Directory

**Keymap:** `<leader>ff`

Searches for files in your project directory (respects `.gitignore`).

```lua
require('telescope.builtin').find_files()
```

### Find Hidden Files

**Keymap:** `<leader>pf`

Searches for hidden files (files starting with `.`) including `.env`, `.gitignore`, etc.

```lua
require('telescope.builtin').find_files({ hidden = true })
```

### Search File Contents (Grep)

**Keymap:** `<leader>fg`

Searches for text within files across your project. Shows preview of matches.

**Example:** Search for "saveEvent" to find all files containing that string.

```lua
require('telescope.builtin').live_grep()
```

**Note:** Requires `ripgrep` to be installed.

### Recent Buffers

**Keymap:** `<leader>fb`

Shows recently opened files with numbers for quick access.

```lua
require('telescope.builtin').buffers()
```

**Alternative:** Use `Ctrl-^` (or `Ctrl-6`) to toggle between current and last buffer.

### File Marking with Harpoon

Harpoon allows you to mark important files for instant access, especially useful when working across multiple files.

**Mark current file:** `<Tab>s` (or your custom mapping)  
**Show marked files:** `<Tab>`  
**Navigate marks:** `<Tab>[` (previous) and `<Tab>]` (next)

**Use Case:** Mark files you're actively debugging while keeping the ability to navigate to specific points of interest.

```lua
-- Example Harpoon configuration
local mark = require('harpoon.mark')
local ui = require('harpoon.ui')

vim.keymap.set('n', '<Tab>s', mark.add_file)
vim.keymap.set('n', '<Tab>', ui.toggle_quick_menu)
vim.keymap.set('n', '<Tab>]', ui.nav_next)
vim.keymap.set('n', '<Tab>[', ui.nav_prev)
```

**Pro Tip:** Each tmux session maintains separate Harpoon marks, preventing conflicts between projects.

### Mark Lines Within File

**Mark line:** `m` (in normal mode on the line)  
**View marks:** Will show in Harpoon menu with line numbers  
**Delete mark:** Delete from Harpoon menu

Useful for marking important sections while debugging or working with large files.

---

## LSP Operations

Language Server Protocol (LSP) features for code intelligence and navigation.

### Go to Definition

**Keymap:** `gd`

Jumps to where a variable, function, or class is defined.

**Use Case:** You're reading code and see `userRepository.findById()` - press `gd` to see the method definition.

```lua
vim.keymap.set('n', 'gd', vim.lsp.buf.definition, { desc = '[G]o to [D]efinition' })
```

### Go to Implementation

**Keymap:** `gi`

Jumps to the implementation of an interface method.

**Use Case:** You have a `UserService` interface and want to see which classes implement the `createUser()` method.

```lua
vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, { desc = '[G]o to [I]mplementation' })
```

**Note:** If multiple implementations exist, shows a list to choose from.

### Go to Declaration

**Keymap:** `gD`

Jumps to the declaration (different from definition in some languages).

```lua
vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, { desc = '[G]o to [D]eclaration' })
```

### Find References

**Keymap:** `gr`

Lists all locations where a symbol is used or referenced in the project.

**Use Case:** Find all places where a method is called or a variable is referenced.

```lua
vim.keymap.set('n', 'gr', vim.lsp.buf.references, { desc = '[G]o to [R]eferences' })
```

### Hover Documentation

**Keymap:** `K` (Shift + k)

Shows documentation, type information, and function signatures in a floating window.

**Use Case:** Quickly check what parameters a method accepts without leaving your current position.

```lua
vim.keymap.set('n', 'K', vim.lsp.buf.hover, { desc = 'Hover Documentation' })
```

### Rename Symbol

**Keymap:** `<leader>rn`

Renames a variable, method, or class across the entire project.

**Use Case:** Rename `account` to `userAccount` everywhere in your codebase.

```lua
vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, { desc = '[R]e[n]ame' })
```

**Important:** This only works when there are no errors in related files. For renaming despite errors, see [Project-wide Find/Replace](#project-wide-findreplace).

### Code Actions

**Keymap:** `<leader>ca`

Shows available code actions (quick fixes, imports, refactorings).

**Examples:**
- Auto-import missing classes
- Organize imports
- Generate getters/setters
- Implement interface methods

```lua
vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, { desc = '[C]ode [A]ction' })
```

### List All Functions/Methods

**Keymap:** `fm` (custom mapping)

Lists all functions, methods, classes, and interfaces in the current file using LSP document symbols.

**Use Case:** Exploring a library or package to see available methods at a glance.

```lua
-- Custom function to filter document symbols
local function list_functions()
  require('telescope.builtin').lsp_document_symbols({
    symbols = { 'method', 'function', 'class', 'interface' }
  })
end

vim.keymap.set('n', 'fm', list_functions, { desc = 'List [F]unctions/[M]ethods' })
```

**Note:** Symbol types vary by language (use `'method'` for Go, `'function'` for Python/Shell).

---

## Code Manipulation

### Project-wide Find/Replace

When you need to replace text across multiple files (even when LSP rename doesn't work due to errors).

**Steps:**

1. **Search for the text:**
   - Use `<leader>fg` to grep for the text (e.g., "Balance")

2. **Send results to quickfix list:**
   - Press `Ctrl-q` in Telescope to populate quickfix list

3. **Edit all matches:**
   - `:cdo s/Balance/Amount/gc | update`
   
   Breaking it down:
   - `cdo` - execute command on each quickfix entry
   - `s/Balance/Amount/` - substitute old with new
   - `g` - replace all occurrences in each line
   - `c` - confirm each replacement
   - `| update` - save the file

4. **Confirm changes:**
   - Press `y` (yes) or `n` (no) for each match
   - Press `a` (all) to replace all remaining in the file
   - Press `q` (quit) to stop

**Pro Tip:** Remove `c` flag for automatic replacement without confirmation: `s/Balance/Amount/g`

### Moving Lines with Auto-Indent

**Visual Mode:**
- Select lines (Visual or Visual Line mode)
- `Shift-J` - Move lines down with proper indentation
- `Shift-K` - Move lines up with proper indentation

```lua
-- Custom keymap for moving lines
vim.keymap.set('v', 'J', ":m '>+1<CR>gv=gv", { desc = 'Move line down' })
vim.keymap.set('v', 'K', ":m '<-2<CR>gv=gv", { desc = 'Move line up' })
```

**Use Case:** Moving code blocks in and out of if statements while maintaining correct indentation.

---

## Text Editing

### Surround Text

Add, change, or delete surrounding characters (quotes, brackets, tags).

**Add surround (in visual mode):**
- Select text → Press `S` → Type character (e.g., `"`, `(`, `[`)

**Change surround (in normal mode):**
- `cs"'` - Change double quotes to single quotes
- `cs"(` - Change quotes to parentheses
- `cs)]` - Change ) to ] (removes inner space)

**Delete surround:**
- `ds"` - Delete surrounding quotes
- `ds(` - Delete surrounding parentheses

**Surround word:**
- `ysiw"` - Surround inner word with quotes
- `ysiw(` - Surround inner word with parentheses

**Surround line:**
- `yss"` - Surround entire line with quotes

**Use Case:** Quickly convert CSV values to quoted strings, add parentheses to method calls, or wrap text in HTML tags.

### Comment Toggle

**Line comment:**
- `gcc` - Toggle comment on current line
- `gc2j` - Comment current line and 2 lines below
- `gc3k` - Comment current line and 3 lines above

**Block comment:**
- Select text (Visual mode) → `gb` - Toggle block comment

**Motion comment:**
- `gcap` - Comment a paragraph
- `gc}` - Comment to end of block

```lua
-- Using Comment.nvim plugin (default keybindings work out of the box)
require('Comment').setup()
```

---

## Terminal Integration

### Floating Terminal Toggle

Opens a floating terminal window for quick commands without splitting panes.

**Use Case:** Quick git commits, docker commands, or running scripts without opening a new tmux pane.

**Features:**
- Toggle on/off with keymap
- All running processes persist when hidden
- Each tmux session has independent terminal states

```lua
-- Custom floating terminal toggle
local term_buf = nil
local term_win = nil

local function toggle_terminal()
  if term_win and vim.api.nvim_win_is_valid(term_win) then
    vim.api.nvim_win_hide(term_win)
    term_win = nil
  else
    if not term_buf or not vim.api.nvim_buf_is_valid(term_buf) then
      term_buf = vim.api.nvim_create_buf(false, true)
      vim.api.nvim_buf_call(term_buf, function()
        vim.fn.termopen(vim.o.shell)
      end)
    end
    
    local width = math.floor(vim.o.columns * 0.8)
    local height = math.floor(vim.o.lines * 0.8)
    
    term_win = vim.api.nvim_open_win(term_buf, true, {
      relative = 'editor',
      width = width,
      height = height,
      col = math.floor((vim.o.columns - width) / 2),
      row = math.floor((vim.o.lines - height) / 2),
      style = 'minimal',
      border = 'rounded',
    })
  end
end

vim.keymap.set('n', '<leader>tt', toggle_terminal, { desc = '[T]oggle [T]erminal' })
```

---

## Help System

### Floating Help Window

Displays Vim help in a clean floating window instead of a split.

**Keymap:** Use `:h <topic>` as usual

**Benefit:** Better separation between code and documentation, easier to read without cluttering workspace.

```lua
-- Custom floating help window
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'help',
  callback = function()
    -- Make help open in floating window
    vim.bo.bufhidden = 'wipe'
    
    local width = math.floor(vim.o.columns * 0.8)
    local height = math.floor(vim.o.lines * 0.8)
    
    vim.api.nvim_open_win(0, true, {
      relative = 'editor',
      width = width,
      height = height,
      col = math.floor((vim.o.columns - width) / 2),
      row = math.floor((vim.o.lines - height) / 2),
      style = 'minimal',
      border = 'rounded',
    })
  end,
})
```

### Search Help with Telescope

**Keymap:** `<leader>fh`

Fuzzy search through Vim help topics.

```lua
require('telescope.builtin').help_tags()
```

---

## Summary

These productivity tips focus on:

1. **Fast navigation** - Between files, errors, and code definitions
2. **Efficient editing** - Bulk operations, smart text manipulation
3. **Code intelligence** - LSP-powered features for understanding code
4. **Minimal friction** - Floating windows, quick access terminals

Remember: The goal is not just speed, but maintaining flow state while coding. These techniques reduce context switching and keep you focused on solving problems rather than fighting your editor.

For keybinding quick reference, see [KEYMAPS.md](./KEYMAPS.md).

