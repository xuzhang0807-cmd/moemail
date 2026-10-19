# moemail（moemail.kazerush.xyz）

## 用途
自托管临时邮箱（MoeMail）：网页收发、OpenAPI 创建邮箱与读取验证码、Webhook；Email Routing catch-all 收信。

## 入口与常用命令
- 站点：`https://moemail.kazerush.xyz`；Pages 项目 `moemail`（Direct Upload）
- 仓库：`https://github.com/xuzhang0807-cmd/moemail`（fork 自 `beilunyang/moemail`，本地 `/root/projects/moemail`）
- 自动同步：`.github/workflows/sync-upstream.yml` 每日 04:00 UTC 合并上游 `master`
- 自动部署：`.github/workflows/deploy-main.yml`，push `master` 或手动触发

## 架构与关键路径
- Next.js on Pages Functions（`uses_functions`），D1 `moemail-db`（`8fc8a228-12a8-46fd-b345-6578292fd486`）+ KV `moemail-kv`（`fef9e00299f14382a7e67f4b272b6f0b`，绑定 `SITE_CONFIG`）
- Worker `email-receiver-worker`：`kazerush.xyz` catch-all 投递入 D1
- Worker `cleanup-worker`：每小时清理过期邮件

## 配置与依赖
- 仓库 Secrets：`CLOUDFLARE_API_TOKEN`、`CLOUDFLARE_ACCOUNT_ID`、`PROJECT_NAME`、`DATABASE_NAME/ID`、`KV_NAMESPACE_NAME/ID`、`CUSTOM_DOMAIN`
- Pages 运行时：`AUTH_SECRET`（另有 GitHub/Google OAuth 配置，均在 Dashboard）
- KV `SITE_CONFIG`：`EMAIL_DOMAINS=kazerush.xyz`、`MAX_EMAILS=1000`、`DEFAULT_ROLE`
- OpenAPI 鉴权：`X-API-Key`（个人中心创建，公爵/皇帝角色）

## 验证方法
```bash
curl -s https://moemail.kazerush.xyz/api/config -H "X-API-Key: <key>"
curl -s -X POST https://moemail.kazerush.xyz/api/emails/generate \
  -H "X-API-Key: <key>" -H 'Content-Type: application/json' \
  -d '{"name":"probe","expiryTime":3600000,"domain":"kazerush.xyz"}'
curl -s https://moemail.kazerush.xyz/api/emails/<id> -H "X-API-Key: <key>"
```
发信测试：从允许的源站直接投递到 `route1.mx.cloudflare.net:25`（需 SPF 放行），收信后按上一步读取。

## 禁区与已知坑
- API Key 在 D1 `api_keys.key` 为明文，泄露即失效，需及时删除重建。
- Pages 项目是 Direct Upload，无法改成 Git 集成；部署走 GitHub Actions + `scripts/deploy`。
- 上游默认分支是 `master`，不是 `main`。
- 部署脚本会按 `DATABASE_NAME`/`KV_NAMESPACE_NAME` 查找资源，名称写错会新建空库。

## 当前状态
- 2026-10-19：fork + 每日同步 + push 自动部署已跑通；API 创建邮箱、收信、读验证码端到端验证通过。
