## 1. 临时禁用清华源并使用官方 PyPI 源

如果你想临时禁用清华源并使用官方 PyPI 源，可以使用 `--index-url` 参数来指定使用 `https://pypi.org/simple`。

### 命令：

```bash
pip install <package_name> --index-url https://pypi.org/simple
```
例如，安装 `ida-pro-mcp`：
```bash
pip install git+https://github.com/mrexodia/ida-pro-mcp --index-url https://pypi.org/simple
```
## 2. 设置本地 HTTP 代理

如果你需要通过本地 HTTP 代理（例如代理地址为 `127.0.0.1:7897`）来连接网络，可以通过设置环境变量 `http_proxy` 和 `https_proxy` 来指定代理。

### 命令：
```bash
set http_proxy=http://127.0.0.1:7897
set https_proxy=http://127.0.0.1:7897
```

这将会在当前命令行会话中设置代理。

## 3. 验证代理设置是否生效

你可以使用 `curl` 或 `pip` 命令来验证代理是否工作正常。比如，通过 `curl` 访问 GitHub 来检查代理是否设置成功：
```bash
curl -I https://www.github.com
```
如果返回 GitHub 的响应头，说明代理设置有效。如果未能成功连接，说明代理设置存在问题。

## 4. 临时禁用代理

如果你不再需要代理或想要禁用代理，可以通过以下命令清除代理设置：
```bash
set http_proxy=
set https_proxy=
```
这将清除当前命令行会话中的代理设置，恢复到没有代理的状态。

## 5. 注意事项

- **临时代理**：`set` 命令仅在当前命令行会话中有效，关闭命令行窗口后，代理设置会失效。
    
- **清华源禁用**：通过 `--index-url` 参数临时禁用清华源，仅影响当前安装命令的执行，不会影响其他命令或会话。
    
- **持久化设置**：如果你希望每次打开命令行时自动设置代理，可以将代理设置添加到系统环境变量中，或使用批处理脚本自动执行代理设置。
```markdown

### 说明：
- 这份总结覆盖了如何临时禁用清华源并使用本地 HTTP 代理，适用于临时解决网络问题或代理配置问题。
- 代理设置是针对当前命令行会话的，关闭命令行后设置会失效。

```
