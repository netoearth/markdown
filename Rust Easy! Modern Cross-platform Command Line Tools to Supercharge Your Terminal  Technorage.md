Rust is taking over the terminal. Rust is a general-purpose programming language that is blazing fast and memory safe. It is the fastest-growing and most loved programming language in the world. It is used to build everything from operating systems to web servers to command-line tools. Recently there has been a surge of command line tools and utilities written in Rust, and many of them are intended to replace standard Unix commands. They are faster, more user-friendly, and have more features than their standard Unix counterparts. In this post, I will cover some of the best Rust command line tools I have used for a while. You can also use these to supercharge your terminal.

These tools are available for both Linux and macOS. I have not tested them on Windows, but most should also work on Windows. I recommend aliasing the commands to replace the standard commands based on your preferences. If you have [Cargo](https://doc.rust-lang.org/cargo/), the rust package manager, you can install all these using Cargo.

## Alacritty

Let us start with the terminal itself. [Alacritty](https://github.com/alacritty/alacritty) is a cross-platform modern terminal emulator with sensible defaults. It is **GPU accelerated**, super fast, and highly configurable. You can use it on Linux, macOS, and Windows. It doesn’t have much in terms of a UI, and hence all [configurations](https://github.com/alacritty/alacritty/releases/download/v0.11.0/alacritty.yml) are done through YAML files. I don’t use it as my primary terminal as I love [Yakuake](https://invent.kde.org/utilities/yakuake) too much for all its cool features. We can get most of those features (tabs, split panes, dropdown mode) using tmux and [tdrop](https://github.com/noctuid/tdrop) if really needed. I use Alacrity when I need speed and GPU acceleration. There is an excellent tutorial on [using Alacritty with tmux](https://arslan.io/2018/02/05/gpu-accelerated-terminal-alacritty/). You could also use [Zellij](https://github.com/zellij-org/zellij), a modern terminal multiplexer written in Rust, with Alacritty.

There is also the [Warp](https://www.warp.dev/) terminal, but it is not open source. It is a great terminal, but I prefer open source software. Thanks to [Fran Sancisco](https://dev.to/francisc) for the suggestion.

![Alacritty](https://i.imgur.com/XPYyJof.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
# Arch Linux
yay -S alacritty
# Fedora/CentOS
dnf copr enable atim/alacritty
dnf install alacritty
# Debian/Ubuntu
add-apt-repository ppa:aslatter/ppa
apt install alacritty
# macOS Homebrew
brew install --cask alacritty
# Windows Scoop
scoop bucket add extras
scoop install alacritty
# Cargo on any
cargo install alacritty
```

## Starship

[Starship](https://starship.rs/) is the best terminal prompt I have ever used. Forget [Oh My Zsh](https://ohmyz.sh/) and stuff like that. Starship is fast, highly customizable, and has a great default theme and settings. I didn’t even change most of the default settings, as things were perfect as it is. Starship works on shells like zsh, fish, and bash and can also work alongside other prompts like Oh My Zsh, in case you still want to use Oh My Zsh for other plugins like autosuggestions and so on. Starship works best with a [Nerd Font](https://www.nerdfonts.com/) as it can show icons and ligatures based on context. I used Oh My Zsh for many years with the [powerlevel10k](https://github.com/romkatv/powerlevel10k) theme, but the prompt was a bit slow. Starship is blazing fast with more features and an excellent UX.

![starship](https://i.imgur.com/bPmhPKH.gif)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S starship
# Fedora/CentOS
dnf install starship
# Debian/Ubuntu
curl -sS https://starship.rs/install.sh | sh
# macOS/Linux Homebrew
brew install starship
# macOS MacPorts
port install starship
# Windows Scoop
scoop install starship
# Cargo
cargo install starship --locked
```

## bat

[bat](https://github.com/sharkdp/bat) is one of my favorite tools from this list. It’s a replacement for , and once you have used , you will never go back. It provides features like syntax highlight, line numbers, Git change highlight, shows special chars, paging, and so on. It is super fast and looks beautiful. I have aliased to immediately after trying it for the first time. By default, bat behaves similarly to by paging large output, but that can be disabled to make it work precisely like . It can be used as a drop-in replacement for even in scripts. can also be used as a previewer for [fzf](https://github.com/junegunn/fzf). It can also be combined with many other commands and tools like , , and , among others, to add syntax highlighting to outputs. Syntax highlighting themes are configurable.`cat``bat``cat``bat``less``cat``cat``bat``tail``man``git`

![bat](https://i.imgur.com/46r4uom.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S bat
# Fedora/CentOS
dnf install bat
# Debian/Ubuntu
apt install bat
# macOS/Linux Homebrew
brew install bat
# macOS MacPorts
port install bat
# Windows Scoop
scoop install bat
# Cargo
cargo install bat --locked
```

## LSD and exa

Both [LSD](https://github.com/Peltoche/lsd) and [exa](https://github.com/ogham/exa) are replacements for the command. They both look gorgeous with nice colors and icons and have features like headers, sorting, tree views, and so on. Exa is a bit faster than LSD for tree views and can show the Git status of files and folders. I prefer exa due to the Git support and faster tree views. I have set up my alias to use exa by default. Both can be configured to show custom columns and sorting behaviors.`ls``ls`

![lsd-exa](https://i.imgur.com/KsMv5xG.png)

### exa Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
# Arch Linux
yay -S exa
# Fedora/CentOS
dnf install exa
# Debian/Ubuntu
apt install exa
# macOS Homebrew
brew install exa
# Cargo
cargo install exa

# Alias ls to exa
alias ls='exa --git --icons --color=always --group-directories-first'
```

### LSD Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
# Arch Linux
yay -S lsd
# Fedora/CentOS
dnf install lsd
# Debian/Ubuntu
dpkg -i lsd_0.23.1_amd64.deb # get .deb file from https://github.com/Peltoche/lsd/releases
# macOS Homebrew
brew install lsd
# macOS MacPorts
port install lsd
# Windows Scoop
scop install lsd
# Cargo
cargo install lsd

# Alias ls to lsd
alias ls='lsd --header --color=always --group-directories-first'
```

## rip

[rip](https://github.com/nivekuil/rip) is an improved version of the command. It is faster, safer, and user-friendly. rip sends deleted files to a temp location so they can be recovered using . I really like the simplicity and the revert feature, as I don’t have to worry about accidentally deleting something using . While rip can be aliased to replace , the creators advise not doing that as you might get used to it and do on other systems where you cannot revert the delete.`rm``rip -u``rm``rm``rm`

### Installation

```
1
2
3
4
5
6
7
8
# Arch Linux
yay -S rm-improved
# Fedora/CentOS/Debian/Ubuntu
# Install from binary or build locally using Cargo
# macOS Homebrew
brew install rm-improved
# Cargo
cargo install rm-improved
```

## xcp

[xcp](https://github.com/tarka/xcp) is a partial clone of the command. It is faster and more user-friendly with progress bars, parallel copying, support, and so on. I like its simplicity and developer experience, especially the progress bars. I have aliased to so I can use it everywhere.`cp``.gitignore``cp``xcp`

![xcp](https://i.imgur.com/NOnkDyx.png)

### Installation

```
1
2
3
4
5
6
7
8
9
# Arch Linux
yay -S xcp
# Fedora/CentOS/Debian/Ubuntu/macOS
# Install from binary or build locally using Cargo
# Cargo
cargo install xcp

# Alias cp to xcp
alias cp='xcp'
```

## zoxide

[zoxide](https://github.com/ajeetdsouza/zoxide) is a smarter replacement. It remembers the directories you visit, and you can jump to them without providing a full path. You can provide partial paths or even a word from the path. When there are similar paths, zoxide offers an interactive selection using [fzf](https://github.com/junegunn/fzf). It is super fast and works with all major shells. I like how it works, and I have aliased to so I can use it everywhere.`cd``cd``z`

![zoxide](https://i.imgur.com/OTZS3yu.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S zoxide
# Fedora/CentOS
dnf install zoxide
# Debian/Ubuntu
apt install zoxide
# macOS/Linux Homebrew
brew install zoxide
# macOS MacPorts
port install zoxide
# Windows Scoop
scoop install zoxide
# Cargo
cargo install zoxide --locked
```

Once installed, you must add the following to your shell config file. For other shells, refer the [docs](https://github.com/ajeetdsouza/zoxide#step-2-add-zoxide-to-your-shell)

```
1
2
3
4
5
6
7
8
9
# bash (~/.bashrc)
eval "$(zoxide init bash)"
# zsh (~/.zshrc)
eval "$(zoxide init zsh)"
# fish (~/.config/fish/config.fish)
zoxide init fish | source

# Alias cd to z
alias cd='z'
```

## dust

[Dust](https://github.com/bootandy/dust) is an alternative for the command. It is fast and has a better UX with nice visualization for disk usage.`du`

![dust](https://i.imgur.com/wfYJPqn.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S dust
# Fedora/CentOS
# Install binary from https://github.com/bootandy/dust/releases
# Debian/Ubuntu
deb-get install du-dust
# macOS Homebrew
brew install dust
# macOS MacPorts
port install dust
# Windows Scoop
scoop install dust
# Cargo
cargo install du-dust
```

## ripgrep

[ripgrep (rg)](https://github.com/BurntSushi/ripgrep) is a line-oriented search tool that recursively searches your current directory for a regex pattern. It is faster than and has many features like compressed files search, colorized output, smart case, file type filtering, multi-threading, and so on. It understands files and skips hidden and ignored files. [Here](https://beyondgrep.com/feature-comparison/) is a feature comparison with other similar tools, and yes, it is faster than all the other tools in the list.`grep``.gitignore`

![ripgrep](https://i.imgur.com/bi8838T.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S ripgrep
# Fedora/CentOS
dnf install ripgrep
# Debian/Ubuntu
apt-get install ripgrep
# macOS/Linux Homebrew
brew install ripgrep
# macOS MacPorts
port install ripgrep
# Windows Scoop
scoop install ripgrep
# Cargo
cargo install ripgrep
```

## fd

[fd](https://github.com/sharkdp/fd) is a simpler alternative to . It is more intuitive to use and comes with sensible defaults. It is extremely fast due to parallel traversing and shows a modern colorized output and supports patterns and regex, parallel commands, smart case, understands files, and so on. I have aliased to as I could never remember what options to pass to get a basic find command working.`find``.gitignore``find``fd`

![fd](https://i.imgur.com/tcSzQ4S.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S fd
# Fedora/CentOS
dnf install fd-find
# Debian/Ubuntu
apt install fd-find
# macOS Homebrew
brew install fd
# macOS MacPorts
port install fd
# Windows Scoop
scoop install fd
# Cargo
cargo install fd-find
```

## sd

[sd](https://github.com/chmln/sd) is a find-and-replace CLI, and you can use it as a replacement for and . It is way more user-friendly and modern. It is also magnitudes faster than .`sed``awk``sed`

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
# Arch Linux
yay -S sd
# Fedora/CentOS
dnf install sd
# Debian/Ubuntu
# Install binary from the release page
# macOS Homebrew
brew install sd
# Windows Scoop
choco install sd-cli
# Cargo
cargo install sd
```

## procs

[procs](https://github.com/dalance/procs) is a replacement. It provides colorized human-readable output, multi-column search, more information than , docker support, paging, watch mode, and tree view. It is a much more user-friendly and modern alternative to . You can filter by name and PID and use logical and/or operators to combine multiple filters. It also has a tree view which is very useful for seeing the process hierarchy. It can also show docker container names for the process running docker containers.`ps``ps``ps`

![procs](https://i.imgur.com/nvho7hM.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S procs
# Fedora/CentOS
dnf install procs
# Debian/Ubuntu
# Install binary from the release page
# macOS Homebrew
brew install procs
# macOS MacPorts
port install procs
# Windows Scoop
scoop install procs
# Cargo
cargo install procs
```

## bottom

[bottom](https://github.com/ClementTsang/bottom) is a replacement with a nice terminal UI. It’s quite feature-rich and customizable.`top`

![bottom](https://i.imgur.com/nbL8gBi.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
# Arch Linux
yay -S bottom
# Fedora/CentOS
dnf copr enable atim/bottom -y
dnf install bottom
# Debian/Ubuntu
dpkg -i bottom_0.6.8_amd64.deb
# macOS Homebrew
brew install bottom
# macOS MacPorts
port install bottom
# Windows Scoop
scoop install bottom
# Cargo
cargo install bottom --locked
```

## Topgrade

[Topgrade](https://github.com/topgrade-rs/topgrade) is a fantastic utility if you prefer to keep your system up-to-date, like me. It detects most of the package managers on your system and triggers updates. It is configurable, so you can configure it to ignore certain package managers. On my system, it detected pacman, SDKMAN, Flatpak, snap, Homebrew, rustup, Linux firmware, Pip, and so on. Topgrade is cross-platform; you can use it on Windows, macOS, and Linux.

![topgrade](https://i.imgur.com/7PXryFh.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
# Arch Linux
yay -S topgrade
# Fedora/CentOS/Debian/Ubuntu/Windows
# Install binary from the release page
# macOS Homebrew
brew install topgrade
# macOS MacPorts
port install topgrade
# Cargo
cargo install topgrade --locked
```

## Broot

[Broot](https://github.com/Canop/broot) is a alternative with a better user experience, and you can use it to navigate a file structure. It’s fast and respects . You can cd into a directory from the tree view, open sub-directories in a panel, and even preview files. It has excellent keyboard navigation as well. It has many more features.`tree``.gitignore`

![broot](https://i.imgur.com/kAuL7oJ.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
# Arch Linux
yay -S broot
# Fedora/CentOS/Debian/Ubuntu/Windows
# Install binary from release page https://dystroy.org/broot/install/
# macOS Homebrew
brew install broot
# macOS MacPorts
port install broot
# Cargo
cargo install broot --locked
```

## Tokei

[Tokei](https://github.com/XAMPPRocky/tokei) is a nice utility to count lines and stats of code. It is very fast, accurate, and has a nice output. It supports over 150 languages and can output in JSON, YAML, CBOR, and human-readable tables.

![tokei](https://i.imgur.com/DGFI13M.png)

### Installation

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
# Arch Linux
yay -S tokei
# Fedora/CentOS
dnf install tokei
# Debian/Ubuntu
# Install binary from the release page
# macOS Homebrew
brew install tokei
# macOS MacPorts
port install tokei
# Windows Scoop
scoop install tokei
# Cargo
cargo install tokei
```

-   [kdash](https://github.com/kdash-rs/kdash/): A fast and simple dashboard for Kubernetes. Its created by me :)
-   [Zellij](https://github.com/zellij-org/zellij): A feature rich modern terminal multiplexer with batteries included.
-   [Nushell](https://github.com/nushell/nushell): A modern shell written in Rust. Looks quite promising.
-   [xh](https://github.com/ducaale/xh): A HTTPie alternative with better performance.
-   [monolith](https://github.com/y2z/monolith): Convert any webpage into a single HTML file with all assets inlined.
-   [delta](https://github.com/dandavison/delta): A syntax-highlighting pager for git, diff, and grep output.
-   [ripsecrets](https://github.com/sirwart/ripsecrets): Find secret keys in your code before committing them to git.
-   [eva](https://github.com/nerdypepper/eva): A CLI REPL calculator.
-   You can find a list of other Rust CLI tools [here](https://gist.github.com/sts10/daadbc2f403bdffad1b6d33aff016c0a)

___

If you like this article, please leave a like or a comment.

You can follow me on [Twitter](https://twitter.com/deepu105) and [LinkedIn](https://www.linkedin.com/in/deepu05/).