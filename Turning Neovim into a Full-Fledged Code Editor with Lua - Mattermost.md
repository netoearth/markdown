In the [first installment](https://mattermost.com/blog/how-to-install-and-set-up-neovim-for-code-editing/) in this tutorial, we explored why Neovim and Lua are a perfect match — and what you can do out of the box to customize Neovim so you can accomplish more every time you sit down to code. 

Now, it’s time to examine how to tap into even more potential with Neovim and Lua by extending the code editor with additional plugins to unlock new capabilities. 

## Install plugins to add new capabilities

This is where the combination of Neovim + Lua really shines. 

The integration of third-party plugins into your Neovim configuration means that you have _unlimited extensibility_ when it comes to personalizing your editor.

Plugins allow you to add functionality to Neovim in the form of git integration, fuzzy finders, file browsing, syntax highlighters, autocompletion, terminal integration, debugging, collaborative editing… The list of [awesome Neovim plugins](https://github.com/rockerBOO/awesome-neovim) is _huge_ and growing by the day.

Let’s set up a few basics that will give you a solid foundation for using Neovim as your editor of choice.

### Manage your plugins with Packer

The first thing you’ll need is a **plugin manager**. While it’s possible to manually install these plugins yourself, it’s much simpler for you to use a solid tool that can take care of installing, updating, and removing packages on your behalf.

One of the most popular package managers for Vim and Neovim alike is [vim-plug](https://github.com/junegunn/vim-plug/). While there is a way to [use vim-plug in Lua](https://vonheikemen.github.io/devlog/tools/configuring-neovim-using-lua/#vim-plug-in-lua), the goal of this tutorial is to respect Lua’s place as a first-class language in the latest releases of Neovim. As such, let’s learn how to use a package manager that’s actually written in Lua itself: [Packer](https://github.com/wbthomason/packer.nvim).

Inspired by the Emacs [use-package macro](https://github.com/jwiegley/use-package), Packer is built on top of Vim’s native functionality for handling packages. It offers several useful features — including lazy loading (for reducing startup time), using Luarocks, and working directly with git branches.

Packer needs to be installed somewhere on your packpath. These are directories that Neovim uses to find packages. To see what locations on your file system are currently in your packpath, you can execute the following Lua command in command mode: 

```
:lua print(vim.o.packpath) 
```

The output should return several comma-separated values showing the absolute filepaths of various locations on your system that Neovim will look through when loading packages. For instance, the first entry in the author’s packpath is `/home/jayascript/.config/nvim`. In other words, Packer can be installed in the same directory that’s been used to configure Neovim so far.

It turns out this is a good solution, as it keeps all of your Neovim configurations — packages and all — in a single directory.

Let’s go ahead and install Packer to this directory. On Unix systems, you can simply clone the repository into the desired directory:

```
git clone --depth 1 https://github.com/wbthomason/packer.nvim\
~/.config/nvim/site/pack/packer/start/packer.nvim
```

This should create a new folder `/site` in your Neovim configuration directory. You can take a look at all of the new files and folders created in this directory. This is where Packer will install and configure any plugins you specify.

First, you’ll need to tell Neovim that it should use the Packer plugin. In your `/lua` subdirectory, open a new file in Neovim called `plug.lua` and add the following lines of code:

```
-- [[ plug.lua ]]

return require('packer').startup(function(use)
  -- [[ Plugins Go Here ]]
end,
config = {
  package_root = vim.fn.stdpath('config') .. '/site/pack'
})
```

This code block says to load the Packer module each time Neovim is started. It also sets the package\_root to the location where you cloned the Packer repository. Next, uncomment the line that imports your plugins module in your `init` file:

```
-- IMPORTS
require('vars')         -- Variables
require('opts')         -- Options
require('keys')         -- Keymaps
require('plug')         -- Plugins: UNCOMMENT THIS LINE
```

Now, when you restart Neovim, you might see an error message that reads module ‘packer’ not found. If this happens, then you’ll need to update the packpath variable to ensure Neovim knows where to find the Packer installation. You can do this by adding the following lines of code to `vars.lua`:

```
--[[ vars.lua ]]

local g = vim.g
g.t_co = 256
g.background = "dark"

-- Update the packpath
local packer_path = vim.fn.stdpath('config') .. '/site'
vim.o.packpath = vim.o.packpath .. ',' .. packer_path
```

Here, you create a local variable `packer_path` that points to the new site folder in your Neovim configuration directory, the same one that was created when you cloned the Packer repository. Then, you update the packpath by appending this location to the existing packpath. (The double dots, i.e., `..`, is the Lua syntax for string concatenation.)

Now, open a new buffer in Neovim and you should see that this error has been resolved.

You can confirm that Packer was successfully installed and located by running `:PackerStatus` in command mode. You’ll probably see an error message that reads something like this: 

```
packer_plugins table is nil! Cannot run packer.status()!
```

All this means is that Packer has no plugins to operate on. Not to worry! We can fix this quite easily. Let’s get started adding some plugins to your Neovim configuration.

### Browse files with nvim-tree

One of the first things you might want to install is a more intuitive file browser. Neovim has several options for you to choose from; [NERDTree](https://github.com/preservim/nerdtree) is a popular one. 

However, in keeping with the theme of choosing Lua-first alternatives, this tutorial will use [nvim-tree](https://github.com/kyazdani42/nvim-tree.lua), which is written in Lua.

To install a new plugin, open `plug.lua` and add a new entry to the Packer startup function:

```
To install a new plugin, open plug.lua and add a new entry to the Packer startup function:
-- [[ plug.lua ]]

return require('packer').startup(function(use)
  -- [[ Plugins Go Here ]]
  use {                                              -- filesystem navigation
    'kyazdani42/nvim-tree.lua',
    requires = 'kyazdani42/nvim-web-devicons'        -- filesystem icons
  }
end,
config = {
  package_root = vim.fn.stdpath('config') .. '/site/pack'
})
```

For each new plugin you want to install, you’ll declare a **use-package** statement. This declaration generally follows the form of use `{ 'username/repository' }` for packages installed from GitHub. This way, Packer will be able to update any specified plugins by pulling down changes directly from the repository.

Then, run the command `:PackerInstall` in command mode. A split should pop up showing that Packer has pulled the code down from GitHub and installed the plugins. If anything goes wrong, it’ll show an error message that you can use to troubleshoot any problems.

After you install the plugin, you should tell Neovim about it to ensure you don’t bump up against any errors. Add a new section to your init file and make sure that `nvim-tree` is a required import:

```
-- IMPORTS
require('vars')         -- Variables
require('opts')         -- Options
require('keys')         -- Keymaps
require('plug')         -- Plugins

-- PLUGINS: Add this section
require('nvim-tree').setup{}
```

Now that your file browser is installed and configured, you’ll need to set up a keybinding to toggle it. Do this by adding the following lines of code to your `keys.lua` file:

```
--[[ keys.lua ]]
local map = vim.api.nvim_set_keymap

-- remap the key used to leave insert mode
map('i', 'jk', '', {})

-- Toggle nvim-tree
map('n', 'n', [[:NvimTreeToggle]], {})
```

This new keymapping is available in _normal mode_. The _key sequence_ uses your leader key (which you defined in your init file) as well as the letter _n_, and the _command_ to execute is the one that toggles nvim-tree. Once you’ve saved and sourced all your files, you should be able to toggle the filetree open and closed. You can also jump between the filetree and any open files you have by pressing CTRL+w and then h, j, k, or l, depending on which direction you want to move. This will make it much faster for you to build out your custom configuration, as you can easily switch back and forth between each file using the tree.

### Lookin’ good

This section will help you to configure your editor aesthetics. Using Packer, you’ll install the following plugins:

-   [**Startify**](https://github.com/mhinz/vim-startify): start screen for Neovim.
-   [**Beacon**](https://github.com/DanilaMihailov/beacon.nvim): helps you see your cursor jump.
-   [**Lualine**](https://github.com/nvim-lualine/lualine.nvim): a statusline written in Lua.
-   [**Dracula**](https://github.com/Mofiqul/dracula.nvim): a Neovim colorscheme written in Lua.

Let’s add a **Theme** section to the plugins file and tell Packer to install them:

```
-- [[ plug.lua ]]

return require('packer').startup(function(use)
  -- [[ Plugins Go Here ]]
  use {                                              -- filesystem navigation
    'kyazdani42/nvim-tree.lua',
    requires = 'kyazdani42/nvim-web-devicons'        -- filesystem icons
  }

  -- [[ Theme ]]
  use { 'mhinz/vim-startify' }                       -- start screen
  use { 'DanilaMihailov/beacon.nvim' }               -- cursor jump
  use {
    'nvim-lualine/lualine.nvim',                     -- statusline
    requires = {'kyazdani42/nvim-web-devicons',
                opt = true}
  }
  use { 'Mofiqul/dracula.nvim' }                     -- colorscheme
end,
config = {
  package_root = vim.fn.stdpath('config') .. '/site/pack'
})
```

Run `:PackerInstall` to add the new plugins.

Exit Neovim and open an empty buffer. You should see Startify enabled, with a quote from [cowsay](https://cowsay.morecode.org/) at the top of the screen. You should be able to quickly jump to a recent file for editing. If you jump up and down using the Vim keys, you should see that Beacon is enabled as well.

However, the Lualine may not be showing up, and the colors are still kind of ugly. You’ll need to make a few additional configurations to solve these issues.

Use Startify to open `opts.lua` and add the following to your **Theme** section:

```
--[[ opts.lua ]]
local opt = vim.opt
local cmd = vim.api.nvim_command

-- Snip...

-- [[ Theme ]]
opt.syntax = "ON"                -- str:  Allow syntax highlighting
opt.termguicolors = true         -- bool: If term supports ui color then enable
cmd('colorscheme dracula')       -- cmd:  Set the colorscheme
```

At the top of the file, you alias the Neovim API command to `cmd`, which allows you to easily run Vim functions in Lua. Then, you call the command you want to execute (here, the one for setting the colorscheme) in the relevant section of your configuration file.

Go ahead and refresh the buffer with `:luafile %`. The colors should update automatically. Much better!

The last thing you’ll do in this section is configure Lualine. Use `nvim-tree` to navigate to your `init` file and add this code to the very bottom:

```
-- PLUGINS
require('nvim-tree').setup{}
-- Add the block below
require('lualine').setup {
  options = {
    theme = 'dracula-nvim'
  }
}
```

[Dracula](https://draculatheme.com/) is an extremely popular colorscheme and has ready-to-use themes for everything from Slack to Emacs to Xresources and beyond. Using the Dracula theme ensures that the colors for Neovim as well as Lualine are in sync. This block of code imports the Lualine module and calls its setup function, setting the theme to dracula-nvim, which comes bundled with Lualine for your convenience.

Source the file again with `:luafile %` and watch your newly configured statusline appear right before your very eyes!

### Develop code

Lastly, we’ll configure a few plugins that will assist you in using Neovim as your IDE of choice when it comes to programming:

-   [**Telescope**](https://github.com/nvim-telescope/telescope.nvim): to quickly find files.
-   [**Tagbar**](https://github.com/preservim/tagbar): to view the structure of any classes or functions defined in a given file.
-   [**indentLine**](https://github.com/Yggdroot/indentLine): to more clearly show indent lines.
-   [**Fugitive**](https://github.com/tpope/vim-fugitive): to integrate with git repositories.
-   [**GV**](https://github.com/junegunn/gv.vim): to easily browse commit history.
-   [**autopairs**](https://github.com/windwp/nvim-autopairs): to automatically close brackets, parentheses, curly braces, and so on.

Let’s add a **Dev** section to the plugins file and tell Packer to install them:

```
-- [[ plug.lua ]]

return require('packer').startup(function(use)
  -- [[ Snip... ]]
  -- [[ Dev ]]
  use {
    'nvim-telescope/telescope.nvim',                 -- fuzzy finder
    requires = { {'nvim-lua/plenary.nvim'} }
  }
  use { 'majutsushi/tagbar' }                        -- code structure
  use { 'Yggdroot/indentLine' }                      -- see indentation
  use { 'tpope/vim-fugitive' }                       -- git integration
  use { 'junegunn/gv.vim' }                          -- commit history
  use { 'windwp/nvim-autopairs' }                    -- auto close brackets, etc.
end,
config = {
  package_root = vim.fn.stdpath('config') .. '/site/pack'
})
```

Run `:PackerInstall` to install the new plugins. You can check that everything is working by trying the following commands in command mode (where `<cr>` is the RETURN key):

-   `:Telescope find_files<cr>` (you can press ESC twice to quit)
-   `:TagbarToggle<cr>` (run this twice to close it)
-   `:IndentLinesToggle` (you may not see any lines show up just yet)
-   `:Git` (this isn’t a git repository, so this should show an error message)
-   `:GV` (same as above, but check that you _do_ see an error message that says not in git repo or something similar)

Everything seems to be working okay. But there are a few changes that can be made to better integrate these new additions into the configuration. 

First, let’s work on the autopairs, since there was no check for this. Head on over to your `init` file to add a new import:

```
-- PLUGINS
require('nvim-tree').setup{}
require('lualine').setup {
  options = {
    theme = 'dracula-nvim'
  }
}
require('nvim-autopairs').setup{} -- Add this line
```

Source the file with `:luafile %` and confirm that autopairs are enabled by inserting an open bracket or parenthesis in insert mode.

Now let’s add some keybindings to toggle the other plugins. Save the following Python code to your machine as `toggle_test.py`:

```
# toggle_test.py
def toggle_test(lines):
   print("Can you see the indent lines?")
   for line in lines:
     if line == "visible":
       print("Hooray!")
       print("I can see them.")
     else:
       print("Uh-oh...")
       print("Something's wrong!")
```

You don’t need to install Python or run this file. Just open it up in Neovim and make sure you can see the indent lines.

In your `keys.lua` file, add a few new keybindings:

```
--[[ keys.lua ]]
local map = vim.api.nvim_set_keymap

-- Snip...

-- Toggle more plugins
map('n', 'l', [[:IndentLinesToggle]], {})
map('n', 't', [[:TagbarToggle]], {})
map('n', 'ff', [[:Telescope find_files]], {})
```

Save and source the file. Then, use your leader key and ff to open Telescope. Search for the `toggle_test.py` Python file and press Enter to go to it. Practice using your leader key and l to toggle the indent lines on and off. The combination of your leader key and t will open up the tagbar, where you can examine the structure of the Python function.

Lastly, let’s make sure that your editor is capable of integrating with git. 

Navigate to a git repository on your local machine and open a Neovim buffer. Run the command `:Git status` and confirm that you can see the current state of the repo. Then, run `:GV` to open up the commit history. You can use the Vim keys to browse the commits and press Enter to view more detailed information. The resulting window will show the commit hash, author, message, and changes that were committed to the repository at that time.

As always, you can `:q` to close any windows and exit Neovim completely. But you already knew that by now, didn’t you? 😉

## What this looks like in practice

To give you a better idea of what this looks like in the real world, here’s a look at what my own personal customizations look like:

![Neovim and Lua](https://mattermost.com/wp-content/uploads/2022/02/Screen-Shot-2022-01-06-at-4.42.14-PM.webp)

## Where to go next with Neovim

Truth be told, we haven’t even _scratched_ the surface when it comes to all of the ways you can configure Neovim to your exact specifications. However, the steps you took here should be sufficient to get you started. 

So, where do you go from here? 

It’s highly recommended that you spend some time reading the docs for each plugin configured here. There may be additional functionality you wish to enable (or disable), and they often come with default keybindings that will help you to use them more efficiently. (For instance, `nvim-tree` has an array of keymaps that you can use to create, rename, and delete files and directories all without leaving Neovim.)

Feel free to set, unset, and map options and keys to your heart’s content. When you’re ready to expand your Neovim environment, there are dozens if not hundreds of additional plugins you can install to improve productivity and enhance your setup. 

Here are some ideas to get started:

-   [Floaterm](https://github.com/voldikss/vim-floaterm): floating terminal manager.
-   [Impatient](https://github.com/lewis6991/impatient.nvim): improve Neovim’s startup time.
-   [Commentary](https://github.com/tpope/vim-commentary): quickly toggle comments in a variety of programming languages.
-   [Undotree](https://github.com/mbbill/undotree): persistent undo; reverse changes even after a file has been closed.
-   [Vimwiki](https://github.com/vimwiki/vimwiki): craft a personal knowledgebase, entirely in Neovim.
-   [Surround](https://github.com/tpope/vim-surround): easily surround chunks of text with brackets, quotes, tags, and more.
-   [Treesitter](https://github.com/nvim-treesitter/nvim-treesitter): and related Language Server Protocol (LSP) tools.

## Conclusion

Thanks to a team of passionate developers, Neovim has become one of the most-loved IDEs in the modern age. Though the config you built through this tutorial was relatively bare-bones, it’s (dare I say) possible to extend Neovim to rival even VSCode.

That being said, Neovim is still in its early stages, with breaking changes coming on an almost nightly basis. Even so, this means that users can only look forward to more plugins and better functionality as work continues on its core.

The combination of Neovim and Lua offers a powerhouse of extensibility for devs who want to stay in the terminal and like to determine every single aspect of how their environment works. And if you store your configs in a .dotfiles repository, you can replicate your setup wherever you code.

For more information, check out [the official Neovim repository](https://github.com/neovim/neovim) as well as this [guide to Neovim + Lua](https://github.com/nanotee/nvim-lua-guide).

_This blog post was created as part of the Mattermost_ [_Community Writing Program_](https://handbook.mattermost.com/contributors/contributors/ways-to-contribute/community-content-program) _and is published under the_ [_CC BY-NC-SA 4.0 license_](https://creativecommons.org/licenses/by-nc-sa/4.0/)_. To learn more about the Mattermost Community Writing Program,_ [_check this out_](https://mattermost.com/blog/blog-announcing-community-writing-program/)_._