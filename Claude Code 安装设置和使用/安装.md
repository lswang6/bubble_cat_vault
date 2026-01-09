目前在 terminal 里面，通过 homebrew 方式安装


安装后输入 claude 可以启动
但此时提示需要登录

可以先同时按 shift+cmd+. 显示系统隐藏的文件
然后在如下路径新建一个文件
/Users/louiswang/.claude/settings.json

里面放置 api 和端点。目前我用本地的 Antigravity Tools 的：

{
  "env": {
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:8045",
    "ANTHROPIC_AUTH_TOKEN": "sk-e765924f328b4d76b616ad8ad96cffb5"
  },
  "language": "Chinese"
}


如果启动初始化有问题，可以编辑这个问题，
/Users/louiswang/.claude.json

第一行加入：
"hasCompletedOnboarding": true,

但是之后似乎这一行会消失，不过能运行就可以。

