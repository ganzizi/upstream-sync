# upstream-sync

这个公开仓库用来定时把公开上游合并进私人仓库。

私人仓库上的定时和这里重复时会多做一次合并，打包时间就浪费了。
所以定时只放在这里，每个私人仓库每天只合并一次。有镜像的，同一次任务里再打包一次。
`.github/workflows/sync-one.yml` 是共用的合并步骤，自己没有定时。
`.github/workflows/point-one.yml` 把 `ghcr-build` 指到刚合并的提交，私人仓库才会打包。

私人仓库名和写入密钥只放在本仓库的 Actions secrets 里，不写进文件。

## 服务器拉取

时间是上海时间。GitHub 的 cron 用 UTC，下面写成了对应的 UTC。
这些时刻是服务器上的拉取时间，不是打包时间。打包排在拉取之前，每天一次。

| 上海时间 | 服务器任务 | 这里怎么处理 |
| --- | --- | --- |
| 05:10 | new-api | 用的是上游镜像，不在这里打包 |
| 05:20 | cli-proxy | 用的是上游镜像，不在这里打包 |
| 05:27 | sub2api | 个人 fork 自己每天 01:40 合并并发布镜像 |
| 05:30 | workbuddy | 本仓库每天 02:10 合并并点一次镜像。服务器时间保持 05:30 |
| 05:40 | grok2api | 个人 fork 自己每天 03:10 同步一次。服务器只 `docker pull`，然后 `up --no-build` |
| 05:50 | chatgpt2api | 用的是上游镜像，不在这里打包 |
| 06:10 | antigravity | 用的是上游镜像，不在这里打包 |
| 06:20 | resin | 用的是上游镜像，不在这里打包 |
| 06:30 | aimilivpn | 没有对应的个人 fork 打包 |
| 06:53 | goo-panel2api | 本仓库每天 02:40 合并并点一次镜像。服务器每小时第 53 分钟核对已投递镜像，不在机器上编译 |

没有排进早上拉取的仓库也是每天一次：`goo-tool-wx` 20:17，`goo-auto-basic` 20:37，`grok-register` 20:47。

## 增加一个私人仓库

1. 在那个私人仓库上另建一把写入用的 deploy key。不要改已经在用的只读 key。
2. 在本仓库添加两个 Actions secrets：一个值是 `owner/name`，一个值是这把 key 的私钥。
3. 在对应分组的 workflow 里加一个 job，调用 `sync-one.yml`。不同产品新建一个 workflow 文件，定时就互不影响。
4. 定时写成每天一次。不要写成每小时，否则私人仓库会反复打包。

同一组里的现有密钥不用改名。`workbuddy` 现在使用 `MANAGER_REPOSITORY`、`MANAGER_DEPLOY_KEY`、`API_REPOSITORY` 和 `API_DEPLOY_KEY`。
`goo-auto-basic` 使用 `BASIC_REPOSITORY` 和 `BASIC_DEPLOY_KEY`，上游是 `tiantianGPU/reg-factory`。
`grok-register` 使用 `GROK_REGISTER_REPOSITORY` 和 `GROK_REGISTER_DEPLOY_KEY`，上游是 `kaibush/grok-register`。它的 `git_ref` 是 `main`。注册机主分支已经回到 v1.0.14 并保留成功率修复，上游合并进这条 main，不要再改到 `codex/upstream-upgrade`。
`goo-tool-wx` 使用 `GOO_TOOL_WX_REPOSITORY` 和 `GOO_TOOL_WX_DEPLOY_KEY`，上游是 `Prometheus-first/helper` 的 `main`。
`goo-panel2api` 使用 `GOO_PANEL2API_REPOSITORY` 和 `GOO_PANEL2API_DEPLOY_KEY`，上游是 `xu20060420fan-cell/workbuddy2api-trae-qoder-minimax-panel` 的 `main`。镜像由私人仓库在 `ghcr-build` 上编译，再投递到服务器。

新的分组可以照下面这样写。仓库名和密钥名都是占位符：

```yaml
name: example-group

on:
  schedule:
    - cron: "10 18 * * *"
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: upstream-sync-example-group
  cancel-in-progress: false

jobs:
  example:
    uses: ./.github/workflows/sync-one.yml
    secrets:
      repository: ${{ secrets.EXAMPLE_REPOSITORY }}
      deploy_key: ${{ secrets.EXAMPLE_DEPLOY_KEY }}
    with:
      git_ref: main
      upstream_slug: example-owner/example-upstream
      upstream_branch: main
```
