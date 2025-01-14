<div align="center">

<img src="assets/logo.svg" alt="ivy.vim" /> 

<br />
<br />

An [ivy-mode](https://github.com/abo-abo/swiper#ivy) port to neovim. Ivy is a
generic completion mechanism for ~~Emacs~~ Nvim

</div>

## Installation

### Manually

```sh
git clone https://github.com/AdeAttwood/ivy.nvim ~/.config/nvim/pack/bundle/start/ivy.nvim
```

### Plugin managers

Using [lazy.nvim](https://github.com/folke/lazy.nvim)
```lua
{
    "AdeAttwood/ivy.nvim",
    build = "cargo build --release",
},
```

TODO: Add more plugin managers

### Compiling

For the native searching, you will need to compile the shard library. You can
do that by running the below command in the root of the plugin.

```sh
cargo build --release
```

You will need to have the rust toolchain installed. You can find more about
that [here](https://www.rust-lang.org/tools/install)

If you get a linker error you may need to install `build-essential` to get
`ld`. This is a common issue if you are running the [benchmarks](#benchmarks)
in a VM

```
error: linker `cc` not found
  |
  = note: No such file or directory (os error 2)
```

To configure auto compiling you can use the [`post-merge`](./post-merge.sample)
git hook to compile the library automatically when you update the package. This
works well if you are managing your plugins via git. For an example
installation you can run the following command. **NOTE:** This will overwrite
any post-merge hooks you have already installed.

```sh
cp ./post-merge.sample ./.git/hooks/post-merge
```

## Features

### Commands

A command can be run that will launch the completion UI

| Command            | Key Map     | Description                                                 |
| ------------------ | ----------- | ----------------------------------------------------------- |
| IvyFd              | \<leader\>p | Find files in your project with a custom rust file finder   |
| IvyAg              | \<leader\>/ | Find content in files using the silver searcher             |
| IvyBuffers         | \<leader\>b | Search though open buffers                                  |
| IvyLines           |             | Search the lines in the current buffer                      |
| IvyWorkspaceSymbol |             | Search for workspace symbols using the lsp workspace/symbol |

### Actions

Action can be run on selected candidates provide functionality

| Action         | Description                                                                    |
| -------------- | ------------------------------------------------------------------------------ |
| Complete       | Run the completion function, usually this will be opening a file               |
| Peek           | Run the completion function on a selection, but don't close the results window |
| Vertical Split | Run the completion function in a new vertical split                            |
| Split          | Run the completion function in a new split                                     |

## API

### ivy.run

The `ivy.run` function is the core function in the plugin, it will launch the
completion window and display the items from your items function. When the
users accept one of the candidates with an [action](#actions), it will call the
callback function to in most cases open the item in the desired location.

```lua
  ---@param name string
  ---@param items fun(input: string): { content: string }[] | string
  ---@param callback fun(result: string, action: string)
  vim.ivy.run = function(name, items, callback) end
```

#### Name `string`

The name is the display name for the command and will be the name of the buffer
in the completion window

#### Items `fun(input: string): { content: string }[] | string`

The items function is a function that will return the candidates to display in
the completion window. This can return a string where each line will be a
completion item. Or an array of tables where the `content` will be the
completion item.

#### Callback `fun(result: string, action: string)`

The function that will run when the user selects a completion item. Generally
this will open the item in the desired location. For example, in the file
finder with will open the file in a new buffer. If the user selects the
vertical split action it will open the buffer in a new `vsplit`

#### Example

```lua
  vim.ivy.run(
    -- The name given to the results window and displayed to the user
    "Title",
    -- Call back function to get all the candidates that will be displayed in
    -- the results window, The `input` will be passed in, so you can filter
    -- your results with the value from the prompt
    function(input)
      return {
        { content = "One" },
        { content = "Two" },
        { content = "Three" },
      }
    end,
    -- Action callback that will be called on the completion or peek actions.
    -- The currently selected item is passed in as the result.
    function(result) vim.cmd("edit " .. result) end
  )
```

## Benchmarks

Benchmarks are of various tasks that ivy will do. The purpose of the benchmarks
are to give us a baseline on where to start when trying to optimize performance
in the matching and sorting, not to put ivy against other tools. When starting
to optimize, you will probably need to get a baseline on your hardware.

There are fixtures provided that will create the directory structure of the
[kubernetes](https://github.com/kubernetes/kubernetes) source code, from
somewhere around commit sha 985c9202ccd250a5fe22c01faf0d8f83d804b9f3. This will
create a directory tree of 23511 files a relative large source tree to get a
good idea of performance. To create the source tree under
`/tmp/ivy-trees/kubernetes` run the following command. This will need to be run
for the benchmarks to run.

```bash
# Create the source trees
bash ./scripts/fixtures.bash

# Run the benchmark script
luajit ./scripts/benchmark.lua
```

Current benchmark status running on a `e2-standard-2` 2 vCPU + 8 GB memory VM
running on GCP.

IvyRs (Lua)

| Name                         | Total         | Average       | Min           | Max           |
| ---------------------------- | ------------- | ------------- | ------------- | ------------- |
| ivy_match(file.lua) 1000000x | 04.153531 (s) | 00.000004 (s) | 00.000003 (s) | 00.002429 (s) |
| ivy_files(kubernetes) 100x   | 03.526795 (s) | 00.035268 (s) | 00.021557 (s) | 00.037127 (s) |

IvyRs (Criterion)

| Name                  | Min       | Mean      | Max       |
| --------------------- | --------- | --------- | --------- |
| ivy_files(kubernetes) | 19.727 ms | 19.784 ms | 19.842 ms |
| ivy_match(file.lua)   | 2.6772 µs | 2.6822 µs | 2.6873 µs |

CPP

| Name                         | Total         | Average       | Min           | Max           |
| ---------------------------- | ------------- | ------------- | ------------- | ------------- |
| ivy_match(file.lua) 1000000x | 01.855197 (s) | 00.000002 (s) | 00.000001 (s) | 00.000177 (s) |
| ivy_files(kubernetes) 100x   | 14.696396 (s) | 00.146964 (s) | 00.056604 (s) | 00.168478 (s) |

## Other stuff you might like

- [ivy-mode](https://github.com/abo-abo/swiper#ivy) - An emacs package that was the inspiration for this nvim plugin
- [Command-T](https://github.com/wincent/command-t) - Vim plugin I used before I started this one
- [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) - Another competition plugin, lots of people are using
