# Java Development Setup for Neovim

Complete guide for setting up Neovim for Java and Spring Boot development.

## Table of Contents

- [Prerequisites](#prerequisites)
- [LSP Configuration](#lsp-configuration)
- [Essential Plugins](#essential-plugins)
- [Java-Specific Keymaps](#java-specific-keymaps)
- [Spring Boot Development](#spring-boot-development)
- [Testing Workflow](#testing-workflow)
- [Build Tool Integration](#build-tool-integration)
- [Debugging](#debugging)

---

## Prerequisites

### Required Software

1. **Java JDK** (11 or later)

   ```bash
   # macOS
   brew install openjdk@17
   
   # Verify
   java -version
   ```

2. **jdtls** (Eclipse Java Language Server)

   ```bash
   # Install via Mason (recommended)
   :Mason
   # Search for "jdtls" and install
   ```

3. **Build Tools**
   - Maven: `brew install maven`
   - Gradle: `brew install gradle`

4. **ripgrep** (for fast file searching)

   ```bash
   brew install ripgrep
   ```

### Optional Tools

- **lombok** support (if using Lombok in projects)
- **google-java-format** for code formatting
- **checkstyle** for linting

---

## LSP Configuration

### Basic jdtls Setup

Add to your `init.lua` or create `lua/custom/plugins/java.lua`:

```lua
return {
  {
    'nvim-java/nvim-java',
    dependencies = {
      'nvim-java/lua-async-await',
      'nvim-java/nvim-java-core',
      'nvim-java/nvim-java-test',
      'nvim-java/nvim-java-dap',
      'MunifTanjim/nui.nvim',
      'neovim/nvim-lspconfig',
      'mfussenegger/nvim-dap',
    },
  },
  {
    'neovim/nvim-lspconfig',
    opts = {
      servers = {
        jdtls = {
          -- Your jdtls specific settings
        },
      },
      setup = {
        jdtls = function()
          require('java').setup()
        end,
      },
    },
  },
}
```

### Advanced jdtls Configuration

For more control, create `lua/custom/jdtls.lua`:

```lua
local jdtls = require('jdtls')

-- Find project root
local root_dir = require('jdtls.setup').find_root({'.git', 'mvnw', 'gradlew', 'pom.xml', 'build.gradle'})

-- Data directory for jdtls
local workspace_dir = vim.fn.stdpath('data') .. '/site/java/workspace-root/' .. vim.fn.fnamemodify(root_dir, ':p:h:t')

local config = {
  cmd = {
    'java',
    '-Declipse.application=org.eclipse.jdt.ls.core.id1',
    '-Dosgi.bundles.defaultStartLevel=4',
    '-Declipse.product=org.eclipse.jdt.ls.core.product',
    '-Dlog.protocol=true',
    '-Dlog.level=ALL',
    '-Xmx1g',
    '--add-modules=ALL-SYSTEM',
    '--add-opens', 'java.base/java.util=ALL-UNNAMED',
    '--add-opens', 'java.base/java.lang=ALL-UNNAMED',
    '-jar', vim.fn.glob('/path/to/jdtls/plugins/org.eclipse.equinox.launcher_*.jar'),
    '-configuration', '/path/to/jdtls/config_mac',
    '-data', workspace_dir,
  },

  root_dir = root_dir,

  settings = {
    java = {
      eclipse = {
        downloadSources = true,
      },
      configuration = {
        updateBuildConfiguration = "interactive",
      },
      maven = {
        downloadSources = true,
      },
      implementationsCodeLens = {
        enabled = true,
      },
      referencesCodeLens = {
        enabled = true,
      },
      references = {
        includeDecompiledSources = true,
      },
      format = {
        enabled = true,
        settings = {
          url = vim.fn.stdpath "config" .. "/lang-servers/intellij-java-google-style.xml",
          profile = "GoogleStyle",
        },
      },
    },
    signatureHelp = { enabled = true },
    completion = {
      favoriteStaticMembers = {
        "org.hamcrest.MatcherAssert.assertThat",
        "org.hamcrest.Matchers.*",
        "org.hamcrest.CoreMatchers.*",
        "org.junit.jupiter.api.Assertions.*",
        "java.util.Objects.requireNonNull",
        "java.util.Objects.requireNonNullElse",
        "org.mockito.Mockito.*",
      },
    },
    contentProvider = { preferred = 'fernflower' },
    extendedClientCapabilities = jdtls.extendedClientCapabilities,
    sources = {
      organizeImports = {
        starThreshold = 9999,
        staticStarThreshold = 9999,
      },
    },
    codeGeneration = {
      toString = {
        template = "${object.className}{${member.name()}=${member.value}, ${otherMembers}}",
      },
      useBlocks = true,
    },
  },

  flags = {
    allow_incremental_sync = true,
  },

  init_options = {
    bundles = {},
  },
}

-- Start jdtls
jdtls.start_or_attach(config)
```

---

## Essential Plugins

### Recommended Plugin Setup

```lua
return {
  -- Java development
  { 'nvim-java/nvim-java' },
  
  -- LSP
  { 'neovim/nvim-lspconfig' },
  { 'williamboman/mason.nvim' },
  { 'williamboman/mason-lspconfig.nvim' },
  
  -- Autocompletion
  { 'hrsh7th/nvim-cmp' },
  { 'hrsh7th/cmp-nvim-lsp' },
  { 'hrsh7th/cmp-buffer' },
  { 'hrsh7th/cmp-path' },
  
  -- Snippets (for code generation)
  { 'L3MON4D3/LuaSnip' },
  { 'saadparwaiz1/cmp_luasnip' },
  
  -- Testing
  { 'nvim-neotest/neotest' },
  { 'rcasia/neotest-java' },
  
  -- Debugging
  { 'mfussenegger/nvim-dap' },
  { 'rcarriga/nvim-dap-ui' },
  
  -- File navigation
  { 'nvim-telescope/telescope.nvim' },
  
  -- Git integration
  { 'lewis6991/gitsigns.nvim' },
  { 'tpope/vim-fugitive' },
  
  -- Code navigation enhancement
  { 'ThePrimeagen/harpoon' },
  
  -- Comments
  { 'numToStr/Comment.nvim' },
  
  -- Auto pairs
  { 'windwp/nvim-autopairs' },
}
```

---

## Java-Specific Keymaps

### LSP-Based Operations

```lua
-- After LSP attaches
local on_attach = function(client, bufnr)
  local map = function(keys, func, desc)
    vim.keymap.set('n', keys, func, { buffer = bufnr, desc = 'Java: ' .. desc })
  end

  -- Standard LSP
  map('gd', vim.lsp.buf.definition, 'Go to definition')
  map('gi', vim.lsp.buf.implementation, 'Go to implementation')
  map('gr', vim.lsp.buf.references, 'Find references')
  map('K', vim.lsp.buf.hover, 'Hover documentation')
  map('<leader>rn', vim.lsp.buf.rename, 'Rename symbol')
  map('<leader>ca', vim.lsp.buf.code_action, 'Code actions')
  
  -- Java-specific
  map('<leader>jo', jdtls.organize_imports, 'Organize imports')
  map('<leader>jv', jdtls.extract_variable, 'Extract variable')
  map('<leader>jc', jdtls.extract_constant, 'Extract constant')
  map('<leader>jm', jdtls.extract_method, 'Extract method')
  map('<leader>jt', jdtls.test_class, 'Test class')
  map('<leader>jn', jdtls.test_nearest_method, 'Test nearest method')
end
```

### Testing Shortcuts

```lua
-- Run tests
vim.keymap.set('n', '<leader>tt', ':lua require("neotest").run.run()<CR>', { desc = 'Run nearest test' })
vim.keymap.set('n', '<leader>tf', ':lua require("neotest").run.run(vim.fn.expand("%"))<CR>', { desc = 'Run test file' })
vim.keymap.set('n', '<leader>ts', ':lua require("neotest").summary.toggle()<CR>', { desc = 'Toggle test summary' })
vim.keymap.set('n', '<leader>to', ':lua require("neotest").output.open()<CR>', { desc = 'Show test output' })
```

---

## Spring Boot Development

### Project Structure Recognition

jdtls automatically recognizes Spring Boot projects through:

- `pom.xml` with Spring Boot dependencies
- `build.gradle` with Spring Boot plugin
- `@SpringBootApplication` annotation

### Spring-Specific Code Actions

Available via `<leader>ca`:

- Generate `@Autowired` constructors
- Add missing `@Component`, `@Service`, `@Repository` annotations
- Generate REST controller methods
- Create `@ConfigurationProperties` class

### Application Properties Support

Install YAML/Properties LSP for autocomplete in `application.properties` and `application.yml`:

```lua
require('lspconfig').yamlls.setup({
  settings = {
    yaml = {
      schemas = {
        ["https://www.schemastore.org/api/json/catalog.json"] = "/*",
      },
    },
  },
})
```

---

## Testing Workflow

### JUnit 5 Integration

```lua
require('neotest').setup({
  adapters = {
    require('neotest-java')({
      ignore_wrapper = false,
    })
  }
})
```

### TDD Workflow

1. Write test first (`<leader>jt` to run test class)
2. See it fail
3. Write minimal implementation
4. Run test again (`<leader>tt` on test method)
5. Refactor with confidence

### Test Navigation

```lua
-- Jump between test and implementation
vim.keymap.set('n', '<leader>ja', ':A<CR>', { desc = 'Alternate file (test/impl)' })

-- Requires vim-projectionist configuration
vim.g.projectionist_heuristics = {
  ['pom.xml'] = {
    ['src/main/java/*.java'] = {
      alternate = 'src/test/java/{}Test.java',
      type = 'source'
    },
    ['src/test/java/*Test.java'] = {
      alternate = 'src/main/java/{}.java',
      type = 'test'
    }
  }
}
```

---

## Build Tool Integration

### Maven

```lua
-- Run Maven commands
vim.keymap.set('n', '<leader>mc', ':!mvn clean<CR>', { desc = 'Maven clean' })
vim.keymap.set('n', '<leader>mi', ':!mvn install<CR>', { desc = 'Maven install' })
vim.keymap.set('n', '<leader>mt', ':!mvn test<CR>', { desc = 'Maven test' })
vim.keymap.set('n', '<leader>mp', ':!mvn package<CR>', { desc = 'Maven package' })
vim.keymap.set('n', '<leader>mr', ':!mvn spring-boot:run<CR>', { desc = 'Maven run Spring Boot' })
```

### Gradle

```lua
-- Run Gradle commands
vim.keymap.set('n', '<leader>gc', ':!./gradlew clean<CR>', { desc = 'Gradle clean' })
vim.keymap.set('n', '<leader>gb', ':!./gradlew build<CR>', { desc = 'Gradle build' })
vim.keymap.set('n', '<leader>gt', ':!./gradlew test<CR>', { desc = 'Gradle test' })
vim.keymap.set('n', '<leader>gr', ':!./gradlew bootRun<CR>', { desc = 'Gradle run Spring Boot' })
```

### Floating Terminal Alternative

```lua
-- Run Maven/Gradle in floating terminal
vim.keymap.set('n', '<leader>mr', function()
  require('toggleterm').exec('mvn spring-boot:run', 1, 12)
end, { desc = 'Run Spring Boot' })
```

---

## Debugging

### DAP Setup for Java

```lua
local dap = require('dap')

dap.configurations.java = {
  {
    type = 'java',
    request = 'attach',
    name = "Debug (Attach) - Remote",
    hostName = "127.0.0.1",
    port = 5005,
  },
}

-- Keymaps
vim.keymap.set('n', '<leader>db', dap.toggle_breakpoint, { desc = 'Toggle breakpoint' })
vim.keymap.set('n', '<leader>dc', dap.continue, { desc = 'Continue' })
vim.keymap.set('n', '<leader>di', dap.step_into, { desc = 'Step into' })
vim.keymap.set('n', '<leader>do', dap.step_over, { desc = 'Step over' })
vim.keymap.set('n', '<leader>du', dap.step_out, { desc = 'Step out' })
vim.keymap.set('n', '<leader>dr', dap.repl.open, { desc = 'Open REPL' })
```

### Debug Spring Boot App

1. Start application with debug enabled:

   ```bash
   mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
   ```

2. Set breakpoints in Neovim: `<leader>db`
3. Attach debugger: `<leader>dc`

---

## Tips for Java Development

### 1. Import Organization

Auto-organize imports on save:

```lua
vim.api.nvim_create_autocmd("BufWritePre", {
  pattern = "*.java",
  callback = function()
    vim.lsp.buf.code_action({
      context = { only = { 'source.organizeImports' } },
      apply = true,
    })
  end,
})
```

### 2. Format on Save

```lua
vim.api.nvim_create_autocmd("BufWritePre", {
  pattern = "*.java",
  callback = function()
    vim.lsp.buf.format({ async = false })
  end,
})
```

### 3. Quick Class Navigation

List all classes/interfaces in project:

```lua
vim.keymap.set('n', '<leader>fc', function()
  require('telescope.builtin').lsp_dynamic_workspace_symbols({
    symbols = { 'class', 'interface' }
  })
end, { desc = 'Find classes' })
```

### 4. Lombok Support

If using Lombok, ensure jdtls can find the lombok jar:

```lua
-- Add to jdtls cmd
'-javaagent:/path/to/lombok.jar',
```

### 5. Spring Boot Devtools Live Reload

Enable Spring Boot Devtools in `pom.xml` for automatic restart on file changes.

---

## Troubleshooting

### jdtls Not Starting

1. Check Java version: `java -version`
2. Verify jdtls installation: `:Mason` → find jdtls
3. Check logs: `:LspLog`

### Slow Performance

1. Increase jdtls memory: Change `-Xmx1g` to `-Xmx2g` in config
2. Disable unused features in jdtls settings
3. Use `.gitignore` to exclude `target/`, `build/` directories

### Imports Not Working

1. Run `:JdtOrganizeImports`
2. Check if Maven/Gradle dependencies are resolved
3. Reload project: `:JdtUpdateConfig`

---

## Additional Resources

- [nvim-java GitHub](https://github.com/nvim-java/nvim-java)
- [jdtls Documentation](https://github.com/eclipse/eclipse.jdt.ls)
- [LSP Configuration Examples](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#jdtls)
- [Spring Boot on Neovim Blog Posts](https://www.google.com/search?q=neovim+spring+boot+setup)

---

For general Vim productivity tips, see [VIM_TIPS.md](./VIM_TIPS.md).
