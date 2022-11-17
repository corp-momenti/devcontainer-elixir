---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
---

<style>
* {
  word-break: keep-all;
}
pre {
  font-size: 80%;
}
</style>

![bg left:40% 80%](https://user-images.githubusercontent.com/286950/201510781-5b7a2230-a60e-4a63-9a0e-7f599a0c36e2.png)

# Remote Development

with Github Codespaces

[@Bret](https://github.com/chitacan)

---

## Who am I

- 김경열
- Bret [@momenti](https://momenti.tv)
- Remote Development 에 관심이 많은 개발자 😎

---

## Contents

**서버 개발 환경**을 셋업하고, 운영하는 것을 중심으로 진행해볼까 합니다.

- Codespace [@momenti](https://momenti.tv)
- Devcontainer & Codespace
- Codespace 시작하기
- Codespace 환경 셋업하기
- Codespace 운영하기 + Tips & Tricks
- Codespace 장점과 단점
- 사전 질문들 답변

---

## Objective

Codespaces 환경을 구축하고, 운영하는데 고려해야할 부분들을 알 수 있다.

---

## Codespace [@momenti](https://momenti.tv)

- nix 로 개발 환경을 셋업 (~ 2022. 6)
- 서버 개발 환경을 devcontainer 로 마이그레이션 (2022. 7 ~)
- 현재는 4명의 서버 개발자들이 Codespace 사용 중
  - 사용시간은 월 평균 600 ~ 700 시간
  - 3개 저장소에 Codespace 셋업

---

![bg left:49% contain](https://user-images.githubusercontent.com/286950/201835897-065e3c3e-d817-4199-b940-3af9b7bc03c3.png)

**Devcontainer & Codespace**

- Devcontainer
  - 컨테이너에 개발 환경을 구축할 수 있는 스펙 모음
  - `devcontainer.json`
- Codespace
  - 선택한 VM 에 Devcontainer 로 개발 환경을 셋업

---

## Codespace 시작하기 (1)

- 저장소 페이지
  - Code 버튼
  - Use this template 버튼 (✨ Universe 2022)
- [Codespaces](https://github.com/codespaces)
  - 내가 생성한 Codespace 의 목록 확인
- vscode ➡️ `Remote Explore: Focus on Github Codespaces View`
- github cli ➡️ `$ gh cs list`

---

![bg left:45% contain](https://user-images.githubusercontent.com/286950/201519586-3d759e5f-fd2d-43d8-b397-c069c0ae7b9f.png)

**Codespace 시작하기 (2)**

👉 [devcontainer-elixir](https://github.com/corp-momenti/devcontainer-elixir)

- momenti 의 elixir 개발환경
- Elixir 1.13.4 (Erlang/OTP 24)
- Node v14.21.0
- Postgres, Jaeger

---

## Codespace 환경 셋업하기 (1)

`.devcontainer/devcontainer.json`

```json
{
  "name": "devcontainer-elixir",
  "dockerComposeFile": ["docker-compose.yml", "docker-compose.external.yml"],
  "service": "app", // primary container
  "workspaceFolder": "/home/vscode/workspace",
  "customizations": {
    "vscode": {
      "extensions": []
    },
    "codespaces": {
      "openFiles": ["docs/welcome.md"]
    }
  },
  "forwardPorts": [4000, 5432, 8080, 16686],
  "remoteUser": "vscode"
}
```

---

## Codespace 환경 셋업하기 (2)

`.devcontainer/docker-compose.yml`

```yml
version: '3.8'

services:
  app: # primary container
    image: ghcr.io/corp-momenti/docker-images/momenti-elixir-dev:8339e0 # base image
    volumes:
      - ..:/home/vscode/workspace:cached
      - config:/home/vscode/.config # persist configs after restart or rebuild
      - extensions:/home/vscode/.vscode-server/extensions
    command: sleep infinity
    init: true
    privileged: true
    network_mode: service:traefik # allow access to traefik service container
volumes:
  config:
  extensions:
```

---

![bg left:49% contain](https://user-images.githubusercontent.com/286950/201957222-aa329113-6501-49c6-a7ec-b2ba2886c77c.png)

**Codespace 환경 셋업하기 (3)**

베이스 이미지 정하기

- Primary Container 가 될 이미지
- 런타임과 도구들을 포함
  - Elixir, Node.js, Go ...
  - fish, fzf, gcloud, ripgrep ...

---

## Codespace 환경 셋업하기 (4)

[베이스 이미지를 Dockerfile 로](https://github.com/microsoft/vscode-remote-try-node/blob/main/.devcontainer/devcontainer.json) 설정할 수 있음

```json
{
  "name": "Node.js",
  "build": {
    "dockerfile": "Dockerfile",
    "args": { "VARIANT": "16-bullseye" }
  }
}
```
```Dockerfile
ARG VARIANT=16-bullseye
FROM mcr.microsoft.com/devcontainers/javascript-node:0-${VARIANT}
RUN apt-get update && apt-get -y install --no-install-recommends \
    && fzf
```

👉 런타임과 도구가 미리 설치되어 있는 베이스 이미지를 만드는 것을 추천

<!-- codespace prebuild 를 설정하면 dockerfile 빌드는 생략가능함 -->
<!-- 하지만 rebuild 는 dockerfile 을 다시 빌드함 -->

---

## Codespace 환경 셋업하기 (5)

Codespace 관련 팀에서 제공하는 이미지들

- 👴 [vscode-dev-containers](https://github.com/microsoft/vscode-dev-containers/tree/main/containers)
  - node, go 부터 dart, swift 까지 다양한 이미지를 devcontainer 지원 이미지로 변경하는 방법이 모여 있는 저장소
- 🐣 [devcontainers/images](https://github.com/devcontainers/images/tree/main/src)
  - https://containers.dev/ 발표 이후 새롭게 셋업된 저장소
  - (vscode-dev-containers 보다) Dockerfile 이 훨씬 심플함

👉 [devcontainers/images](https://github.com/devcontainers/images) 로 부터 팀에 필요한 도구를 설정하는 것을 추천

---

## Codespace 환경 셋업하기 (6)

이미지 하나로 개발 환경 (Codespace), 배포 환경을 모두 지원하는 것은 비추

- 배포 환경에서는 개발 도구들이 필요없다.
- 이미지의 사이즈가 곧 배포의 속도

👉 용도에 따라 `base`, `dev`, `run` 3가지로 나누는 것을 추천

---

**Codespace 환경 셋업하기 (7)**

![bg left:40% contain](https://user-images.githubusercontent.com/286950/201968902-e5f793e6-fe60-4e27-81ae-ad6135f84e4c.png)

- `base` ➡️ `dev`, `run` 에 모두 필요한 공통 의존성
- `dev` ➡️ 개발 환경에(Codespace) 필요한 의존성
- `run` ➡️ 배포 환경에 필요한 의존성
- 이미지들은 [Github Packages](https://github.com/orgs/corp-momenti/packages) 로 관리

---

## Codespace 운영하기 (1)

[Codepsace prebuilds 설정](https://docs.github.com/en/codespaces/prebuilding-your-codespaces/about-github-codespaces-prebuilds)

- prebuilds 를 사용할 region, history 설정
- Codespace 실행과 동시에 어플리케이션을 실행할 수 있도록 [Lifecycle scripts](https://containers.dev/implementors/json_reference/#lifecycle-scripts) 설정

```json
{
  "updateContentCommand": "mix deps.get && mix compile",
}
```

---

![bg left:40% contain](https://user-images.githubusercontent.com/286950/201525812-3f2eae15-7a08-4de3-9f8b-72edee4d08f1.png)

## Codespace 운영하기 (2)

Codespace policies 설정

- 사용 가능한 Machine Type
- Port visibility
- Maximum idle timeout
- Retention period
- Base images
- Policy target (repo)

---

## Codespace 운영하기 (3)

[devcontainer features](https://code.visualstudio.com/blogs/2022/09/15/dev-container-features)

- 선택적으로 필요한 의존성이나 도구를 `devcontainer.json` 에 추가할 수 있는 스펙
- Github Actions 에서 필요없는 도구들은 features 로 설정하면 좋다.

```json
{
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/docker-in-docker:1": {},
    "ghcr.io/dhoeric/features/google-cloud-cli:1": {}
  }
}
```

---

## Codespace 운영하기 (4)

- dotfiles 설정
- Codespaces secrets 설정
- extensions 설정

```json
{
  "customizations": {
    "vscode": {
      "extensions": ["eamodio.gitlens", "GitHub.vscode-pull-request-github"]
    }
  }
}
```


---

## Codespace 운영 Tips & Tricks (1)

생성한 Codespace VM 을 (삭제하지 않고) 계속 사용하기

- VM 을 새로 생성하는 속도보다, 재시작 속도가 더 빠르다.
- 수정중인 코드나, 시스템에 추가로 설치된 도구들도 VM stop, restart, rebuild 간에 유지됨
- 볼륨을 설정해 원하는 디렉토리를 유지할 수 있음 (e.g `~/.config`)

---

![bg left:35% contain](https://user-images.githubusercontent.com/286950/201529111-9cb01149-0b5a-4a2d-b321-805e57761f9c.png)

**Codespace 운영 Tips & Tricks (2)**

`git worktree` 를 사용해 하나의 Codespace 에서 여러 브랜치를 동시에 관리

```shell
$ git worktree add -b <NEW_BRANCH> ../<NEW_BRANCH>
$ code ../<NEW_BRANCH>
```

---

## Codespace 운영 Tips & Tricks (3)

모든 개발 과정에서 Codespace VM 을 사용할 필요는 없음

- PR 리뷰 과정에서 changeset 에 포함되지 않은 파일을 확인할 때
- 다른 저장소의 코드를 검색, 탐색하고 싶을 때

👉 [github.dev](https://docs.github.com/en/codespaces/the-githubdev-web-based-editor), [Remote Repositories](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-repositories) 활용

---

<style scoped>
li {
  font-size: 80%;
}
</style>

![bg vertical left:44% contain](https://user-images.githubusercontent.com/286950/201836395-1f70580c-ae11-4d20-8cf7-d41b9e563f8e.png)
![bg left:44% contain](https://user-images.githubusercontent.com/286950/201836517-beb33895-b234-4f04-90af-7cad37c1ac59.png)

**Codespace 운영 Tips & Tricks (4-1)**

Devcontainer 에서 docker-compose 사용하기

- `network_mode: service:<NAME>` 을 제외한 나머지 컨테이너와 네트워킹 할때는 `<SERVICE_NAME>:<PORT>` 포맷을 사용해야 함
- 문제는 Codespace 에서는 커스텀 호스트 이름에 대해 port fowarding 을 지원하지 않고 있음 <small>(2022. 11. 13 기준)</small>
- `socat` 을 사용해 컨테이너 간 포워딩을 설정할 수 있지만 불편함

<!-- socat tcp-listen:<PORT>,reuseaddr,fork tcp:<SERVICE_NAME>:<PORT> -->
---

<style scoped>
li {
  font-size: 80%;
}
</style>

![bg left:44% contain](https://user-images.githubusercontent.com/286950/201836784-10367e75-1471-49ab-ba11-5aa3b485ffa2.png)

**Codespace 운영 Tips & Tricks (4-2)**

Devcontainer 에서 docker-compose 사용하기

- [traefik](https://traefik.io/) 을 게이트웨이로 사용해 primary 컨테이너와 다른 컨테이너를 연결
- 나머지 컨테이너로 연결되는 포트들이 primary 컨테이너에 생성되기 때문에 편리하게 port forwarding 을 사용할 수 있음

---

## Codespace 장점

- 가장 편리하게 개발 환경을 통일할 수 있음
  - devcontainer 가 설정된 저장소에는 누구나 Codespace 를 켜서 서버를 실행할 수 있음
  - 개발중인 서버 브랜치를 따로 배포하지 않아도 프론트, 모바일 개발자들과 협업할 수 있음
- PR 에서 곧장 Codespace 를 생성해 코드리뷰를 진행하고 필요하면 테스트와 서버를 실행
- 머신의 사양을 원하는 만큼 사용할 수 있음
  - 개인 머신을 가볍게 유지할 수 있다.

---

<style scoped>
li {
  font-size: 90%;
}
</style>

## Codespace 단점

- 현재 Codespace 가 지원하는 가장 가까운 리전은 싱가폴
  - 거리 4,500 km, 평균 60 ~ 70 ms 정도의 지연
- Github 가 다운되면, Codespace 를 사용할 수 없을지도? 🤔
  - 각자의 머신에서 devcontainer 를 실행하는 방법으로 대응은 가능함
- 커밋하지 않은 파일들을 백업하기 위해서는 반드시 Codespace 를 실행해야 함
- 베이스 이미지를 만들고 유지하는데에도 비용이 발생한다.
- Codespace VM 을 재시작하는데 (약간의) 시간이 걸림
- 비용을 절감이 가능할까?
  - 이미 고사양 노트북을 지급한 이후에 전환한다면 바로 효과를 보기 어렵다.
  - 고사양 [Codespace VM 은 꽤 비싸다](https://docs.github.com/en/billing/managing-billing-for-github-codespaces/about-billing-for-github-codespaces#pricing-for-paid-usage)

---

<style scoped>
li {
  font-size: 80%;
}
</style>

### 사전 질문들

- IntelliJ 와 사용 가능할까요?
  - [Codespaces with JetBrains IDEs](https://github.blog/changelog/2022-11-09-github-codespaces-with-jetbrains-ides-public-beta/)
- [비용 얼마나 나오나요?](https://user-images.githubusercontent.com/286950/201532371-68bd6ade-51ed-4a3d-975d-9fa6f906d14b.png)
- GCP 사용 로그인을 어떻게 하면 더 좋을까요?
  - `docker-compose` volume 에 `~/.config` 를 추가합니다.
- SSH 밀림 개선 가능한가요?
- Code Server 와 비교했을 때 어떤가요?
  - SSH 연결이 되는 상황에서는 vscode 의 [Remote SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) 확장을 사용하고 있어서 한번도 code-server 를 사용해 보지 않음
  - SSH 연결이 가능한 머신이 있다면, Remote SSH & devcontainer 조합을 사용해 Codespace 와 비슷한 환경 셋업 가능
- git GUI client?
  - 주력으로 vanilla git, tig 를 사용하고 있으며 vscode 에서는 gitlens 확장을 사용함
