# stunning-octo-system
Skip to content

You have no unread notifications
Code
Issues
2
Pull requests
Discussions
Actions
Projects
Security
Insights
Notification settings
Fork your own copy of ufo5260987423/scheme-langserver
Star this repository
Scheme language server

License
 View license
 64 stars
 5 forks
 5 watching
 Activity
Public repository
ufo5260987423/scheme-langserver
 Branches
 Tags
Latest commit
@ufo5260987423
ufo5260987423
…
last week
Git stats
Files
README.md


Scheme-langserver
Implementing support like autocomplete, goto definition, or documentation on hover is a significant effort for programming. However, comparing to other language like java, python, javascript and c, language server protocol implementation for lisp language are just made in a vacuum. Geiser, racket langserver and swish-lint etc., their works are all based on repl(Read-Eval-Print Loop) or keyword tokenizer instead of programming. For example, if a programmer was coding on an unaccomplished project, in which the codes are not fully runnable, Geiser or any others would only complete top-level binding identifiers listed by environment-symbols procedure (for Chez). Which means for local bindings and unaccomplished codes, though making effort for programming is supposed of the importance mostly, Geiser and its counterparts help nothing. Familiar cases occur with goto definition and many other functionalities.

A primary cause is, for scheme and other lisp dialects, their abundant data sets and flexible control structures raise program analysis a big challenge. In fact, scheme even don't have commonly acknowledged project management frameworks and corresponding extension system. As for extensions SS, and SCM, most programmers suppose their codes are writing for a running environment and don't provide any library information. Although with Akku and Snow, SLS and SLD files can base a project on a stable library framework, load, load-program and many other procedures which maintain dynamic library linkages make static code analysis telling less.

This package is a language server protocol implementation helping scheme programming. It provides completion, definition and will provide many other functionalities for SLS and SLD files. These functionalities are established on static code analysis with r6rs standard and some obvious rules for unaccomplished codes. This package itself and related libraries are published or going to be published with Akku, which is a package manager for Scheme.

This package also has been tested with Chez Scheme versions 9.4 and 9.5.

Your donation will make this world better. Also, you can issue your advice and I might implement if it was available. paypal

NOTE: This implementation is mainly appliable for .sls and .sld files, because .ss and .sps suppose that they're executing in a running virtual machine. A detailed discussion is now running.

Recent Status
I'm working for user-friendly diagnostic issues by making type inference rules for higher-order procedures like car. Otherwise, I have a plan to do some profiling to accelerate index speed.

NOTE: This project is still in early development, so you may run into rough edges with any of the features. The following list shows the status of various features.

