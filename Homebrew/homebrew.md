# Homebrew

>1. [M1配置Homebrew](https://zhuanlan.zhihu.com/p/372576355 "homebrew")

## 常用命令

+ `/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"`	// 安装
+ `brew update`	// 更新brew
+ `brew outdated`	// 查看已安装的需要更新的软件
+ `brew upgrade Package`	// 更新包
+ `brew install Package`	// 安装包
+ `brew uninstll Package`	// 卸载包
+ `brew cleanup (-n -s)`	// 清理所有包的旧版本，-n显示要清除的内容，-s清除缓存
+ `brew list (--version)`	// 查看安装列表
+ `brew info Package`	// 查看包信息
+ `brew search Package`	// 搜索包
+ `brew deps Package`	// 显示包依赖
+ `brew services list`	// 获取service列表
+ `brew services start/stop/restart serverName`	// 服务控制
+ `brew config`	// 查看brew配置
+ `/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"`	// 卸载