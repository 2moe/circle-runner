# CircleCI Runner

| Languages/語言                                 | ID         |
| ---------------------------------------------- | ---------- |
| English                                        | en-Latn-US |
| [中文](./docs/Readme-zh.md)                    | zh-Hans-CN |
| [中文 (Traditional)](./docs/Readme-zh-Hant.md) | zh-Hant-TW |

## Q&A

> Q: What does this thing do?

A:
  Using GitHub Actions to Invoke CircleCI Machines and Write CI Configuration Using Actions Syntax.

  In a nutshell: the syntax for using Github Actions on a CircleCI machine.

> Q: Why did I create this repo?

A: To make it more convenient to use CircleCI’s arm64 machines while leveraging the existing GitHub Actions ecosystem.

## Get Started

### Step1: Generate PAT

<https://github.com/settings/personal-access-tokens/new>

Settings --> Developer Settings --> Personal Access tokens --> generate

- Expiration: Custom
- Repository access:All repositories

![image](https://github.com/2moe/circle-runner/assets/25324935/201ad663-050f-4b40-8d12-f0e8c5cf765e)

- Permissions:
  - Administration: Read and Write

Finally click **generate token**。

### Step2: Create CircleCI Context

Organization Settings --> Contexts --> Create Context

![org settings](https://github.com/2moe/circle-runner/assets/25324935/4c6ae216-9383-4f71-9233-ea8838279788)

![create contexts](https://github.com/2moe/circle-runner/assets/25324935/2fb7020a-5d17-4f3a-b80a-baf6437156e4)

### Step3: Add Secret Environment Variables

![env](https://github.com/2moe/circle-runner/assets/25324935/cf5c688c-3a12-4268-a452-8386fae45007)

name is **_PAT**, value is the value of the token created in Step1 (e.g., github_pat_12A34Bxxyy).

### Step 4: Implement Specific CI Workflows

You can refer to the following files in this repository: **.circleci/config.yml** and **.github/workflows/circle-arm.yml**. Use them as a guide to implement your own CI workflow.

- For CircleCI: Deploy a GitHub self-hosted runner on CircleCI. You likely won't need to make significant changes to this part.
- For GitHub Actions: Focus on implementing the core CI tasks specific to your project.

These are the only things to focus on in **.circleci/config.yml**.

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

Please change the `REPO: 2moe/circle-runner` to your own repository.

If the value of REPO does not contain "/", it will be automatically recognized as an organization.

For example, `REPO: 2cd` refers to an organization.

---

If you need to use an x64 environment:

- Change `arm.medium` to `medium` or `large`.
- Change `ARCH: arm64` to `ARCH: x64`.

See also:

- <https://circleci.com/docs/using-linuxvm/>
- <https://circleci.com/product/features/resource-classes/#arm>

## Other Notes

### Obtain (generate) A Registration Token

```zsh
# shell: zsh

# e.g., github_pat_123456ABC_xxyy
token=""

# A project can either be an organization project or a personal project.
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

See also：[REST API/Create a registration token for a repository](https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#create-a-registration-token-for-a-repository)

### Why Do We Need a PAT (Token)?

Isn't it simpler to use the token provided when creating a runner directly?

> Settings → Runners → Add new self-hosted runner
![Screenshot_2024-05-23__20-54-12](https://github.com/2moe/circle-runner/assets/25324935/b6298ff6-395c-407a-a71d-44ded967fb95)

**Answer**: The registration token has an expiration period, which defaults to one hour.

For environments that change frequently, manually obtaining a new token each time it expires can be cumbersome.

Using a personal access token automatically generates a new registration token, making it even easier.
