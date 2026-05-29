# GitHub SSH 走 443

## 结论

Key 没问题，`github.com:22` 被网络干掉了。改走 `ssh.github.com:443`。

## 配置

写到 `~/.ssh/config`：

```sshconfig
Host github.com
    User git
    HostName ssh.github.com
    Port 443
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
    UseKeychain yes
    AddKeysToAgent yes
```

## 为什么

- `Host github.com`：仓库地址不用改，继续用 `git@github.com:owner/repo.git`。
- `HostName ssh.github.com` + `Port 443`：避开被限制的 SSH 22 端口。
- `IdentityFile` + `IdentitiesOnly yes`：只用 GitHub 那把 key，少走弯路。

## 验证

```bash
ssh -T git@github.com
```

看到这句就是成了：

```text
Hi Provenr! You've successfully authenticated, but GitHub does not provide shell access.
```

