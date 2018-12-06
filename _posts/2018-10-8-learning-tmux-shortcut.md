---
title: Tmux 快捷方式学习
tags: shortcut
---

## tmux 常用命令

```
tmux  运行tmux
tmux ls  显示所有会话
tmux new -s session-name 新建会话
tmux a || tmux a -t session-name  接入一个之前的会话
tmux detach 从会话中断开
tmux kill-session -t session-name 关闭一个会话
```

## 按下 Ctrl-b 后的快捷键

ctrl+b 是tmux的快捷方式的前缀命令，即为激活快捷方式的开关

### 基础
```
?  获取帮助信息
t 在当前窗格显示时间
```

### 会话管理
```
s 列出所有会话
$ 重命名当前的会话
d 断开当前的会话
```

### 窗口管理
```
c 创建一个新窗口
, 重命名当前窗口
w 列出所有窗口
% 水平分割窗口
" 竖直分割窗口
n 选择下一个窗口
p 选择上一个窗口
0~9 选择0~9对应的窗口
```

### 窗格管理
```
% 创建一个水平窗格
" 创建一个竖直窗格
h 将光标移入左侧的窗格*
j 将光标移入下方的窗格*
l 将光标移入右侧的窗格*
k 将光标移入上方的窗格*
q 显示窗格的编号
o 在窗格间切换
} 与下一个窗格交换位置
{ 与上一个窗格交换位置
! 在新窗口中显示当前窗格
x 关闭当前窗格> 要使用带“*”的快捷键需要提前配置，配置方法可以参考上文的“在窗格间移动光标”一节
```