# Wechat

使用打包好的 [Deepin Wine Wechat Arch](https://github.com/vufa/deepin-wine-wechat-arch)，仓库中已经给出了详细的安装方法、字体更换等

## Trouble shotting
### 窗口阴影
换到 i3wm 后，混成器使用了 picom，此时使用 Deepin Wine Wechat 会发现整个窗口被灰色遮罩，且有弧形的黑色阴影，关闭 picom 后上述情况消失，因此判断是 picom 的问题

查阅 arch wiki 上 picom 条目发现，picom 可以针对窗口禁用半透明、阴影等特性，其规则可以细化到匹配窗口名称

首先通过 `xprop` 查询微信的窗口名:
```
# xprop
WM_NAME(STRING) = "WeChat"
...
WM_CLASS(STRING) = "wechat.exe", "Wine"
```

然后在 `picom.conf` 里添加如下规则: 
```conf
~/.config/picom/picom.conf
# Specify a list of conditions of windows that should have no shadow.
#
# examples:
#   shadow-exclude = "n:e:Notification";
#
# shadow-exclude = []
shadow-exclude = [
  # ...
  "name = 'WeChat'",
  "class_g = 'wechat.exe'",
  "class_g = 'Wine'"
];
```

重启 picom 即可

### 字体发虚
打开 wine 设置
```
# /opt/apps/com.qq.weixin.deepin/files/run.sh winecfg
```
graphics 选项卡里调高DPI即可

### 部分 Emoji 为方块
尚未解决