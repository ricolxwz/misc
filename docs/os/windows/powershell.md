---
title: Powershell
layout: doc
navbar: true
sidebar: true
aside: true
outline: deep
lastUpdated: true
editLink: true
footer: true
---

# Powershell

## 常用命令

- `set-location`: 改变当前目录 
- `get-location`: 打印当前目录路径
- `get-childitem`: 打印当前目录下的文件/目录
- `write-host <param>`: 打印变量的值
- `<param> = read-host -Prompt <prompt>`: 读取终端输入
- `$param = @[]`: 列表
- `$param = @{}`: 字典
- `new-item -type file`: 创建新文件
- `new-item -type directory`: 创建新目录
- `copy-item <原文件> -destination <目标文件>`: 复制文件
- `move-item <原文件> -destination <目标文件>`: 移动文件
- `remove-item <文件>`: 移除文件
- `test-path <path>`: 测试path是否存在
- `rename-item <文件> -newname <新名字>`: 重命名文件
- `get-content`: 打印文件内容
- `select-string`: 搜索文本, 类似于grep
- `get-process`: 显示进程
- `invoke-restmethod`: 网络请求, 类似于curl或者wget
- `get-command`: 查找命令
- `get-volume`: 查看卷的信息
- `stop-process -id <id>`: 终止某个进程
- `get-service`: 列出所有服务
- `start-service -name <name>`: 启动某个服务
- `stop-service -name <name>`: 终止某个服务
- `write-output`: 适用于数据处理和传递, 如果你的脚本需要将数据作为对象传递给其他命令或者进一步处理, 应该使用这个. 它会把输出写入管道, 确保数据的可扩展性和灵活. 这是Powershell的默认输出方式, 直接写变量也是会自动输出的, 如`$result = 42; $result`等价于`$result = 42; write-output $result`
- `out-file -filepath <path>`: 输出字符串到文件, 也可以使用重定向符号, 如`get-process > process.txt`, 但是可能参数没有`out-file`丰富, `out-file -filepath .\test.txt -inputobject "test"`
- `select-object`: 选择特定的属性

## 管道

https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.4

## 条件

```powershell
$PokemonCaught = "911"
if ($PokemonCaught -eq 908) { write-host "You are a pokemon master!" } elseif ($PokemonCaught -ge 910) { write-host "what?" } else { write-host "Go catch more pokemon!" }
```

## 循环

```powershell
$halopeeps = @("Master Chief", "Cortana", "Captain Keyes", "Flood")
for ($counter = 0; $counter -le 3; $counter ++) {
    write-host "Holy smokes, it's" $halopeeps[$counter]
}
```

```powershell
$halopeeps = @("Master Chief", "Cortana", "Captain Keyes", "Flood")
foreach ($peep in $halopeeps) {
    write-host $peep
}
```

```powershell
$Xmen = @('Wolverine', 'Cyclops', 'Storm', 'Professor X')
$counter = 0
while ($counter -ne 6) {
    write-host $Xmen[$counter]
    $counter ++
}
```

```powershell
$Xmen = @('Wolverine', 'Cyclops', 'Storm', 'Professor X')
$counter = 0
do {
    write-host "Xmen[$counter]"
    $counter++
} while ($counter -ne 6)
```

## 函数

```powershell
function test-spacex {
    param (
        $pingcount
    )
    test-connection spacex.com -Count $pingcount
}
test-spacex -pingcount 10
```

```powershell
function test-spacex {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [int32] $pingcount
    )
    test-connection spacex.com -Count $pingcount
}
test-spacex
```

## 错误

```powershell
# throw "it's a trap"
write-error -message "It's a trap!" -erroraction stop
```

```powershell
function test-spacex {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [int32] $pingcount
    )
    test-connection spacex.com -Count $pingcount
    write-error -message "It's a trap!" -erroraction stop
}
try {
    test-spacex -erroraction stop
} catch {
    write-output "launch problem!"
    write-output $_
}
```

## 执行

- `.`: 将脚本中的变量, 函数导入当前会话, 脚本执行完后仍然可以访问这些内容
- `&`: 在新作用域中执行脚本, 脚本执行后所有内容都会被销毁, 不影响当前会话

## 作用域

AllUsersAllHosts: 针对所有用户和所有PowerShell主机
AllUsersCurrentHost: 针对所有用户但仅在当前PowerShell主机中执行
CurrentUserAllHosts: 针对当前用户的所有PowerShell主机
CurrentUserCurrentHost: 针对当前用户但仅在当前PowerShell主机中执行

AllHosts: 对所有的Powershell实例都生效, 如Powershell控制台, Powershell ISE, VSCode中的Powershell Extension. 

## 环境变量

可以使用`$env:`查看当前所有可用的环境变量.

```powershell
get-childitem env:
```

可以通过`$env:`变量前缀来访问特定的环境变量, 如访问`PATH`环境变量.

```powershell
$env:PATH
```

## 权限

### 执行策略

Powershell的执行策略决定了脚本是否可以在系统上运行, 常见的执行策略有:

- Restricted: 禁止执行任何Powershell脚本(默认)
- AllSigned: 仅允许由受信任发布者签名的脚本
- RemoteSigned: 本地脚本可以运行, 但是从网络下载的脚本必须经过数字签名
- Unrestrited: 允许执行所有的脚本, 下载的脚本在第一次运行的时候会提示警告
- Bypass: 没有任何限制, 允许所有脚本运行

```powershell
get-executionpolicy
set-executionpolicy remotesigned
```

### 文件和目录

`icacls`是windows上传统的cmd的命令行权限设置工具, `get-acl`, `set-acl`是powershell提供的命令行权限设置工具, 我们先来看`icacls`, 然后再看powershell的实现.

#### `icacls`

权限代码: 

- F: 完全控制
- M: 修改
- RX: 读取和执行 
- R: 读取
- W: 写入

权限控制:

查看权限: `ICACLS "C:\Path\To\FileOrFolder"`
授予权限: `ICACLS "C:\Path\To\FileOrFolder" /grant Users:(RX)`
移除权限: `ICACLS "C:\Path\To\FileOrFolder" /remove Users`
设置所有者: `ICACLS "C:\Path\To\FileOrFolder" /setowner Administrator`
递归设置权限: `ICACLS "C:\Path\To\Directory" /grant Users:(R) /T`

其中:

- `/grant`: 在现有的基础上授予新的权限, 不会替换旧的权限
- `/grant:r`: 替换用户或者组的现有权限, 并授予指定的新权限

#### Powershell实现

Powershell使用的是ACL进行权限控制. 首先, 获取文件或者目录的ACL. 注意, 虽然Powershell Core支持跨平台运行, 但是Linux上的Powershell中`set-acl`和`get-acl`是无法使用的, 可以直接使用`chmod`, `chown`

```powershell
get-acl <文件>
```

```powershell
$acl = Get-Acl "D:\test\test.txt"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("Users", "ReadAndExecute", "Allow")
$acl.SetAccessRule($rule)
Set-Acl "D:\test\test.txt" $acl
```