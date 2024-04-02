## NeoVim 配置

- lazy.nvim
- nvim-surround

安装在

- win: `C:\Users\USERNAME\AppData\Local\nvim\init.lua`
- unix: `~/.config/nvim/init.lua`

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

local nvim_surround = {
    "kylechui/nvim-surround",
    version = "*", -- Use for stability; omit to use `main` branch for the latest features
    event = "VeryLazy",
    config = function()
        require("nvim-surround").setup({
            -- Configuration here, or leave empty to use defaults
        })
    end
}

require("lazy").setup({
    nvim_surround
})

```

### 注意

1. apt 安装的 neovim 版本过低，需要通过其他方式安装
