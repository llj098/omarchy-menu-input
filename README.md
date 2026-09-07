# Omarchy Input IME

原版 `Menu.qml` 的小补丁：只让 `omarchy menu input` 使用真正的 Qt `TextInput`，保留原窗口、主题和命令接口。无新增运行依赖或脚本。

代码位于 `input-ime.patch`，目前新增 49 行、替换/删除 4 行。原版源码及本地生成的完整 `Menu.qml` 不纳入版本管理。

## 状态

- 已在 Omarchy 4.0.2 / Quickshell 0.3.1 / Qt 6.11.2 上通过 `qmllint`（原版与补丁版均退出 0；原版含类型信息相关 warnings）。
- 已通过实际 Quickshell 组件加载：`Configuration Loaded`、`MENU_LOAD_OK`，退出 0；测试实例没有打开窗口，结束后退出。
- 已部署到 `192.168.1.149`（`lj-t480s`）的 `~/.config/omarchy/plugins/lj.menu/`，系统原文件校验值不变。
- 真实 Rime 预编辑与候选窗已出现；通过原命令测试“选词不提交 → Enter 提交”，UTF-8 中文返回、退出码 0；空框 Escape 取消返回 1。
- 注意：本机仅 rescan 未能更新已缓存的 QML，重启一次 Omarchy Shell 后生效。未完成跨主题截图对比。

## 应用

先解锁桌面。使用目标机的原版代码克隆；不要复制别的 Omarchy 版本覆盖系统文件。

```bash
omarchy plugin clone omarchy.menu
plugin="$HOME/.config/omarchy/plugins/$(id -un).menu"
patch --dry-run --fuzz=0 -d "$plugin" -p1 < input-ime.patch
patch --fuzz=0 -d "$plugin" -p1 < input-ime.patch
omarchy-shell shell rescanPlugins
# 若仍显示旧版本，在桌面未锁定时执行：
omarchy restart shell
```

克隆命令本身会启用个人菜单；如已有个人克隆，不重复执行或覆盖。补丁检查失败则停止，并用下面的回退命令恢复原菜单。

```bash
omarchy plugin enable omarchy.menu
```

调用方式不变：

```bash
text=$(omarchy menu input "Ask Pi" --width 650) || exit
```

测试基线 `/usr/share/omarchy/shell/plugins/menu/Menu.qml` SHA-256：
`0154d0ec3855fbb48aa06eaa724a19f3c07c377494d4a3de65daed2d7d9100e3`。

个人克隆继续共享宿主主题组件，但系统升级不会自动合并 Menu.qml 的更新；更新基线后重新应用小补丁即可。
