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




### nvim-surround


```bash
ys{motion}{char}        # add
ds{char}                # delete
cs{target}{replacement} # change
```


## NeoVim


保留 vscode 快捷键

|        | vim                  | vscode          |     |
| ------ | -------------------- | --------------- | --- |
| Ctrl+b | 向上滚动一屏         | 打开/收起侧边连 |     |
| Ctrl+f | 向下滚动一屏         | 搜索            |     |
| Ctrl+w | 删除光标前的一个单词 | 关闭标签页      |     |



切换 Explorer 和 标签页: `ctrl+0` `ctrl+1`


## 在 Explorer 窗口中使用类似于 vim 的快捷键

```bash
j/k         # 向下/上滚动
zc/zm       # 折叠当前/全部
/           # 搜索

r           # 重命名
x           # 剪切
y           # 复制
p           # 粘贴
d           # 删除
a/A         # 创建文件/文件夹
```