# upstream-sync

这个公开仓库用来定时把公开上游合并进私人仓库。

私人仓库上的 GitHub 定时没有投递过 schedule 事件，所以定时放在这里。
`.github/workflows/sync-one.yml` 是共用的合并步骤，自己没有定时。
其他每个 workflow 文件是一组定时任务。`workbuddy` 是其中一组。

私人仓库名和写入密钥只放在本仓库的 Actions secrets 里，不写进文件。

## 增加一个私人仓库

1. 在那个私人仓库上另建一把写入用的 deploy key。不要改已经在用的只读 key。
2. 在本仓库添加两个 Actions secrets：一个值是 `owner/name`，一个值是这把 key 的私钥。
3. 在对应分组的 workflow 里加一个 job，调用 `sync-one.yml`。不同产品新建一个 workflow 文件，定时就互不影响。

同一组里的现有密钥不用改名。`workbuddy` 现在使用 `MANAGER_REPOSITORY`、`MANAGER_DEPLOY_KEY`、`API_REPOSITORY` 和 `API_DEPLOY_KEY`。
`goo-auto-basic` 使用 `BASIC_REPOSITORY` 和 `BASIC_DEPLOY_KEY`，上游是 `tiantianGPU/reg-factory`。

新的分组可以照下面这样写。仓库名和密钥名都是占位符：

```yaml
name: example-group

on:
  schedule:
    - cron: "23 * * * *"
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
