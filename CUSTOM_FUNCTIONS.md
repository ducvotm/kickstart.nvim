# Custom Vim Functions Reference

Documentation for custom Lua functions that enhance Neovim productivity, based on common workflow needs.

## Table of Contents

- [Diagnostic Functions](#diagnostic-functions)
- [Window Management](#window-management)
- [Terminal Functions](#terminal-functions)
- [LSP Helper Functions](#lsp-helper-functions)
- [Text Manipulation](#text-manipulation)

---

## Diagnostic Functions

### Floating Diagnostic Window

Shows the full diagnostic message in a floating window, making it easier to read long error messages.

```lua
-- Keymap: <leader>e
vim.keymap.set('n', '<leader>e', function()
  vim.diagnostic.open_float(nil, {
    focus = false,
    scope = 'cursor',
    border = 'rounded',
    source = 'always',
    prefix = ' ',
  })
end, { desc = 'Show diagnostic [E]rror messages' })
```

**Configuration Options:**
- `focus = false` - Keeps cursor in current window
- `scope = 'cursor'` - Shows diagnostic at cursor position
- `border = 'rounded'` - Rounded border style
- `source = 'always'` - Always show the source (e.g., "java", "eslint")

### Copy Diagnostic to Clipboard

Copies the diagnostic message at the current cursor position to the clipboard.

```lua
-- Keymap: <leader>ce
local function copy_diagnostic()
  local line = vim.fn.line('.') - 1
  local col = vim.fn.col('.') - 1
  
  local diagnostics = vim.diagnostic.get(0, { lnum = line })
  
  if #diagnostics == 0 then
    print('No diagnostic at cursor')
    return
  end
  
  -- Get the first diagnostic at this line
  local diagnostic = diagnostics[1]
  local message = diagnostic.message
  
  -- Copy to system clipboard
  vim.fn.setreg('+', message)
  
  -- Also copy to default register
  vim.fn.setreg('"', message)
  
  print('Diagnostic copied: ' .. message:sub(1, 50) .. '...')
end

vim.keymap.set('n', '<leader>ce', copy_diagnostic, { desc = '[C]opy diagnostic [E]rror' })
```

**Enhancement: Copy All Diagnostics**

```lua
local function copy_all_diagnostics()
  local diagnostics = vim.diagnostic.get(0)
  
  if #diagnostics == 0 then
    print('No diagnostics in buffer')
    return
  end
  
  local messages = {}
  for _, diag in ipairs(diagnostics) do
    local line_num = diag.lnum + 1
    table.insert(messages, string.format('Line %d: %s', line_num, diag.message))
  end
  
  local result = table.concat(messages, '\n')
  vim.fn.setreg('+', result)
  
  print(string.format('Copied %d diagnostics', #diagnostics))
end

vim.keymap.set('n', '<leader>cE', copy_all_diagnostics, { desc = '[C]opy all [E]rrors' })
```

---

## Window Management

### Floating Help Window

Automatically opens help in a centered floating window instead of a split.

```lua
-- Auto-command approach
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'help',
  callback = function(event)
    -- Only float help if opened in a normal window
    if vim.api.nvim_win_get_config(0).relative ~= '' then
      return
    end
    
    local buf = event.buf
    local width = math.floor(vim.o.columns * 0.8)
    local height = math.floor(vim.o.lines * 0.8)
    local col = math.floor((vim.o.columns - width) / 2)
    local row = math.floor((vim.o.lines - height) / 2)
    
    -- Create floating window
    local win = vim.api.nvim_open_win(buf, true, {
      relative = 'editor',
      width = width,
      height = height,
      col = col,
      row = row,
      style = 'minimal',
      border = 'rounded',
    })
    
    -- Set window options
    vim.api.nvim_win_set_option(win, 'winblend', 0)
    
    -- Close with q
    vim.keymap.set('n', 'q', ':close<CR>', { buffer = buf, silent = true })
  end,
})
```

**Manual Float Help Function:**

```lua
local function float_help(topic)
  topic = topic or vim.fn.expand('<cword>')
  
  -- Try to get help
  local ok, _ = pcall(vim.cmd, 'help ' .. topic)
  if not ok then
    print('No help for: ' .. topic)
    return
  end
  
  -- The help is now open in current window
  local buf = vim.api.nvim_get_current_buf()
  
  -- Create floating window
  local width = math.floor(vim.o.columns * 0.8)
  local height = math.floor(vim.o.lines * 0.8)
  
  vim.api.nvim_open_win(buf, true, {
    relative = 'editor',
    width = width,
    height = height,
    col = math.floor((vim.o.columns - width) / 2),
    row = math.floor((vim.o.lines - height) / 2),
    style = 'minimal',
    border = 'rounded',
  })
end

vim.keymap.set('n', '<leader>h', float_help, { desc = 'Float [H]elp' })
```

---

## Terminal Functions

### Toggle Floating Terminal

Creates a persistent floating terminal that can be toggled on/off.

```lua
local term_buf = nil
local term_win = nil

local function toggle_terminal()
  -- If window exists and is valid, hide it
  if term_win and vim.api.nvim_win_is_valid(term_win) then
    vim.api.nvim_win_hide(term_win)
    term_win = nil
    return
  end
  
  -- Create buffer if it doesn't exist
  if not term_buf or not vim.api.nvim_buf_is_valid(term_buf) then
    term_buf = vim.api.nvim_create_buf(false, true)
    
    -- Set buffer options
    vim.api.nvim_buf_set_option(term_buf, 'buflisted', false)
    
    -- Start terminal in the buffer
    vim.api.nvim_buf_call(term_buf, function()
      vim.fn.termopen(vim.o.shell, {
        on_exit = function()
          -- Cleanup when terminal exits
          term_buf = nil
          term_win = nil
        end,
      })
    end)
  end
  
  -- Calculate window size (80% of editor)
  local width = math.floor(vim.o.columns * 0.8)
  local height = math.floor(vim.o.lines * 0.8)
  local col = math.floor((vim.o.columns - width) / 2)
  local row = math.floor((vim.o.lines - height) / 2)
  
  -- Create floating window
  term_win = vim.api.nvim_open_win(term_buf, true, {
    relative = 'editor',
    width = width,
    height = height,
    col = col,
    row = row,
    style = 'minimal',
    border = 'rounded',
  })
  
  -- Enter insert mode automatically
  vim.cmd('startinsert')
  
  -- Set terminal-specific keymaps
  vim.keymap.set('t', '<Esc><Esc>', '<C-\\><C-n>', { buffer = term_buf })
  vim.keymap.set('t', '<C-w>', '<C-\\><C-n><C-w>', { buffer = term_buf })
end

vim.keymap.set('n', '<leader>tt', toggle_terminal, { desc = '[T]oggle [T]erminal' })
vim.keymap.set('t', '<leader>tt', toggle_terminal, { desc = '[T]oggle [T]erminal' })
```

**Features:**
- Persistent buffer (processes keep running when hidden)
- Auto-enter insert mode when opened
- Easy toggle from normal and terminal mode
- Clean exit handling

### Multiple Terminal Buffers

For managing multiple terminal sessions:

```lua
local terminals = {}

local function toggle_terminal_by_id(id)
  id = id or 1
  
  local term = terminals[id]
  
  -- If window exists, hide it
  if term and term.win and vim.api.nvim_win_is_valid(term.win) then
    vim.api.nvim_win_hide(term.win)
    term.win = nil
    return
  end
  
  -- Create buffer if needed
  if not term or not vim.api.nvim_buf_is_valid(term.buf) then
    local buf = vim.api.nvim_create_buf(false, true)
    vim.api.nvim_buf_call(buf, function()
      vim.fn.termopen(vim.o.shell)
    end)
    
    terminals[id] = { buf = buf, win = nil }
    term = terminals[id]
  end
  
  -- Create window
  local width = math.floor(vim.o.columns * 0.8)
  local height = math.floor(vim.o.lines * 0.8)
  
  term.win = vim.api.nvim_open_win(term.buf, true, {
    relative = 'editor',
    width = width,
    height = height,
    col = math.floor((vim.o.columns - width) / 2),
    row = math.floor((vim.o.lines - height) / 2),
    style = 'minimal',
    border = 'rounded',
    title = ' Terminal ' .. id .. ' ',
    title_pos = 'center',
  })
  
  vim.cmd('startinsert')
end

-- Keymaps for 3 terminals
vim.keymap.set('n', '<leader>t1', function() toggle_terminal_by_id(1) end, { desc = 'Terminal 1' })
vim.keymap.set('n', '<leader>t2', function() toggle_terminal_by_id(2) end, { desc = 'Terminal 2' })
vim.keymap.set('n', '<leader>t3', function() toggle_terminal_by_id(3) end, { desc = 'Terminal 3' })
```

---

## LSP Helper Functions

### List Functions/Methods in File

Filters LSP document symbols to show only functions, methods, and classes.

```lua
local function list_functions()
  -- Determine symbol types based on filetype
  local ft = vim.bo.filetype
  local symbol_types = { 'method', 'function', 'class', 'interface' }
  
  -- Adjust for specific languages
  if ft == 'go' then
    symbol_types = { 'method', 'function', 'struct', 'interface' }
  elseif ft == 'python' or ft == 'sh' then
    symbol_types = { 'function', 'class', 'method' }
  end
  
  require('telescope.builtin').lsp_document_symbols({
    symbols = symbol_types,
    symbol_width = 50,
  })
end

vim.keymap.set('n', 'fm', list_functions, { desc = 'List [F]unctions/[M]ethods' })
```

### Find Classes in Workspace

Search for classes and interfaces across the entire workspace:

```lua
local function find_classes()
  require('telescope.builtin').lsp_dynamic_workspace_symbols({
    symbols = { 'class', 'interface', 'struct' },
  })
end

vim.keymap.set('n', '<leader>fc', find_classes, { desc = '[F]ind [C]lasses' })
```

### Smart Go to Definition

Opens definition in split or floating window based on file size:

```lua
local function smart_goto_definition()
  local current_buf = vim.api.nvim_get_current_buf()
  
  -- Save position before jump
  vim.cmd('normal! m`')
  
  -- Go to definition
  vim.lsp.buf.definition()
  
  -- Wait a bit for the jump
  vim.defer_fn(function()
    local new_buf = vim.api.nvim_get_current_buf()
    
    -- If we jumped to a different buffer
    if current_buf ~= new_buf then
      local lines = vim.api.nvim_buf_line_count(new_buf)
      
      -- If file is small, show in floating window
      if lines < 100 then
        local width = math.floor(vim.o.columns * 0.6)
        local height = math.min(lines + 2, math.floor(vim.o.lines * 0.6))
        
        vim.api.nvim_open_win(new_buf, true, {
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
  end, 100)
end

vim.keymap.set('n', 'gD', smart_goto_definition, { desc = 'Smart go to definition' })
```

---

## Text Manipulation

### Move Lines with Auto-Indent

Moves selected lines up/down while maintaining proper indentation.

```lua
-- Move lines down
vim.keymap.set('v', 'J', ":m '>+1<CR>gv=gv", { desc = 'Move lines down' })

-- Move lines up
vim.keymap.set('v', 'K', ":m '<-2<CR>gv=gv", { desc = 'Move lines up' })
```

**Explanation:**
- `:m '>+1` - Move selection to after last selected line + 1
- `gv` - Reselect the moved text
- `=` - Auto-indent
- `gv` - Reselect again to keep visual mode active

### Duplicate Lines

Quickly duplicate current line or visual selection:

```lua
-- Duplicate line down
vim.keymap.set('n', '<leader>d', 'yyp', { desc = 'Duplicate line down' })

-- Duplicate line up
vim.keymap.set('n', '<leader>D', 'yyP', { desc = 'Duplicate line up' })

-- Duplicate visual selection
vim.keymap.set('v', '<leader>d', 'y`>p', { desc = 'Duplicate selection' })
```

### Join Lines Without Extra Space

Join lines but keep only one space between them:

```lua
-- Normal J adds space, this removes extra spaces
vim.keymap.set('n', '<leader>j', 'J', { desc = 'Join lines' })
vim.keymap.set('n', '<leader>J', 'mzJ`z', { desc = 'Join and return cursor' })
```

---

## Utility Functions

### Center on Search

Keeps search results centered on screen:

```lua
vim.keymap.set('n', 'n', 'nzzzv', { desc = 'Next search (centered)' })
vim.keymap.set('n', 'N', 'Nzzzv', { desc = 'Prev search (centered)' })
```

### Yank to System Clipboard

Easy copy to system clipboard:

```lua
-- Yank to system clipboard
vim.keymap.set({'n', 'v'}, '<leader>y', '"+y', { desc = 'Yank to clipboard' })

-- Yank entire file
vim.keymap.set('n', '<leader>Y', 'gg"+yG', { desc = 'Yank entire file' })

-- Paste from system clipboard
vim.keymap.set({'n', 'v'}, '<leader>p', '"+p', { desc = 'Paste from clipboard' })
```

### Delete Without Yanking

Delete without affecting registers:

```lua
-- Delete to black hole register
vim.keymap.set({'n', 'v'}, '<leader>d', '"_d', { desc = 'Delete (no yank)' })

-- Change without yanking
vim.keymap.set({'n', 'v'}, '<leader>c', '"_c', { desc = 'Change (no yank)' })
```

### Quick Substitute

Replace word under cursor in entire file:

```lua
vim.keymap.set('n', '<leader>s', ':%s/\\<<C-r><C-w>\\>/<C-r><C-w>/gI<Left><Left><Left>', { desc = 'Substitute word' })
```

---

## Integration with Plugins

### Telescope Extensions

Custom pickers for common workflows:

```lua
local pickers = require('telescope.pickers')
local finders = require('telescope.finders')
local conf = require('telescope.config').values

-- Custom picker for project-specific files
local function find_project_files()
  require('telescope.builtin').find_files({
    cwd = require('lspconfig').util.root_pattern('.git', 'pom.xml', 'build.gradle')(vim.fn.expand('%:p')),
  })
end

vim.keymap.set('n', '<leader>fp', find_project_files, { desc = '[F]ind [P]roject files' })
```

### Harpoon Integration

Mark and navigate to frequently used files:

```lua
local mark = require('harpoon.mark')
local ui = require('harpoon.ui')

-- Custom mark with visual feedback
local function harpoon_add()
  mark.add_file()
  print('Added to Harpoon: ' .. vim.fn.expand('%:t'))
end

vim.keymap.set('n', '<Tab>s', harpoon_add, { desc = 'Harpoon mark' })
vim.keymap.set('n', '<Tab>', ui.toggle_quick_menu, { desc = 'Harpoon menu' })

-- Jump to specific marks
for i = 1, 5 do
  vim.keymap.set('n', '<Tab>' .. i, function() ui.nav_file(i) end, { desc = 'Harpoon file ' .. i })
end
```

---

## Tips for Writing Custom Functions

1. **Use `vim.api` for Neovim-specific operations**
2. **Check validity before operating on buffers/windows**
3. **Provide user feedback with `print()` or notifications**
4. **Use `vim.schedule()` for async operations**
5. **Make functions reusable by accepting parameters**
6. **Document your functions with comments**
7. **Test edge cases (empty buffers, no LSP, etc.)**

---

For more productivity tips, see [VIM_TIPS.md](./VIM_TIPS.md).

