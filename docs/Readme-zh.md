# CircleCI Runner

| Languages/語言         | ID         |
| ---------------------- | ---------- |
| 中文                   | zh-Hans-CN |
| [English](./Readme.md) | en-Latn-US |

## Q&A

> Q: 这玩意儿有什么用？

A: 使用 github actions 调用 CircleCI 的机器，并使用 actions 的语法来编写 CI 配置。

> Q: 为什么我会创建这个 repo 呢?

A: 为了更方便地使用 CircleCI 的 arm64 机器，同时利用现有的 actions 生态。

## 快速上手

### Step1. 生成 token

<https://github.com/settings/personal-access-tokens/new>

Settings --> Developer Settings --> Personal Access tokens --> generate

- Expiration: Custom
- Repository access:All repositories

![image](https://github.com/2moe/circle-runner/assets/25324935/201ad663-050f-4b40-8d12-f0e8c5cf765e)

- Permissions:
  - Administration: Read and Write

最后点击 **generate token**。

### Step2. 创建 CircleCI Context

Organization Settings --> Contexts --> Create Context

![org settings](https://github.com/2moe/circle-runner/assets/25324935/4c6ae216-9383-4f71-9233-ea8838279788)

![create contexts](https://github.com/2moe/circle-runner/assets/25324935/2fb7020a-5d17-4f3a-b80a-baf6437156e4)

### Step3. 添加私密环境变量

![env](https://github.com/2moe/circle-runner/assets/25324935/cf5c688c-3a12-4268-a452-8386fae45007)

name 为 _PAT, value 为 Step1 中创建的 token 的值(e.g., github_pat_12A34Bxxyy)。

### Step4. 实现具体的 CI 流程

您可以参考本仓库的 **.circleci/config.yml** & **.github/workflows/circle-arm.yml**， 然后实现自己的 CI 流程。

---

- circleci 的流程：在 circleci 上部署 github self-hosted runner。
- github actions 的流程：核心的 CI 任务，您需要自己实现相关的任务。

---

**.circleci/config.yml** 中需要重点关注的内容只有这些。

```yaml
version: 2.1
jobs:
  setup-runner:
    machine:
      image: ubuntu-2204:current
    resource_class: arm.medium
    # resource_class: arm.large
    environment:
      ARCH: arm64
      REPO: 2moe/circle-runner
```

请将 `REPO: 2moe/circle-runner` 修改为您自己的仓库。

如果 REPO 的值不包含 "/", 则自动识别为组织。

> 组织可以共享 runners, 而不用像个人仓库那样为不同仓库创建不同的 runners。

比如 `REPO: 2cd`, `2cd` 是一个组织。

---

您如果需要使用 x64 的环境，则需：

- 将 `arm.medium` 改为 `medium` 或 `large`。
- 将 `ARCH: arm64` 改为  `ARCH: x64`

另请参阅:

- <https://circleci.com/docs/using-linuxvm/>

- <https://circleci.com/product/features/resource-classes/#arm>

## 其他说明

### 获取 registration token

```zsh
# shell: zsh

# e.g., github_pat_1234567ABC_xxyy
pat_token=""

# 一个项目要么是组织项目，要么是个人项目
# 实际上，这里使用 enum（枚举类型）会更合适。
# enum Project { Personal(String), Org(String) }
{
  # e.g., 2cd
  org=""

  # e.g., 2moe/xxx
  personal_repo=""
}

url="https://api.github.com/repos/$personal_repo/actions/runners/registration-token"

if (($#org)) {
  url="https://api.github.com/orgs/$org/actions/runners/registration-token"
}

args=(
  -L
  -X POST
  -H 'Accept: application/vnd.github+json'
  -H "Authorization: Bearer $pat_token"
  -H 'X-GitHub-Api-Version: 2022-11-28'
  $url
)
curl $args
```

output:

```json
{
  "token": "AGBG5B9I2BBINHD7H6EW7Q3GJ7BFJ",
  "expires_at": "2024-05-23T05:33:41.303+00:00"
}
```

另请参阅：[REST API/Create a registration token for a repository](https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#create-a-registration-token-for-a-repository)

### 为什么需要 PAT (token)?

直接使用创建 runner 时提供的 token 不是更简单吗？

> Settings --> Runners --> Add new self-hosted runner
> ![Screenshot_2024-05-23__20-54-12](https://github.com/2moe/circle-runner/assets/25324935/b6298ff6-395c-407a-a71d-44ded967fb95)

答：因为 registration-token 存在有效期，默认为一小时。

对于频繁变动的环境，每次失效都要手动获取就很麻烦。

而使用 pat (a.k.a, personal-access-token) 可以自动生成新的 registration token，这样会更方便。
