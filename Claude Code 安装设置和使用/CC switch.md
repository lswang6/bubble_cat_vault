
# 目前使用别人的转发的 api 服务器，如果遇到 400 错误，需要在 cc switch 设置中的 env 加入"ENABLE_TOOL_SEARCH": "true"


## antigravity 反代组：
{
  "enabledPlugins": {
    "context7@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true,
    "frontend-design@claude-plugins-official": true,
    "rust-analyzer-lsp@claude-plugins-official": true,
    "skill-creator@claude-plugins-official": true,
    "superpowers@claude-plugins-official": true,
    "vercel@claude-plugins-official": true
  },
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-ZKEKS5ocXsTN37ZoP89CsckelogkPEGGgRxiBNJJicx9uKML",
    "ANTHROPIC_BASE_URL": "https://www.fucheers.top",
    "ENABLE_TOOL_SEARCH": "true"
  },
  "model": "opus[1m]",
  "permissions": {
    "deny": []
  },
  "skipDangerousModePermissionPrompt": true
}

## kiro 反代组：
{
  "enabledPlugins": {
    "context7@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true,
    "frontend-design@claude-plugins-official": true,
    "rust-analyzer-lsp@claude-plugins-official": true,
    "skill-creator@claude-plugins-official": true,
    "superpowers@claude-plugins-official": true,
    "vercel@claude-plugins-official": true
  },
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-4HJ6Omfp4etlD4nSOut7qMtEC7RMoxyLo9WvYVbisn17aXHf",
    "ANTHROPIC_BASE_URL": "https://www.fucheers.top",
    "ENABLE_TOOL_SEARCH": "true"
  },
  "model": "claude-opus-4-6",
  "permissions": {
    "deny": []
  },
  "skipDangerousModePermissionPrompt": true
}