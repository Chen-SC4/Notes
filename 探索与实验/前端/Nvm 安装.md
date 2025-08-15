###### 1. 卸载电脑上已经安装的 Node

可以通过控制面板卸载已安装的 Node，卸载的时候记一下 Node 版本，方便之后重新安装。

###### 2. 安装 Nvm

在 Github 上安装 Nvm，Windows 和 Linux 两个平台的 Nvm 是两个不同的项目，不要弄错了。

###### 3. 安装之前卸载掉的 Node

```powershell
nvm install 16.20.2
```

- `16.20.2` 是之前 Node 的版本号

###### 4. 安装最新版本的 Node

```powershell
nvm install lts
```

###### 5. 使用某个特定的 Node 版本

```powershell
nvm use 16.20.2
```

###### 6. 调整 Windows 上的脚本执行策略

在 `Powershell` 中执行 `Get-ExecutionPolicy` 命令，如果结果是 `Restricted` 则说明本机上不允许脚本的执行。

此时，我们需要退出，然后是用管理员身份重新打开终端，执行 `Set-ExecutionPolicy RemoteSigned` 来修改脚本的执行策略。修改完成之后，退出再重新打开一个普通终端即可正常运行脚本。