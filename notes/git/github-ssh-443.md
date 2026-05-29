# GitHub SSH 走 443 端口

## 背景

本机已将 `~/.ssh/id_ed25519_github` 对应的公钥添加到 GitHub，但执行：

```bash
ssh -T git@github.com
```

出现连接失败。

## 排查结论

问题不在密钥本身，而在默认 SSH 端口 `22` 的网络链路上。

调试日志显示，连接 `github.com:22` 时还没进入公钥认证阶段，就被远端关闭：

```text
kex_exchange_identification: Connection closed by remote host
```

改用 GitHub 提供的 SSH over HTTPS 端口后，显式指定密钥可以认证成功：

```bash
ssh -vT -i ~/.ssh/id_ed25519_github -o IdentitiesOnly=yes -p 443 git@ssh.github.com
```

成功信息：

```text
Hi Provenr! You've successfully authenticated, but GitHub does not provide shell access.
```

这说明 `~/.ssh/id_ed25519_github` 已被 GitHub 接受。

## 两种配置的区别

默认配置通常连接：

```text
github.com:22
```

这是 GitHub 的标准 SSH 端口。如果当前网络、代理、公司或校园网对 22 端口有限制，就可能在认证前失败。

修复后的配置仍然让日常命令使用 `git@github.com`，但实际连接：

```text
ssh.github.com:443
```

`443` 是 HTTPS 常用端口，通常更容易通过网络限制。GitHub 专门提供 `ssh.github.com:443` 作为 SSH 端口受限时的替代入口。

## 推荐配置

在 `~/.ssh/config` 中配置：

```sshconfig
Host github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    UseKeychain yes
    AddKeysToAgent yes
    HostName ssh.github.com
    Port 443
    IdentitiesOnly yes
```

关键字段含义：

- `Host github.com`：命令和 Git remote 仍然可以写 `git@github.com:owner/repo.git`。
- `HostName ssh.github.com`：实际连接 GitHub 的 443 SSH 入口。
- `Port 443`：绕过默认 SSH 22 端口的网络限制。
- `IdentityFile ~/.ssh/id_ed25519_github`：指定使用这把 GitHub 专用密钥。
- `IdentitiesOnly yes`：只使用显式配置的密钥，避免 SSH agent 或 1Password agent 尝试其他 key。

## 验证

配置后执行：

```bash
ssh -T git@github.com
```

看到以下输出即表示认证成功：

```text
Hi Provenr! You've successfully authenticated, but GitHub does not provide shell access.
```

该命令返回 `Exit status 1` 是正常现象，因为 GitHub 不提供交互式 shell。

