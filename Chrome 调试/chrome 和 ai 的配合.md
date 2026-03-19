确认了，你有 **2 个独立的 Chrome 实例**在运行：

|实例|User-Data-Dir|说明|
|---|---|---|
|**1. Antigravity 调试浏览器**|`/Users/louiswang/.gemini/antigravity-browser-profile`|MCP 连接的就是这个|
|**2. 普通 Chrome**|默认 profile（没有指定 user-data-dir）|你平时用的 Chrome|

它们可以同时运行是因为使用了**不同的 user-data-dir**。

**现状**：

- MCP 连接的是 Antigravity 的调试浏览器（实例 1）
- 普通 Chrome（实例 2）不受 MCP 控制，但下载功能正常

这就是 Chrome 可以同时运行两个实例的原因 - 它们是完全隔离的。

你可以通过 chrome devtool 连接平时用 chrome 吗？

Thought for 3s

**不能**，除非普通 Chrome 也启用了远程调试端口。

**原因**：

- 普通 Chrome 没有带 `--remote-debugging-port` 参数启动
- MCP 只能连接到监听 9222 端口的浏览器
- 目前只有 Antigravity 的调试浏览器在监听这个端口

**理论上可以的做法**（但复杂）：

1. 关闭普通 Chrome 的所有窗口
2. 杀掉普通 Chrome 的所有后台进程
3. 用调试参数重新启动普通 Chrome：
    
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9223
    
    （用 9223 避免和 Antigravity 的 9222 冲突）
4. 修改 MCP 服务器配置连接到 9223（这个我没权限做）

**结论**：MCP 的端口配置是 Antigravity 工具固定的（9222），我无法修改。所以只能连接 Antigravity 自己的调试浏览器。

如果你需要在普通 Chrome 调试，只能手动用 `chrome://inspect` 或开发者工具。