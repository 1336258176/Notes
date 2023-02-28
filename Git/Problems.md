# Problems
***
## Contents
- [Problems](#problems)
  - [Contents](#contents)
  - [前言](#前言)
  - [VSCode git push Timed out](#vscode-git-push-timed-out)
  - [参考资料](#参考资料)

## 前言
本篇主要想记录一下自己所遇到的Git相关问题，并记录一下解决方法和错误原理，旨在拓展学识和帮助他人。

## VSCode git push Timed out
报错信息：
```
# Failed to connect to github.com port 443 after 21074 ms: Timed out
```
主要是网络代理原因，因此我们更改一下git配置中的代理就可以了，将git配置中的代理换成自己的。
- 第一步：取消代理
```
git config --global --unset http.proxy
git config --global --unset https.proxy
```
- 第二步：更新DNS
按 `Win`+`R`，键入 `cmd`，输入下列命令 `ipconfig /flushdns`。
- 第三步：设置代理
将下方的IP地址和端口，自行改为自己IP地址和代理端口。
```
 git config --global http.proxy http://127.0.0.1:7500
 git config --global https.proxy https://127.0.0.1:7500
```

## 参考资料
- [记录一下解决git总是超时的问题](https://juejin.cn/post/7064467287658479623)