Log
1.0.11: Gradual Typing system, all basic rules have been passed (you can verify it with test/analysis/type/*.sps and test/analysis/type/rules/*.sps). Detailed documentation has been published.

1.0.10: Fix bugs in 1.0.9.

1.0.9: Abandoned: add parallel and synchronize mechanism, which can harshly speed up indexing.

1.0.8: Build index as document synchronizing instead of workspace initializing.

1.0.7: Catch syntax-* identifier bindings.

Setup
Building
Pre-require
Chez Scheme;
NOTE If you wanted to enable scheme-langserver's muti-thread feature, it would require Chez Scheme to have been compiled with the --threads option. I've test multi-thread feature for Linux, and according to this page, it will also excutable for Windows.

Akku；
chez-exe；
NOTE chez-exe requires boot files and kernel files of Chez Scheme. So, the compile command maybe like follows:scheme --script gen-config.ss --bootpath /path-to-ChezScheme/{machine-type}/boot/{machine-type}

For Linux
git clone https://github.com/ufo5260987423/scheme-langserver
cd scheme-langserver
akku install
bash .akku/env
compile-chez-program run.ss
./run path-to-logfile
TODO: for Windows
The run file is also executable for windows WSL environment, and I'm waiting for Chez Scheme 10. As their announcement, they will merge the racket implemented Chez Scheme into main branch. And in this page it seems that the racket implementation has solved the multi-thread problem.

Further building is going to be done after Chez scheme 10 released. Because, there're many issues about building scheme-langsever on Windows: first, for Chez scheme, its library loading mechanism is *nix-liked, which means for srfi, whose library path is like srfi/:1/lists.sls, can't be handled on Windows7; Second, many instruments are based on *nix, like Akku, it depends on Guile, which only has linux-based releases. Third, Chez now doesn't have portable bytecode (pb) mode, which is mainly useful for bootstrapping a build on any platform.

Installation for LunarVim(1.3)
I have pull request to mason.nvim and mason-lspconfig.nvim. In that case, you can get this implementation automatically with LunarVim.

But now, above configuration haven't been tested. So, manual installation is still needed: for installed plugin nvim-lspconfig, after manually building from above step Building, an executable file run would be available at {path-to-run}. Then, create file ~/.local/share/lunarvim/site/pack/lazy/opt/nvim-lspconfig/lua/lspconfig/server_configurations/scheme_langserver.lua as follows:

local util = require 'lspconfig.util'
local bin_name = '{path-to-run}'
local cmd = { bin_name }

return {
  default_config = {
    cmd = cmd,
    filetypes = { 'scheme' },
    root_dir = util.find_git_ancestor,
    single_file_support = true,
  },
  docs = {
    description = [[
https://github.com/ufo5260987423/scheme-langserver
`scheme-langserver`, a language server protocol implementation for scheme
]]   ,
  },
}
Then configure your ~/.config/lvim/config.lua and add following codes like:

vim.cmd [[ au BufRead,BufNewFile *.sld set filetype=scheme ]]
vim.cmd [[ au BufRead,BufNewFile *.sls set filetype=scheme ]]

vim.api.nvim_create_autocmd('LspAttach', {
  group = vim.api.nvim_create_augroup('UserLspConfig', {}),
  callback = function(ev)
    -- Enable completion triggered by <c-x><c-o>
    vim.bo[ev.buf].omnifunc = 'v:lua.vim.lsp.omnifunc'

    -- Buffer local mappings.
    -- See `:help vim.lsp.*` for documentation on any of the below functions
    local opts = { buffer = ev.buf }
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
  end,
})

-- local capabilities = vim.lsp.protocol.make_client_capabilities()
-- capabilities = require("cmp_nvim_lsp").default_capabilities(capabilities)
local lspconfig = require('lspconfig')
lspconfig.scheme_langserver.setup {}
lvim.builtin.cmp.enabled()
lvim.builtin.cmp.sources = {
  { name = 'nvim_lsp' },
  { name = 'luasnip' },
  { name = 'buffer' },
}
NOTE: detailed configuration for lvim.builtin.cmp.sources and LspAttach can refer this page and this page.

Enable Multi-thread Response and Indexing
Current scheme-langserver haven't been fully tested in multi-thread scenario, and corresponding functionalities is default disabled. Please make sure you'd truly want to enable multi-thread features and the following changes were what you wanted.

Taking LunarVim as an example, steping above instructions and rewrite file ~/.local/share/lunarvim/site/pack/lazy/opt/nvim-lspconfig/lua/lspconfig/server_configurations/scheme_langserver.lua as follows:

local util = require 'lspconfig.util'
local bin_name = '{path-to-run}'
local cmd = { bin_name ,'{path-to-log}','enable'}

return {
  default_config = {
    cmd = cmd,
    filetypes = { 'scheme' },
    root_dir = util.find_git_ancestor,
    single_file_support = true,
  },
  docs = {
    description = [[
https://github.com/ufo5260987423/scheme-langserver
`scheme-langserver`, a language server protocol implementation for scheme
]]   ,
  },
}
TODO: Installation for VScode
Features
Top-level and local identifiers binding completion. Top-level and local identifiers binding
Goto definition. Goto definition with telescope.nvim
Compatible with package manager: Akku.
File changes synchronizing and corresponding index changing.
Hover.
References and document highlight (document-scoped references). Find references with telescope.nvim
Document symbol. Find document symbols with telescope.nvim
Catching *-syntax(define-syntax, let-syntax, etc.) based local identifier binding.
Cross-platform parallel indexing.
Self-made source code annotator to be compatible with .sps files.
Peephole optimization for API requests.
Type inference.
TODOs
Virtual identifier catching machine for .sps, .ss, .scm files.
Renaming.
Fully compatible with r6rs standard.
Macro expanding.
Code eval.
Code diagnostic.
Add cross-language semantic supporting. Well, would java, c, python and many other languages can be supported with an AST transformer?
TODO:Contributing
Debug
How to Debug
https://www.scheme.com/debug/debug.html#g1

Output Log
Following tips from Building, Installation for Lunar Vim and Installation for VScode, if anyone wants to do some developing and log something, it will be convenient to add path-to-log-file and re-write file ~/.local/share/lunarvim/site/pack/packer/start/nvim-lspconfig/lua/lspconfig/server_configurations/scheme_langserver.lua as follows:

local util = require 'lspconfig.util'
local bin_name = '{path-to-run}'
local cmd = { bin_name ,"path-to-log-file"}

return {
  default_config = {
    cmd = cmd,
    filetypes = { 'scheme' },
    root_dir = util.find_git_ancestor,
    single_file_support = true,
  },
  docs = {
    description = [[
https://github.com/ufo5260987423/scheme-langserver
`scheme-langserver`, a language server protocol implementation for scheme
]]   ,
  },
}
Recurring with Log
With above output log, you may use tests/log-debug.sps recurring bugs:

Rename {path-to-log}(usually ~/scheme-langserver.log) as ~/ready-for-analyse.log;
run scheme --script tests/log-debug.sps. If you want to re-produce the multi-thread environment, it would also be available to run scheme --script tests/log-debug.sps.
Test
Almost all key procedures and APIs are tested. My work is just so rough but useful, maybe you would like to find what I've done in tests directory or just run following command in {scheme-langserver-root-directory}

bash test.sh
NOTE It's hard to do test with threaded environment. So, current tests focus on single thread.

Code Count
find . -name "*.sls" ! -path "./.akku/*" |xargs wc -l
Detailed Document
Catching identifier bindings
Synchronizing
Type inference, 类型推断
API Analysis
Releases 6
run-1.0.11-ta6le
Latest
on Apr 30
+ 5 releases
Packages
No packages published
Languages
Scheme
99.9%
 
Shell
0.1%
Footer
© 2023 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
ufo5260987423/scheme-langserver: Scheme language server 
