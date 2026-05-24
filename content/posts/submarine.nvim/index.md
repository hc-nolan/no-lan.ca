---
date: '2026-05-24T12:32:00-04:00'
draft: false
title: 'submarine.nvim'
---

[Neovim](https://neovim.io/) has been my preferred text editor for a few years now. Last year, I started contributing to various plugins that I had been using. This year, I decided it was time to try out writing a plugin of my own.

Introducing: [submarine.nvim](https://github.com/hc-nolan/submarine.nvim)
<!--more-->

# What's it do?

Submarine provides a deep dive into the Lua modules in your Neovim runtime.

{{ image(src="1.png", alt="submarine.nvim screenshot 1")}}

When you first open the picker, all loaded modules will be displayed - this will include all Neovim defaults, and those imported by any plugins that you use. Above, I am filtering for the submarine module - the plugin itself.

{{ image(src="2.png", alt="submarine.nvim screenshot 2")}}

After selecting the module, you can view the functions exported by that module and their docstrings.

## Why?

The idea for the plugin came one night after I added [Gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim) to my config. I was tabbing through the autocompletions for `:Gitsigns` to see what the plugin could do, and thought about how it would be nice if I could see the LSP definitions for the available functions so I can see what they do and what arguments they require, instead of having to open the help pages or the repo itself.

Additionally, I can just browse through the modules loaded and look for things I might find useful. It's a bit more interactive than reading the docs front to back.

Furthermore, pickers like [Snacks](https://github.com/folke/snacks.nvim) and [Telescope](https://github.com/nvim-telescope/telescope.nvim) have builtin functions for searching through help files and commands, but from what I could find, no other Neovim plugin exists to look through all the functions you have available to use.

The name *submarine* came from the idea of a periscope poking out of the water, resembling how the plugin provides a view into the Neovim runtime that you'd normally have to go elsewhere to see.

# Building it

I rely on LSP definitions pretty much every time I code. For the uninitiated, pressing `<S-k>` on a symbol will pull up the definition of that symbol.

{{ image(src="3.png", alt="LSP definition screenshot")}}

I knew that all I really wanted to do was replicate this functionality, but not require the user to have a buffer open with their cursor over the function they want to inspect. Therefore, the first thing I did was find out what is actually happening when you press `<S-k>`, so I checked what the keybind actually does.

{{ image(src="5.png", alt="Shift-K mapping")}}

Then I looked at what that function actually does, and saw that it sends a `textDocument/hover` request to the LSP.

```lua
--- @param config? vim.lsp.buf.hover.Opts
function M.hover(config)
  validate('config', config, 'table', true)

  config = config or {}
  config.focus_id = 'textDocument/hover'

  lsp.buf_request_all(0, 'textDocument/hover', client_positional_params(), function(results, ctx)
    local bufnr = ctx.bufnr
    if not bufnr or not ctx_is_valid(ctx) then
      return -- Ignore result if context changed. Can happen for slow LS.
    end
  ...

```

At first I thought I might be able to simply pass a module and function name to the LSP and have it return the definition, but the [Language Server Protocol](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#languageServerProtocol) requires a specific position that points to the symbol. I tried a few different things to work around this until I learned about Lua's `debug.getinfo()`:

{{ image(src='./6.png', alt='Screenshot of Lua documentation for debug.getinfo') }}

Part of the info it returns is the file where the function is defined, and the line on which it is defined. So, instead of having to write anything or have our actual cursor anywhere, we can just use the actual file where the function exists already.

My next question was how to determine all functions exported by a module - thich is [easily done by using pairs()](https://stackoverflow.com/a/37310201). Now we have everything for a super basic proof of concept.

Snacks is my picker of choice, so it is the one I implemented into the plugin. For the initial proof of concept I just did something like this:

```lua
local mod = require('debug')
-- get all functions in 'debug'
local fns = {}
for key, value in pairs(mod) do
    if type(value) == "function" then
    local info = debug.getinfo(value, "S")
    fns[key] = { fn = value, info = info }
end

-- get the LSP client; i had manually attached to client 0 at the time
local client = vim.lsp.get_client_by_id(0)

-- get the hover doc for each function
local results = {}
for _, entry in ipairs(fns) do
    local info = entry.info
    local src_path = info.source:sub(2)
    local uri = vim.uri_from_fname(src_path)
    table.sort(entry.names)
    local display_name = table.concat(entry.names, " / ")
    -- send the request to the LSP
    client:request(
        "textDocument/hover",
        -- subtract 1 due to different indexing
        { textDocument = { uri = uri }, position = { line = info.linedefined - 1, character = 0 } },
        function(err, result)
            local docs
            -- do what vim.lsp.buf.hover() does to render the content as markdown
            local lines = vim.lsp.util.convert_input_to_markdown_lines(result.contents)
            results[#results + 1] = { name = display_name, docs = docs }
        end
    )
end

-- put the data in the format needed for Snacks
local items = vim.tbl_map(function(r)
    return { text = r.name, preview = { text = r.docs, ft = "markdown" } }
end, results)

-- open Snacks picker
Snacks.picker.pick({
    title = "debug",
    items = items,
    format = "text",
    preview = "preview"
})
```

This is just a reconstruction of the early code from memory, so it probably doesn't actually work, but it covers the main parts of the overall logic that is still in the code now.

Once I had the initial problem solved, I then just needed to figure out how to get all the modules currently loaded, which turned out to be `package.loaded()`.

{{ image(src='./4.png', alt='Screenshot of Lua documentation for package.loaded') }}

This was the final piece of the puzzle; the rest was just adding a first Snacks picker to select the module, with a callback to open the picker for the functions of that module.

## Why no docs?

I ran into one bug that took a while to figure out after everything was pretty much working and ready.

While developing, I was testing the plugin with Lua files open, so LuaLS was already attached. I noticed that, if the file was open for ~15 seconds or more, everything worked perfectly, but if I tried quicker than that, the hover doc wouldn't load. Then I noticed that, if I opened the plugin from a non-Lua file, it would *always* fail to load any of the docstrings at first, and always require ~15 seconds before it would work properly.

Eventually I realized it was related to the time it took LuaLS to index the runtime workspace, hence why the 15 second timer only started once I initiated the plugin on non-Lua files - LuaLS would only start indexing once it was attached by the plugin. I then learned that the LSP supports a message for [reporting progress](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#progress), that [LuaLS implements it](https://github.com/LuaLS/lua-language-server/blob/master/script/progress.lua), and that I could use a handler in `vim.lsp.start()` to run a callback function to notify the user if the workspace is still being indexed.

{{ image(src='./7.png', alt='Screenshot of workspace indexing notification') }}


<!--more-->

I learned some interesting stuff about how LSPs work with this and I look forward to getting deeper into the Neovim and Lua ecosystems.

