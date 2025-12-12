# Vim 快捷键 Vim Shortcuts

## 模式切换 Mode Switching
- `i`: 进入插入模式（在光标前）
- `a`: 进入插入模式（在光标后）
- `o`: 在下方新建一行并进入插入模式
- `O`: 在上方新建一行并进入插入模式
- `Esc` 或 `Ctrl + [`: 返回普通模式
- `v`: 进入可视模式（字符选择）
- `V`: 进入可视模式（行选择）
- `Ctrl + v`: 进入可视块模式

## 移动光标 Cursor Movement

### 基础移动
- `h`, `j`, `k`, `l`: 左、下、上、右
- `w`: 移动到下一个单词开头
- `b`: 移动到上一个单词开头
- `e`: 移动到单词结尾
- `0`: 移动到行首
- `^`: 移动到行首（非空白字符）
- `$`: 移动到行尾

### 快速移动
- `gg`: 移动到文件开头
- `G`: 移动到文件结尾
- `数字 + G`: 移动到指定行（如 `10G` 移动到第10行）
- `Ctrl + f`: 向下翻页
- `Ctrl + b`: 向上翻页
- `Ctrl + d`: 向下半页
- `Ctrl + u`: 向上半页

## 编辑操作 Editing Operations

### 删除
- `x`: 删除光标处的字符
- `dd`: 删除整行
- `dw`: 删除到下一个单词开头
- `d$` 或 `D`: 删除到行尾
- `d0`: 删除到行首

### 复制和粘贴
- `yy`: 复制整行
- `yw`: 复制一个单词
- `y$`: 复制到行尾
- `p`: 在光标后粘贴
- `P`: 在光标前粘贴

### 撤销和重做
- `u`: 撤销
- `Ctrl + r`: 重做

### 查找和替换
- `/pattern`: 向下查找
- `?pattern`: 向上查找
- `n`: 下一个匹配
- `N`: 上一个匹配
- `:s/old/new/`: 替换当前行第一个匹配
- `:s/old/new/g`: 替换当前行所有匹配
- `:%s/old/new/g`: 替换全文所有匹配
- `:%s/old/new/gc`: 替换全文所有匹配（需确认）

## 多文件操作 Multiple Files

### 文件操作
- `:e filename`: 打开文件
- `:w`: 保存
- `:wq` 或 `:x`: 保存并退出
- `:q!`: 不保存退出

### 窗口分割
- `:split` 或 `:sp`: 水平分割窗口
- `:vsplit` 或 `:vs`: 垂直分割窗口
- `Ctrl + w + w`: 切换窗口
- `Ctrl + w + h/j/k/l`: 移动到左/下/上/右窗口

### 标签页
- `:tabnew`: 新建标签页
- `:tabnext` 或 `gt`: 下一个标签页
- `:tabprev` 或 `gT`: 上一个标签页

## 实用技巧 Useful Tips

### 重复命令
- `.`: 重复上一次操作

### 组合命令
- `ci"`: 修改双引号内的内容
- `ci(`: 修改括号内的内容
- `diw`: 删除当前单词
- `dap`: 删除当前段落

### 宏录制
- `q + 字母`: 开始录制宏（如 `qa` 录制到寄存器a）
- `q`: 停止录制
- `@ + 字母`: 执行宏（如 `@a` 执行寄存器a的宏）
- `@@`: 重复执行上次的宏

## 配置 Configuration

创建 `~/.vimrc` 文件进行个性化配置：
```vim
set number          " 显示行号
set relativenumber  " 显示相对行号
set autoindent      " 自动缩进
set tabstop=4       " Tab 宽度
set shiftwidth=4    " 缩进宽度
set expandtab       " 使用空格代替 Tab
syntax on           " 语法高亮
set hlsearch        " 高亮搜索结果
set incsearch       " 增量搜索
```

## 标签 Tags
`vim` `editor` `shortcuts` `命令行`
