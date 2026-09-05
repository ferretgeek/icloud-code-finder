# iCloud 验证码查找

中文 · [English](README_EN.md)

从 iCloud 邮箱最近的邮件中查找验证码，可按时间和收件地址筛选，方便查找发给某个别名的邮件。

适用环境：Windows、macOS 或 Linux，Python 3.10+；需 iCloud 邮箱和 Apple App 专用密码，并联网读取邮箱，不能使用 Apple 登录密码。

[使用准备](#使用准备) · [本地运行](#本地运行) · [部署教程](docs/DEPLOYMENT.md)

## 界面

![合成演示中的验证码结果](./docs/images/dashboard.png)

![入口与隐私边界设计](./docs/images/intro.png)

## 它刻意只做这些

- 固定连接 `imap.mail.me.com:993`，使用系统 CA 校验 TLS。
- **只读方式**打开 `INBOX`：先读 `RFC822.SIZE` 和有限头部，再通过上限 1 MiB 的 `BODY.PEEK[]` 范围抓取最近邮件；超限或损坏的邮件单独跳过。
- 按时间和目标收件地址筛选，识别 4–8 位验证码。
- 返回验证码、主题、时间，以及**遮罩后的**发件人与收件人。
- 晴空、青玉、晚霞和深灰四套全局主题，隐私遮罩和响应式界面。

它**不会**保存邮箱、密码、邮件或扫描结果；**不会**使用 Apple 登录密码、2FA、Cookie、令牌、私有 API、浏览器自动化或账号批量操作。

## 本地运行

需要 Python 3.10+，运行时只使用 Python 标准库。

```bash
python -m venv .venv
```

Windows：

```powershell
.venv\Scripts\python -m pip install .
.venv\Scripts\mail-lantern
```

macOS / Linux：

```bash
.venv/bin/python -m pip install .
.venv/bin/mail-lantern
```

打开终端显示的、带临时 `#token=` 片段的网址。浏览器读到之后会立刻把令牌从地址栏移除。

只想看虚构数据：

```bash
mail-lantern --demo
```

## 使用准备

1. 在 Apple 账户页面创建 **App 专用密码**——**不要输入你的 Apple 登录密码。**
2. 填写 iCloud 邮箱。如果只想查某个「隐藏邮件地址」或别名，可以填目标收件地址。
3. 选择邮件数量和时间范围，然后开始扫描。

参考：[Apple 官方 · 使用 App 专用密码](https://support.apple.com/102654)

## 技术上值得一提的地方

**抓取是分两步的，为了不下载大附件。** 先读 `RFC822.SIZE` 和有限的头部，判断这封信值不值得取正文，再用上限 1 MiB 的 `BODY.PEEK[]` 范围获取。`PEEK` 意味着**不会把邮件标记为已读**——你在手机上看到的未读状态不会因为扫描而改变。

**超限和损坏的邮件单独跳过。** 一封畸形邮件不会让整次扫描失败，而是被记为跳过并继续。

**没有 JavaScript 也能用，而且不泄露。** 即使浏览器禁用 JavaScript，表单也只使用 POST——**邮箱和密码永远不会出现在 URL 里**，因此不会进入浏览器历史、Referer 或服务器日志。

**访问令牌走 URL fragment。** 令牌放在 `#` 之后，浏览器不会把它发给服务器；前端读取后立即从地址栏移除。

**目的地是写死的。** 只连 Apple 公布的 iCloud IMAP 主机。一个能拿着你的 App 专用密码去连任意主机的工具，本身就是钓鱼工具。

## 部署与安全

本机默认只监听 `127.0.0.1:8769`。

服务器部署**必须**保持应用监听回环地址，通过 SSH 隧道或 HTTPS 反向代理访问；**不要把明文 HTTP 直接暴露到公网。**

## 它不做什么

- 不发信、不删信、不标已读、不改动邮箱任何内容。
- 不接受 Apple 登录密码，不做 2FA / Cookie / 令牌登录。
- 不调用未公开的 Apple 接口，不做浏览器自动化，不做账号批量操作。
- 不保存邮箱、密码、邮件或扫描结果。

## 验证

```bash
python -m unittest discover -s tests -v
ruff check .
ruff format --check .
bandit -r src -ll
```

## 更多文档

[部署教程](./docs/DEPLOYMENT.md) · [隐私边界](./docs/PRIVACY.md) · [架构说明](./docs/ARCHITECTURE.md) · [发布审计](./docs/发布审计.md) · [版本变更](./CHANGELOG.md) · [参与贡献](./CONTRIBUTING.md) · [安全报告](./SECURITY.md)

## 许可与声明

MIT License，见 [LICENSE](./LICENSE)。

这是独立的社区项目，与 Apple 没有隶属、授权或背书关系。请只用它访问你自己的邮箱。
