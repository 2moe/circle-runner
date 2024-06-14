# CircleCI Runner

| Languages/語言                            | ID         |
| ----------------------------------------- | ---------- |
| 中文 (Traditional)                       | zh-Hant-TW |
| [English](../Readme.md)                   | en-Latn-US |
| [中文 (Simplified)](./Readme-zh.md)      | zh-Hans-CN |

## Q&A

> Q: 這玩意兒有什麼用？

A: 使用 github actions 呼叫 CircleCI 的機器，並使用 actions 的語法來編寫 CI 配置。

> Q: 為什麼我會建立這個 repo 呢?

A: 為了更方便地使用 CircleCI 的 arm64 機器，同時利用現有的 actions 生態。

## 快速上手

### Step1. 生成 token

<https://github.com/settings/personal-access-tokens/new>

Settings --> Developer Settings --> Personal Access tokens --> generate

- Expiration: Custom
- Repository access:All repositories

![image](https://github.com/2moe/circle-runner/assets/25324935/201ad663-050f-4b40-8d12-f0e8c5cf765e)

- Permissions:
  - Administration: Read and Write

最後點選 **generate token**。

### Step2. 建立 CircleCI Context

Organization Settings --> Contexts --> Create Context

![org settings](https://github.com/2moe/circle-runner/assets/25324935/4c6ae216-9383-4f71-9233-ea8838279788)

![create contexts](https://github.com/2moe/circle-runner/assets/25324935/2fb7020a-5d17-4f3a-b80a-baf6437156e4)

### Step3. 新增私密環境變數

![env](https://github.com/2moe/circle-runner/assets/25324935/cf5c688c-3a12-4268-a452-8386fae45007)

name 為 _PAT, value 為 Step1 中建立的 token 的值(e.g., github_pat_12A34Bxxyy)。

### Step4. 實現具體的 CI 流程

您可以參考本倉庫的 **.circleci/config.yml** & **.github/workflows/circle-arm.yml**， 然後實現自己的 CI 流程。

---

- circleci 的流程：在 circleci 上部署 github self-hosted runner。
- github actions 的流程：核心的 CI 任務，您需要自己實現相關的任務。

---

**.circleci/config.yml** 中需要重點關注的內容只有這些。

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

請將 `REPO: 2moe/circle-runner` 修改為您自己的倉庫。

如果 REPO 的值不包含 "/", 則自動識別為組織。

> 組織可以共享 runners, 而不用像個人倉庫那樣為不同倉庫建立不同的 runners。

比如 `REPO: 2cd`, `2cd` 是一個組織。

---

您如果需要使用 x64 的環境，則需：

- 將 `arm.medium` 改為 `medium` 或 `large`。
- 將 `ARCH: arm64` 改為  `ARCH: x64`

另請參閱:

- <https://circleci.com/docs/using-linuxvm/>

- <https://circleci.com/product/features/resource-classes/#arm>

## 其他說明

### 獲取 registration token

```zsh
# shell: zsh

# e.g., github_pat_123456ABC_xxyy
pat_token=""

# 一個專案要麼是組織專案，要麼是個人專案
  # e.g., 2cd
  org=""

  # e.g., 2moe/xxx
  personal_repo=""

url="https://api.github.com/repos/$personal_repo/actions/runners/registration-token"

if (($#org)) {
  url="https://api.github.com/orgs/$org/actions/runners/registration-token"
}

args=(
  -L
  -X POST
  -H 'Accept: application/vnd.github+json'
  -H "Authorization: Bearer $token"
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

另請參閱：[REST API/Create a registration token for a repository](https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#create-a-registration-token-for-a-repository)

### 為什麼需要 PAT (token)?

直接使用建立 runner 時提供的 token 不是更簡單嗎？

> Settings --> Runners --> Add new self-hosted runner
> ![Screenshot_2024-05-23__20-54-12](https://github.com/2moe/circle-runner/assets/25324935/b6298ff6-395c-407a-a71d-44ded967fb95)

答：因為 registration-token 存在有效期，預設為一小時。

對於頻繁變動的環境，每次失效都要手動獲取就很麻煩。

而使用 pat (a.k.a, personal-access-token) 可以自動生成新的 registration token，這樣會更方便。
