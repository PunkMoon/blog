## mac配置git自动补全

### 安装homebew

如果事先没有安装homebrew,执行命令

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 安装 bash-completion

```
brew install bash-completion
```

### 修改 ~/.bash_profile

```
cd ~
nano .bash_profile
```

将下面的语句添加到.bash_profile

```
source ~/.git-completion.bash
if [ -f $(brew --prefix)/etc/bash_completion ]; then
  . $(brew --prefix)/etc/bash_completion
fi
```

保存后重启terminal即可。