title: Setting up my dev environment on a Mac
___

I'm planning to do some hacking on
[`stdexec`](https://github.com/NVIDIA/stdexec), NVIDIA's reference
implementation of [P2300](https://wg21.link/p2300), the proposal that added
structured concurrency facilities in the
[`std::execution`](https://eel.is/c++draft/exec) namespace to C++26. For many,
many years, I've done nearly all of my hacking on my corporate laptop, or a
Linux VM in my corporate "cloud", where a team of engineers keeps my dev
environment working; this project will be on personal time and on a personal
machine, though, so I need to set up a dev environment by myself for the first
time in a long time. This post is an attempt to document my setup so there's a
chance I can recreate it if necessary.

## Things to install
 * [Rectangle](#rectangle)
 * [iTerm2](#iterm2)
 * [Homebrew](#homebrew)
    * [bash](#bash)
    * [cmake](#cmake)
    * [cmake-docs](#cmake-docs)
    * [llvm](#llvm)
    * [neovim](#neovim)
    * [node](#node)
    * [tmux](#tmux)
    * [tree-sitter](#tree-sitter)
    * [tree-sitter-cli](#tree-sitter-cli)
 * [Nerd Fonts](#nerd-fonts)
 * [Configuring iTerm2](#configuring-iterm2)
 * [Configuring `git`](#configuring-codegitcode)
 * [Configuring neovim](#configuring-neovim)
    * [lazy.nvim](#lazynvim)
    * [nvim-treesitter](#nvim-treesitter)
    * [catppuccin/nvim](#catppuccinnvim)
    * [Other tweaks](#other-tweaks)

## Rectangle
[Rectangle](https://rectangleapp.com/) is a window-management app for macOS. I
use Ctrl + Opt + Left/Right to snap windows to the left/right halves of my
monitor so often that I had to look up the keyboard shortcut to type it
out—normally my fingers just invoke the command without conscious thought.
Installing Rectangle is easy: download the `.dmg` file and drag the application
into the Applications folder.

## iTerm2
I use [iTerm2](https://iterm2.com/) at work and I love it. I particularly like
its [shell
integration](https://iterm2.com/documentation-shell-integration.html) and [tmux
integration](https://iterm2.com/documentation-tmux-integration.html).

### Installation
Installing iTerm2 is a matter of downloading the latest stable from [the
downloads page](https://iterm2.com/downloads.html), extracting the application
from the `.zip` file, and dragging the result into the Applications folder.

### Configuration
My preferred configuration is to always start iTerm2 with a running `tmux`
instance so that each iTerm2 window is a `tmux` virtual window. Getting that
fully configured requires [installing Homebrew](#homebrew) and [`tmux`](#tmux)
first, though, so I bootstrapped with the default profile that just launches a
login shell for each new window.

#### Change shells
The first step after starting iTerm2 is to change my shell from `zsh` to `bash`:
```sh
$ chsh -s /bin/bash
```
then restart. Apple, in its infinite wisdom, will now annoy you with the
following message on every login:
```
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
```
If you follow [that link](https://support.apple.com/kb/HT208050), you'll learn
that you can suppress the annoying prompt with the following lines in
`~/.bash_profile`:
```sh
# tell macOS not to whine about using Bash instead of zsh
export BASH_SILENCE_DEPRECATION_WARNING=1
```

#### iTerm2 shell integration
[The shell integration
docs](https://iterm2.com/documentation-shell-integration.html) suggest a `curl
| bash` security hole, which I didn't use; I _think_ I clicked "Install Shell
Integration" in the iTerm2 menu and followed the steps. Shell integration
provides another feature like Rectangle's snap-to-left/right that my fingers
just know: Cmd + Shift + Up/Down scrolls the terminal back and forth between
prompts. Once this step is complete, my `.bash_profile` ends with the following:
```sh
test -e ~/.iterm2_shell_integration.bash && source ~/.iterm2_shell_integration.bash || true
```
In preparation for configuring iTerm2's `tmux` integration, I inserted the
following just before the foregoing in `.bash_profile`:
```sh
# tell iTerm2 to do shell integration inside tmux
export ITERM_ENABLE_SHELL_INTEGRATION_WITH_TMUX=YES
```

## Homebrew
[Homebrew](https://brew.sh/) describes itself as "The Missing Package Manager
for macOS (or Linux)" and it's the easiest way I've found to install open
source developer tools like [neovim](#neovim) or [Clang](#llvm).

Rather than follow the default `curl | bash` [install
instructions](https://brew.sh/#install), I prefer to download the latest `.pkg`
file from [the releases page](https://github.com/Homebrew/brew/releases) and
install that way.

I _think_ I was prompted during Homebrew installation to install the Xcode
command-line tools with the following:
```sh
$ xcode-select --install
```
but I don't clearly remember.

At some point after installing Homebrew, I updated my `.bash_profile` to include
the following command:
```sh
# update PATH to put Homebrew first
eval "$(/opt/homebrew/bin/brew shellenv)"
```
which prepends `$PATH` with `/opt/homebrew/bin:/opt/homebrew/sbin` to
prioritize Homebrew-installed binaries.

### bash
macOS ships with an ancient version of
[Bash](https://www.gnu.org/software/bash/). For a better shell experience,
install the latest Bash with Homebrew:
```sh
$ brew install bash
```
Once that's done, append `/opt/homebrew/bin/bash` to `/etc/shells` to authorize
modern Bash as a valid shell; my `/etc/shells` looks like this:
```sh
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
/opt/homebrew/bin/bash
```
With modern Bash authorized, change shells again:
```sh
$ chsh -s /opt/homebrew/bin/bash
```
and restart iTerm2.

### cmake
stdexec relies on [CMake](https://cmake.org/) for build configuration so we
need to install it:
```sh
$ brew install cmake
```

### cmake-docs
After installing `cmake`, I was prompted (I think in the install logs) to
install [the CMake docs](https://cmake.org/cmake/help/latest/), too, with
```sh
$ brew install cmake-docs
```
Afterwards, I was pleasantly surprised to discover that I'd installed `man`
pages for CMake.

### llvm
I'm most familiar with [Clang](https://llvm.org/), which can be installed via
Homebrew with the following:
```sh
$ brew install llvm
```
The logs will prompt you thus:
```
CLANG_CONFIG_FILE_SYSTEM_DIR: /opt/homebrew/etc/clang
CLANG_CONFIG_FILE_USER_DIR:   ~/.config/clang

LLD is now provided in a separate formula: brew install lld

Using `clang`, `clang++`, etc., requires a CLT installation at
`/Library/Developer/CommandLineTools`.  If you don't want to install the CLT,
you can write appropriate configuration files pointing to your SDK at
~/.config/clang.

To use the bundled libunwind please use the following LDFLAGS:
LDFLAGS="-L/opt/homebrew/opt/llvm/lib/unwind -lunwind"

To use the bundled libc++ please use the following LDFLAGS:
LDFLAGS="-L/opt/homebrew/opt/llvm/lib/c++
-L/opt/homebrew/opt/llvm/lib/unwind-lunwind"

NOTE: You probably want to use the libunwind and libc++ provided by macOS unless
you know what you're doing.

llvm is keg-only, which means it was not symlinked into /opt/homebrew, because
macOS already provides this software and installing another version in parallel
can cause all kinds of trouble.

If you need to have llvm first in your PATH, run: echo 'export
PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> /Users/ianpetersen/.bash_profile

For compilers to find llvm you may need to set: export
LDFLAGS="-L/opt/homebrew/opt/llvm/lib" export
CPPFLAGS="-I/opt/homebrew/opt/llvm/include"

For cmake to find llvm you may need to set: export
CMAKE_PREFIX_PATH="/opt/homebrew/opt/llvm"
```

I ended up discovering that I want Clang early in my path and I want it to use
the bundled `libunwind` and `libc++` as suggested so I updated my
`.bash_profile` with the following:
```sh
# put the homebrew-installed Clang before Xcode's Clang in PATH
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```
and:
```sh
# make sure CMake, etc. finds the right Clang
export LDFLAGS="-L/opt/homebrew/opt/llvm/lib" export
CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
```

### neovim
I'm used to editing all sorts of text, code included in
[`vim`](https://www.vim.org/), but [neovim](https://neovim.io/) seems to be all
the rage so I thought I'd give it a try. Installing is easy:
```sh
$ brew install neovim
```
_Configuring_ neovim is a whole separate process, which I've documented
[below](#configuring-neovim).

### node
[Node](https://nodejs.org/) was a surprising dependency, but it showed up at
some point as an optional install to support tree-sitter somehow. I don't
understand this in detail, but it's easy:
```sh
$ brew install node
```

### tmux
[tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer that also
supports persistent shells that you can detach from and then reattach to later.
I find it most useful when working on a remote host; I can leave a `tmux` server
running with my current state persisted while I'm not connected. Like other
Homebrew installs, the command is simple:
```sh
$ brew install tmux
```

My `~/.tmux.conf` contains the following:
```
set-option -g allow-passthrough on
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
```
I don't remember where all of that came from. I think the `allow-passthrough`
setting is there to make iTerm2's `tmux` integration work better, and the
terminal-related settings are leftover from my thrashing about while trying to
make _italics_ work.

### tree-sitter
I installed [tree-sitter](https://tree-sitter.github.io/) after discovering it'd
be necessary (I think?) for using tree-sitter for syntax-highlighting in neovim.
```sh
$ brew install tree-sitter
```

### tree-sitter-cli
Similar to [tree-sitter](#tree-sitter), I installed
[tree-sitter-cli](https://tree-sitter.github.io/) to satisfy some dependency of
using tree-sitter in neovim.
```sh
$ brew install tree-sitter-cli
```

## Nerd Fonts
While learning how to configure syntax highlighting for neovim, one of the
Google results I found recommended installing a [Nerd
Font](https://www.nerdfonts.com/). The [downloads
page](https://www.nerdfonts.com/font-downloads) has a long list of fonts with
previews and links to a `.zip` file for each font. While browsing the list, I
happened upon the names "FiraCode Nerd Font" and "FiraMono Nerd Font", which
reminded me that I'd previously explored
[FiraCode](https://github.com/tonsky/FiraCode) at work. My favourite feature is
programmer-centric ligatures that turn common character sequences into "nicer
looking", more mathematical symbols, like replacing `!=` with `≠`.

After downloading a `.zip` file containing my new font, I unzipped it and then
launched the built-in app "Font Book", clicked "Add Fonts to Current User…" in
the "File" menu, and selected the directory containing the newly-unzipped
TrueType Font files. Having done so, "FiraCode Nerd Font Mono" and friends are
now options in font selectors.

## Configuring iTerm2
The [tmux integration best
practices](https://gitlab.com/gnachman/iterm2/-/wikis/tmux-Integration-Best-Practices)
turned out to be very useful. In summary:
 * Update the Default profile to run a custom command rather than a login shell:
    ```sh
    /usr/bin/ssh-agent /opt/homebrew/bin/tmux -CC new -A -s main
    ```
    Running `ssh-agent` first means that all the child processes (which includes
    every Bash process I subsequently launch) will inherit the environment
    variables necessary to make `ssh` easy to use (I can add keys with `ssh-add`
    from any shell and benefit from the new authority in any shell).

    Running `tmux -CC new -A -s main` next will ensure a `tmux` server is
    running a session named "main" and connect to it; the `-CC` runs `tmux` in a
    mode that lets iTerm2 render `tmux` virtual windows as new iTerm2 windows.
 * Ensure under the profile's Text configuration, "Allow italic text" is
   selected.
 * Choose "FiraCode Nerd Font Mono" as the terminal font and enable "Use
   ligatures".
 * Enable "Unlimited scrollback" under the profile's Terminal configuration.
 * Go to the "Advanced" top-level configuration tab and, in the _lower_ of the
   two visible "Search" text boxes, type "italic" and look for the advanced
   setting named "Convert italics to reverse video in tmux integration?" and
   choose "No". Without this, `tmux` in `-CC` mode will render italic font in
   reverse video upright font instead of italics, which completely ruins syntax
   highlighting configurations that render things like code comments in italics.

## Configuring `git`
Git needs several configuration changes to be usable. First, set the user name
and email address with the following:
```sh
$ git config --global --edit
```
That will drop you into an editor with a default-configured file that has
comments suggesting how to set things up correctly.

Second, GitHub won't allow password-based authentication so `ssh`-mediated
connections have to be configured with crypto keys. [The GitHub docs for
generating new
keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
work great. I generated a special set of keys just for accessing GitHub into a
pair of files named `~/.ssh/github` and `~/.ssh/github.pub`. I'm terrible about
remembering passwords so I didn't set one; I figure I'll rely on physical
security of my computer and the ability to revoke access from within GitHub to
ensure that's safe.

I also took two suggestions in the docs that, together, ensure that `ssh-agent`
always knows my GitHub keys. I'm not sure which of the two steps is required
(could be either or both), but the first is adding the following to
`~/.ssh/config`:
```
Host github.com
  AddKeysToAgent yes
  IdentityFile ~/.ssh/github
```
and the second is running this command:
```sh
$ ssh-add --apple-use-keychain ~/.ssh/github
```
Since I've configured iTerm2 to run `ssh-agent` as my shell's top-level process,
and `ssh-agent` always knows my GitHub keys, _every_ shell I run knows how to
talk to GitHub—very convenient!

Having created new keys for a dev environment, I have to tell GitHub about them;
that's done by browsing to <https://github.com/settings/keys> while logged in
and pasting the contents of `~/.ssh/github.pub` into the text box you're
presented when you click "New SSH key".

## Configuring neovim

I have been lugging a janky `vim` configuration from corporate laptop to
corporate laptop for _years_ now and was not looking forward to transferring it
to my personal computer. I mostly like bare-bones configurations so that I can
be productive in unconfigured `vim`s, but I do like some creature comforts, like
syntax highlighting, line numbers, and a few other things. After Googling about,
I re-discovered "[tree-sitter](#nvim-treesitter)" and decided to give it a try.

I found the [nvim-treesitter
quick-start](https://github.com/nvim-treesitter/nvim-treesitter#quickstart)
confusing, until I somehow found [this
issue](https://github.com/nvim-treesitter/nvim-treesitter/issues/5519) where
[Paul Evans](https://github.com/leonerd) explains how to get started from actual
zero.

### lazy.nvim
It turns out, the first step in configuring neovim to do anything fancy is to
choose a plug-in manager. I don't know how to choose so I went with
[lazy.nvim](https://github.com/folke/lazy.nvim), which was offered as a kind-of
default in a few places.

The first step, from Paul Evans' explanation, is to stick the following in
`~/.config/nvim/init.lua` and then launch nvim:

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

As far as I can tell, this stanza of Lua code instructs neovim to update the
lazy.nvim plug-in manager on boot if it's out of date.

### nvim-treesitter
With a plug-in manager selected and installed, the next step is to ask it to
install [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)
for you. nvim-treesitter is, as far as I can tell, a neovim plug-in that exposes
[tree-sitter](https://tree-sitter.github.io/tree-sitter/) infrastructure to
neovim for several different purposes. I want it for syntax highlighting, but
it's apparently possible to do all sorts of things with it, like
language-sensitive code folding, and more.

To get nvim-treesitter running with lazy.nvim as the plug-in manager, continue
to follow Paul Evans' instructions and add the following to
`~/.config/nvim/init.lua`:
```lua
require("lazy").setup(
  {
    { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },
  }
)

require'nvim-treesitter.configs'.setup {
  -- A list of parser names, or "all" (the listed parsers MUST always be installed)
  ensure_installed = { "cpp", "lua", "vim", "vimdoc", "query", "markdown", "markdown_inline" },

  -- Install parsers synchronously (only applied to `ensure_installed`)
  sync_install = false,

  -- Automatically install missing parsers when entering buffer
  -- Recommendation: set to false if you don't have `tree-sitter` CLI installed locally
  auto_install = true,

  -- List of parsers to ignore installing (or "all")
  ignore_install = {
	  --"javascript"
  },

  ---- If you need to change the installation directory of the parsers (see -> Advanced Setup)
  -- parser_install_dir = "/some/path/to/store/parsers", -- Remember to run vim.opt.runtimepath:append("/some/path/to/store/parsers")!

  highlight = {
    enable = true,

    -- NOTE: these are the names of the parsers and not the filetype. (for example if you want to
    -- disable highlighting for the `tex` filetype, you need to include `latex` in this list as this is
    -- the name of the parser)
    -- list of language that will be disabled
    --disable = { "c", "rust" },
    -- Or use a function for more flexibility, e.g. to disable slow treesitter highlight for large files
--    disable = function(lang, buf)
--        local max_filesize = 100 * 1024 -- 100 KB
--        local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(buf))
--        if ok and stats and stats.size > max_filesize then
--            return true
--        end
--    end,

    -- Setting this to true will run `:h syntax` and tree-sitter at the same time.
    -- Set this to `true` if you depend on 'syntax' being enabled (like for indentation).
    -- Using this option may slow down your editor, and you may see some duplicate highlights.
    -- Instead of true it can also be a list of languages
    additional_vim_regex_highlighting = false,
  },
}
```
Note that I'm specifically interested in editing C++ (the parser for which is
named `cpp`). Most of the above configuration came from the nvim-treesitter
quick-start guide, but I commented out a few things, and replaced `c` with `cpp`
because I write C++ but not C.

If I remember correctrly, restarting neovim with the above configuration in
place, and with all the previously-described dependencies installed through
Homebrew, will run a bunch of first-time installation steps. Later, the
configuration can be checked with
```vimscript
:checkhealth nvim-treesitter
```

### catppuccin/nvim
[This Reddit
post](https://www.reddit.com/r/neovim/comments/191f5o0/what_are_the_most_popular_colorschemes_for_neovim/)
seeking to try every neovim colorscheme "one by one" led me to [dotfyle's "Top
Neovim Colorschemes"](https://dotfyle.com/neovim/colorscheme/top), which listed
[nvim-catppuccin](https://dotfyle.com/plugins/catppuccin/nvim) near the top.
[The project's GitHub page](https://github.com/catppuccin/nvim) includes
[straightforward
instructions](https://github.com/catppuccin/nvim?tab=readme-ov-file#installation)
for importing into your configuration with many plug-in managers, including
lazy.nvim, so I gave it a whirl like so:
```lua
require("lazy").setup(
  {
    { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },
    { "catppuccin/nvim", name = "catppuccin", priority = 1000 },
  }
)

…

-- let catpuccin figure out that tree-sitter is installed
require("catppuccin").setup({
    auto_integrations = true,
})

vim.cmd.colorscheme "catppuccin-mocha"
```
I also tried [Folke Lemaitre](https://github.com/folke)'s
[Tokyo Night](https://github.com/folke/tokyonight.nvim) theme, but I found it
too dark for my taste. As you can see from the final line of my `init.lua`, I've
settled on `catppuccin-mocha` for now.

### Other tweaks
Finally, I've learned how to port a couple of `vim` preferences to neovim's Lua
syntax, so I've enabled line numbers and the "ruler" with the following at the
end of `init.lua`:
```lua
vim.cmd.set "number"
vim.cmd.set "ruler"
```

I suspect there's a cleaner way to do that, and I'm fairly certain there are
settings that I like in my corporate configuration that I'm missing, but it's a
good start.

With neovim configured, I've also added the following to `~/.bash_profile` to
default to neovim when I get dumped into an editor:
```sh
# prefer neovim
export EDITOR=/opt/homebrew/bin/nvim
```
We'll see if that's supposed to be fully-qualified.
