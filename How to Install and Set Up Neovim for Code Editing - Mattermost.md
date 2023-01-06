PC versus Mac. iOS versus Android. Vim versus Emacs.

It seems that us devs just can’t help but compare our software tools to see who’s got the “best” tech stack.

To our credit, IDEs have come a long way over the course of the last half-century. Early attempts like Dartmouth BASIC — while offering support for debugging, editing, compiling, and more — are a far cry from the point-and-click graphical user interfaces that we’ve come to know and love. 

Moreover, the addition of tools such as code completion, continuous integration, syntax highlighting, and language server protocols have led to a suite of highly-functional, extremely convenient development environments.

(I mean, with GitHub Copilot, the program essentially writes itself… )

In light of all these modern conveniences, why would anyone in their right mind use something like [Vim](https://www.vim.org/)?

## Why Vim?

To be sure, many developers are surprised when they hear a colleague is working with the terminal-based text editor. But there are several reasons why someone might choose Vim for their modern workflow:

-   **It’s ergonomic.** As a developer, you’re probably spending most of your day in the terminal anyway. Your job is much more efficient when you don’t have to keep reaching for the mouse.
-   **It’s intuitive.** The Vim keybindings tend to make a lot of sense when it comes to moving around, manipulating text, and switching modes. You don’t have to think about what command you want to run because it’s seamlessly connected to the keys you press.
-   **It’s changeable.** In the event that a default keybinding is _not_ so intuitive, it’s a rather trivial process to change it to something else. Furthermore, prefix keys like the leader and local leader allow you to define additional namespaces for keymaps and even localize them to certain file types if desired.
-   **It’s everywhere.** Vim is installed by default on almost every server. If you have to SSH into a remote machine, you’ll know that Vim will always be there.

Even though Vim was originally released [way back in the last century](https://en.wikipedia.org/wiki/Vim_(text_editor)), it hasn’t stayed there. In fact, development has continued through the present day. And it’s even resulted in something new: a dynamic fork called [Neovim](https://neovim.io/).

## What is Neovim?

Neovim is an extension of Vim that has the potential to offer many of the same conveniences you’d expect from most modern IDEs—all while retaining the classic Vim functionality that so many developers are used to. What’s more, the latest versions of Neovim include an embedded scripting language, Lua, which offers much more control and extensibility to the user.

With the powerhouse combo of Neovim and Lua, you can customize your own, personal IDE that’s configured to your exact specifications. 

This article walks you through how to set up Neovim and write your config files entirely in Lua. It also introduces you to Neovim’s extensive plugin environment.

## Getting Started with Neovim

To get started, you’ll need to install the Neovim package.

### Install Neovim

There are several different ways to install Neovim on your system, depending on which OS you run. Windows, macOS, and Linux all have pre-built [packages](https://github.com/neovim/neovim/releases/tag/v0.6.0) that you can download and run directly. Other options include [installing Neovim via your package manager](https://github.com/neovim/neovim/wiki/Installing-Neovim#install-from-package) or even [building it directly from the source](https://github.com/neovim/neovim/wiki/Building-Neovim). This tutorial will use the pre-built package for the latest release.

Go to the Releases page on GitHub and grab the [latest release](https://github.com/neovim/neovim/releases/latest) of Neovim for your operating system (v0.6.0 at the time of this writing). Download the archive file and save it to your machine. On Linux, you can save the Neovim AppImage to the folder of your choice with the following command:

```
$ cd ~ && wget 
https://github.com/neovim/neovim/releases/download/v0.6.0/nvim.appimage
```

Once you’ve downloaded the file, you’ll need to extract the contents using your file manager or by running a shell command like `tar xzvf name-of-file.tar.gz`. If you’re using the AppImage and have FUSE installed on your system, you won’t need to extract the file. But you will need to modify the file permissions:

```
$ chmod u+x nvim.appimage
```

Now that you have Neovim installed, you can run it by either clicking the executable file or prepending a dot slash to the location of the binary:

```
$ ./nvim.appimage
```

**The author has aliased this command to `nvim` to improve readability throughout the rest of the tutorial.** You can learn about setting up and configuring aliases [here](https://www.tecmint.com/create-alias-in-linux/).

One of the things that makes Neovim so powerfully extensible is direct integration with the Lua programming language. Lua is embedded inside Neovim itself, offering power users direct access to the scripting language’s capabilities without forcing them to download any external dependencies.

You can get started writing Lua by simply opening up an empty Neovim buffer. Then, you’ll want to enter command mode by pressing the colon character (i.e., :). There, you’ll type `lua` followed by the command you want to execute. 

Try executing the command `:lua print("Hello, world!")`. The output should be rendered at the bottom of your screen. Again, you didn’t need to install Lua yourself in order to execute this code. The just-in-time compiler ([LuaJIT](https://luajit.org/)) comes bundled with this version of Neovim, so as long as you’re calling Lua code inside a Neovim buffer, it should execute without any trouble.

### Learning the Basics

If you’ve never used Vim, Neovim, or Lua before, then you might want to check out the following guides:

-   [Learn Vim in Y Minutes](https://learnxinyminutes.com/docs/vim/): This tutorial will quickly walk you through the basics of using Vim (and therefore Neovim). You’ll learn how to exit Vim (a frustrating task for many a developer), how to navigate using the Vim keys, how to toggle between different modes, and how to actually edit text. Everything that you learn in this tutorial will translate directly to your usage of Neovim.
-   [Learn Lua in Y Minutes](https://learnxinyminutes.com/docs/lua/): Likewise, this tutorial will introduce the syntax for the Lua programming language. You’ll learn how to set variables, work with different data types (like strings, numbers, and bools), structure conditional statements, define functions, and aggregate data in a Lua table.

### Configuration

Now that you’ve got Neovim running and you have a basic understanding of the Lua scripting language, it’s time to start customizing your config files.

Many tutorials that have been written so far are geared towards users who are new to Neovim but have used Vim in the past. In other words, a lot of tutorials show how to transition one’s configuration from the Vimscript file `init.vim` to the Neovim file `init.lua`.

That’s not what we’re going to do here! 

Instead, we’re going to take advantage of Lua’s position as a first-class language inside Neovim and build config files using this language from the start. The reader will get a better understanding of what they’ll find in the `init.vim` file, and the process they need to take to convert this to `init.lua`. They’ll configure line numbers, terminal colors, whitespace, custom keybinds, and more.

#### Create a configuration file

Neovim expects to find configuration files under the hidden directory `~/.config/nvim/` or `~/AppData/Local/nvim/`, depending on your operating system. If this folder does not exist, go ahead and create it now. On Linux, you’d run the following command:

```
mkdir ~/.config/nvim
```

Your base configuration file is going to be `init.lua`. Create this file in your Neovim configuration directory. You can use the following code block as a skeleton to get started:

```
--[[ init.lua ]]

-- LEADER
-- These keybindings need to be defined before the first /
-- is called; otherwise, it will default to "\"
vim.g.mapleader = ","
vim.g.localleader = "\\"

-- IMPORTS
-- require('vars')      -- Variables
-- require('opts')      -- Options
-- require('keys')      -- Keymaps
-- require('plug')      -- Plugins
```

Move it into your configuration directory, which should now look like this:

```
tree ~/.config/nvim
```

Then, open the file in Neovim by calling nvim and passing in the path to the file:

```
nvim ~/.config/nvim/init.lua
```

The first thing you’ll see is a section titled LEADER. Here, two **leader** keys are defined. You can use them to create personalized shortcuts by binding keys to custom commands. Feel free to change these values to whatever you’d like by replacing the character inside the string.

The next section is titled IMPORTS. These are other Lua files that Neovim will initialize whenever you run the program. Separating your code out into modular files is a good way to improve readability by compartmentalizing functionality into discrete chunks. That way, you can maintain all of your keybindings at once without getting bogged down in plugin configuration, for example.

### Call the Vim API

Neovim integrates extremely well with existing Vim functionality. The core development team has often stressed that the goal of Neovim is _not_ to displace Vim but to offer a fork of Vim that’s more focused on modern UI elements and extensibility. 

As a result, you’ll find that Neovim is capable of playing well with Vim in many ways. This allows you to quickly bootstrap your Lua configuration without having to implement that functionality yourself.

To do this, you’ll need to access the global vim variable. This acts as the entry point for calling Neovim functions directly in Lua.

In particular, the `vim.api` module allows you to control the configuration of your editor without writing a single line of Vimscript.

-   **vim.api.nvim\_set\_var** to set internal variables.
-   **vim.api.nvim\_set\_option** to set options.
-   **vim.api.nvim\_set\_keymap** to set key mappings.

#### Set internal variables

You can use `vim.api.nvim_set_var` to set internal variables. You may have noticed two of these sets already if you’re using the skeleton `init.lua` file:

```
vim.g.mapleader = ","
vim.g.localleader = "\\"
```

These lines of code set two global variables, the `<leader>` and `<localleader>` keys, to the comma and the backslash, respectively. These mappings allow you to set a prefix key that, when pressed, will activate a set of mappings that you can use to quickly run certain commands. 

Your leader key will work across all files you edit in Neovim, whereas the local leader key will be for a specific file type — like a Python file or a Bash script. It’s a good idea to set these [leader keys](https://learnvimscriptthehardway.stevelosh.com/chapters/06.html) before any other variables to ensure that they’re mapped correctly and are ready to work with any plugins you install.

Note that here you called `vim.g` instead of `vim.api.nvim_set_var`. That’s because Neovim offers a helpful set of **meta-accessors** that you can use to reduce the amount of code contained in your file. Some of the available meta-accessors for the `vim.api.nvim_set_var` command are as follows:

-   `vim.g:` maps to `vim.api.nvim_set_var`; sets global variables.
-   `vim.o:` maps to `vim.api.nvim_win_set_var`; sets variables scoped to a given window.
-   `vim.b`; maps to `vim.api.nvim_buf_set_var`; sets variables scoped to a given buffer.

The API also exposes other functions to get or delete the value of a variable. To access these functions, you would replace the set in the above commands with `get` and `del`, respectively. You can confirm that these values are set by running a Lua command in an open Neovim buffer. Press the colon to enter command mode, then tell Neovim to print the current value of the leader key:

```
:lua print(vim.api.nvim_get_var('mapleader'))
```

The character you assigned as the leader key should be displayed beneath the statusline.

Let’s use Lua to set one or two more global variables for this configuration. In addition to your `init.lua` file, Neovim will also look for any files that are included in the `/lua` subdirectory. All code contained in this subfolder is part of your **runtimepath** and can be imported for use in Neovim with the command `require('name-of-file')`.

Lua modules are found inside a `/lua` folder in your ’runtimepath’ (for most users, this will mean `~/.config/nvim/lua` on \*nix systems and `~/AppData/Local/nvim/lua` on Windows). You can `require()` files in this folder as Lua modules.

Create a new `/lua` directory inside of your Neovim directory, and add a file named `vars.lua`:

```
mkdir ~/.config/nvim/lua && touch ~/.config/nvim/lua/vars.lua
```

Open this file in Neovim and add the following lines of code:

```
--[[ vars.lua ]]

local g = vim.g
g.t_co = 256
g.background = "dark"
```

The first line further reduces the amount of code you have to write by aliasing the `vim.g` meta-accessor to a local Lua variable, _g_. The second line sets the global variable `t_co` to 256, enabling support for 256 colors in whichever terminal emulator you’re using to run Neovim. The final line sets the global variable background to tell Neovim whether your terminal background will use dark or light mode. These variables may come in handy when you’re solidifying your theme.

Save the file, then go back to your `init.lua` file and uncomment the line that imports this module:

```
-- IMPORTS
require('vars')         -- Variables: UNCOMMENT THIS LINE
-- require('opts')      -- Options
-- require('keys')      -- Keymaps
-- require('plug')      -- Plugins
```

Note that you don’t need to include the `.lua` file extension when you’re specifying the module to import. Now, whenever you run Neovim, it will load all code in the `lua/vars.lua` file into your runtime environment. You can confirm that the variables you defined in this file are indeed accessible globally by running `:lua print(vim.api.nvim_get_var('t_co'))` and `:lua print(vim.api.nvim_get_var('background'))` in command mode.

To learn more about a given variable, you can run the command `:help name-of-variable` in command mode. If it’s available, Neovim will point you to the relevant section in the docs for more information on what that variable means and what values you can assign to it.

You’ll set more variables as you start to install new plugins and configure their functionality to work across all file types.

#### Set options to customize your experience 

This is where you’ll really start to see your configuration take shape. You’ll spend most of your time setting options to tell Neovim how you want your development environment to operate both functionally and aesthetically. For instance, options allow you to determine settings for whitespace, line numbers, search functionality, file encodings, and more.

Like variables, you can set options in Lua by using the Neovim API. Here are some of the functions and meta-accessors that are available to you:

-   `vim.o:` maps to `vim.api.nvim_set_option`; equivalent to `:set`.
-   `vim.go:` maps to `vim.api.nvim_set_option`; equivalent to `:setglobal`.
-   `vim.bo`; maps to `vim.api.nvim_buf_set_option`; equivalent to `:setlocal` for buffer options.
-   `vim.wo:` maps to `vim.api.nvim_win_set_option`; equivalent to `:setlocal` for window options.

The API also exposes related functions to get the value of an option; to access them, you can replace `set` in the above commands with `get`.

Let’s set some global options for Neovim. Inside the `/lua` directory, add a file named `opts.lua`:

```
touch ~/.config/nvim/lua/opts.lua
```

Open the file and configure the following options:

```
--[[ opts.lua ]]
local opt = vim.opt

-- [[ Context ]]
opt.colorcolumn = '80'           -- str:  Show col for max line length
opt.number = true                -- bool: Show line numbers
opt.relativenumber = true        -- bool: Show relative line numbers
opt.scrolloff = 4                -- int:  Min num lines of context
opt.signcolumn = "yes"           -- str:  Show the sign column

-- [[ Filetypes ]]
opt.encoding = 'utf8'            -- str:  String encoding to use
opt.fileencoding = 'utf8'        -- str:  File encoding to use

-- [[ Theme ]]
opt.syntax = "ON"                -- str:  Allow syntax highlighting
opt.termguicolors = true         -- bool: If term supports ui color then enable

-- [[ Search ]]
opt.ignorecase = true            -- bool: Ignore case in search patterns
opt.smartcase = true             -- bool: Override ignorecase if search contains capitals
opt.incsearch = true             -- bool: Use incremental search
opt.hlsearch = false             -- bool: Highlight search matches

-- [[ Whitespace ]]
opt.expandtab = true             -- bool: Use spaces instead of tabs
opt.shiftwidth = 4               -- num:  Size of an indent
opt.softtabstop = 4              -- num:  Number of spaces tabs count for in insert mode
opt.tabstop = 4                  -- num:  Number of spaces tabs count for

-- [[ Splits ]]
opt.splitright = true            -- bool: Place new window to right of current one
opt.splitbelow = true            -- bool: Place new window below the current one
```

First, we’ve set a local variable opt so that we don’t have to type `vim.opt` for each option we want to set. Then, we define a section to group settings together. Organizing your config in this way will help you maintain your custom environment, as it’s easy to see the settings that each section is responsible for.

The first group, **Context**, defines options for how your lines render to the screen. Some of the options it sets are a visual indicator for the max number of characters you want on a line, as well as line numbers relative to the position of your cursor.

The **Filetypes** group configures Unicode compatibility, whereas the **Theme** group sets options that are related to Neovim’s look and feel. This section will expand as you continue to specify the desired aesthetics you want your installation of Neovim to have.

The last three sections configure basic functionality for handling whitespace, optimizing search, and splitting the screen in a more intuitive fashion.

Feel free to change any (or all!) of these settings and omit those you don’t want. The point of configuring Neovim with Lua is to _customize_ _your own personal development environment_ and make it perfect for your own needs. The options shown here are simply a few ideas to get started with.

Once your options file is configured to your liking (for now, don’t worry — you’ll be making changes to all these files as you continue to expand on your configuration in the future), you can go back to your `init.lua` skeleton file and uncomment its line so that Neovim knows to import the module:

```
-- IMPORTS
require('vars')         -- Variables
require('opts')         -- Options: UNCOMMENT THIS LINE
-- require('keys')      -- Keymaps
-- require('plug')      -- Plugins
```

As always, you can confirm that Neovim is importing your file correctly by printing one of the options in command mode.

If you’re ever confused about what a particular option is, you can run the command `:help name-of-option` in command mode. If it’s available, Neovim will point you to the relevant section in the docs for more information on what that option is and what values you can give to it.

#### Set key mappings to improve productivity

Vim is loved by developers, system administrators, and others who may spend most of their days in the terminal. For such individuals, productivity and efficiency can increase when your fingers leave the keyboard as little as possible.

The Vim keybindings ensure that you don’t need to reach for the mouse every few seconds to change something on your screen. However, sometimes the default keymaps are a bit unintuitive or require you to stretch your fingers farther than you’d like. 

For example, to return to normal mode from insert mode, you have to press the ESCAPE key — which, though not as far away as the mouse might be, is still quite a bit of a reach!

Thankfully, it’s easy to define your own custom keybindings — even ones that override Neovim’s default functionality. Let’s remap the ESCAPE key to something a bit more intuitive, like “jk”. In the `/lua` directory, run Neovim with a new filename, `keys.lua`:

```
nvim keys.lua
```

Your file should contain the following lines of code:

```
--[[ keys.lua ]]
local map = vim.api.nvim_set_keymap

-- remap the key used to leave insert mode
map('i', 'jk', '', {})
```

Just like with settings options and variables, you call out to the Neovim API in order to set keybindings from Lua. The function you need to access is vim.api.nvim\_set\_keymap, which has been assigned to the local variable map here.

This function takes in four parameters:

1.  **mode**: the mode you want the key bind to apply to (e.g., insert mode, normal mode, command mode, visual mode).
2.  **sequence**: the sequence of keys to press.
3.  **command**: the command you want the keypresses to execute.
4.  **options**: an optional Lua table of options to configure (e.g., silent or noremap).

In the code block above, the defined keymapping is available in _insert mode_. The _sequence_ of keys to press is ‘jk’, and this will call the _escape command_ (returning you to normal mode). There are no additional options configured here. But you’ll still need to pass in curly braces to signify an empty table.

If you save the file and try to use your new keybinding straight away, you might notice that it doesn’t work. That’s because you’ll usually need to **refresh** your Neovim buffer before any configuration changes will take effect. You won’t need to close the file and open a new Neovim instance (although this would work as well). Instead, you can simply **source** the current file from command mode by calling `:luafile %` .

Now, your keymapping should work!

Don’t forget to uncomment the line that imports this new module in your init file:

```
-- IMPORTS
require('vars')         -- Variables
require('opts')         -- Options
require('keys')         -- Keymaps: UNCOMMENT THIS LINE
-- require('plug')      -- Plugins
```

To keep your config as clean as possible, it’s recommended that you define any future keybindings in this file.

## Congratulations! You’ve reached the end of the first part of this tutorial

By now, you’ve learned the basics of Neovim, how it’s directly integrated with the Lua programming language, and some ways you can customize your environment to increase your productivity.

In the second part of this tutorial, [we’ll explore how to install plugins in Neovim](https://mattermost.com/blog/turning-neovim-into-a-full-fledged-code-editor-with-lua/) to add new capabilities and accomplish even more. Stay tuned!

_This blog post was created as part of the Mattermost_ [_Community Writing Program_](https://handbook.mattermost.com/contributors/contributors/ways-to-contribute/community-content-program) _and is published under the_ [_CC BY-NC-SA 4.0 license_](https://creativecommons.org/licenses/by-nc-sa/4.0/)_. To learn more about the Mattermost Community Writing Program,_ [_check this out_](https://mattermost.com/blog/blog-announcing-community-writing-program/)_._