# 인증

Specify CLI는 카탈로그 소스, 확장 다운로드, 릴리스 확인을 위한 HTTP 요청에
**선택형(opt-in) 인증**을 사용합니다. 명시적으로 설정하지 않는 한 자격 증명은
전송되지 않습니다.

## 설정

인증을 활성화하려면 `~/.specify/auth.json`을 만듭니다.

```json
{
  "providers": [
    {
      "hosts": ["github.com", "api.github.com", "raw.githubusercontent.com", "codeload.github.com"],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_TOKEN"
    }
  ]
}
```

> **보안:** 파일을 소유자만 접근할 수 있도록 제한하세요.
>
> ```bash
> chmod 600 ~/.specify/auth.json
> ```

이 파일이 없으면 모든 HTTP 요청은 인증 없이 이루어집니다.

## 필드

`providers` 배열의 각 항목에는 다음 필드가 있습니다.

| 필드 | 필수 | 설명 |
|---|---|---|
| `hosts` | 예 | 이 항목이 적용되는 호스트 이름 배열. 정확한 호스트 이름, 또는 하위 도메인에만 해당하는 맨 앞 `*.` 와일드카드(예: `*.visualstudio.com`)를 지원합니다. `*.visualstudio.com`은 `foo.visualstudio.com`과 일치하지만 `visualstudio.com`과는 일치하지 않습니다. `*github.com`이나 `gith?b.com` 같은 다른 glob 패턴은 지원되지 않습니다. |
| `provider` | 예 | 내장 제공자 키: `github`, `azure-devops`, `bitbucket`. |
| `auth` | 예 | 인증 방식(아래 참고). |
| `token` | 아니요 | 토큰 값(인라인). 가능하면 `token_env`를 대신 사용하세요. |
| `token_env` | 아니요 | 토큰을 읽어 올 환경 변수 이름. |
| `username` | `basic`일 때 | Basic 자격 증명의 사용자 이름 부분. Bitbucket API 토큰의 경우 Atlassian 계정 이메일입니다. `:`를 포함할 수 없습니다. |

`azure-ad` 인증에는 추가 필드가 필요합니다.

| 필드 | 필수 | 설명 |
|---|---|---|
| `tenant_id` | 예 | Azure AD 테넌트 ID. |
| `client_id` | 예 | 서비스 주체 클라이언트 ID. |
| `client_secret_env` | 예 | 클라이언트 시크릿을 담은 환경 변수. |

`bearer`, `basic-pat`, `basic` 방식에는 `token` 또는 `token_env` 중 하나가 설정되어야 합니다.

## 제공자와 인증 방식

### GitHub (`github`)

| 방식 | 헤더 | 용도 |
|---|---|---|
| `bearer` | `Authorization: Bearer <token>` | PAT, 세분화된 PAT, OAuth 토큰, GitHub App 토큰 |
| `github-cli` | `Authorization: Bearer <token>` | `gh auth token`으로 필요할 때 가져오는 자격 증명 |

**예시 — 환경 변수를 통한 PAT:**

```json
{
  "hosts": ["github.com", "api.github.com", "raw.githubusercontent.com", "codeload.github.com"],
  "provider": "github",
  "auth": "bearer",
  "token_env": "GH_TOKEN"
}
```

**예시 — GitHub CLI:**

```json
{
  "providers": [
    {
      "hosts": [
        "github.com",
        "api.github.com",
        "raw.githubusercontent.com",
        "codeload.github.com"
      ],
      "provider": "github",
      "auth": "github-cli"
    }
  ]
}
```

이 모드를 사용하기 전에 [GitHub CLI](https://cli.github.com/)를 설치하고
`gh auth login`을 실행하세요. Specify는 고정된 명령
`gh auth token --hostname github.com`을 셸 없이 실행하고, 결과를 현재 프로세스
동안 메모리에 캐시하며, 토큰을 디스크에 절대 기록하지 않습니다.
앞뒤 공백은 제거됩니다. 비ASCII 문자, 내장된 캐리지 리턴, 줄바꿈을 포함한 토큰은
오류에 토큰을 포함하지 않은 채 거부됩니다.
자격 증명 획득에 실패하면 다른 자격 증명이나 비인증 요청을 시도하지 않고 해당
요청이 실패합니다. 기존 URL 오류 범주를 사용하므로 호출자는 평소의 오류
처리(예: 사용할 수 없는 카탈로그에 대해 경고하고 건너뛰기)를 유지합니다.
릴리스 확인과 업그레이드는 네트워크 장애를 보고하는 대신 인증 진단을 보존합니다.
온라인 번들 카탈로그 가져오기와 참조 검증은 인증 실패를 패키지된 스냅샷이나
확인되지 않은 참조로의 가용성 폴백이 아니라 오류로 취급합니다.

`github-cli`에 한해, **`hosts`의 첫 번째 호스트 이름**이
`gh auth token --hostname`에 전달됩니다. `github.com` 또는 엔터프라이즈
인스턴스의 GitHub CLI 로그인 호스트 이름을 맨 앞에 두세요. 와일드카드가 아니라
정확한 호스트 이름이어야 합니다. 나열된 모든 호스트는 그 자격 증명을 받도록
승인된 요청 매칭 대상으로 남으며, 이후 항목은 지원되는 와일드카드를 사용할 수
있습니다. Specify는 서비스 접두사나 요청된 URL로부터 자격 증명 호스트 이름을
추론하지 않습니다. 메모리 내 캐시는 첫 번째 호스트 이름을 키로 사용합니다.
서로 다른 자격 증명이 필요한 호스트에는 별도의 제공자 항목을 사용하세요. 호스트
순서는 기존 인증 모드에는 영향을 주지 않습니다.

### GitHub Enterprise Server (GHES)

GitHub Enterprise Server 인스턴스에 호스팅된 비공개 카탈로그나 확장을 사용하려면
GHES 호스트를 나열한 `github` 항목을 추가하세요. 같은 항목이 카탈로그 JSON
가져오기**와** 비공개 릴리스 자산 다운로드를 모두 인증합니다. Specify는 나열된
호스트를 GitHub Enterprise로 인식하고 GHES REST API(`/api/v3`)를 통해 릴리스
다운로드를 해석합니다.

```json
{
  "providers": [
    {
      "hosts": ["ghes.example.com", "raw.ghes.example.com", "codeload.ghes.example.com"],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_ENTERPRISE_TOKEN"
    }
  ]
}
```

**접두사 없는** 웹 호스트(예: `ghes.example.com`)를 나열하세요. 릴리스 다운로드
URL이 그곳에 있습니다. 인스턴스가 하위 도메인 격리를 사용한다면 카탈로그/확장
URL이 사용하는 `raw.` 및 `codeload.` 하위 도메인도 나열하세요.
`*.ghes.example.com` 와일드카드는 하위 도메인과 일치하지만 접두사 없는 호스트와는
일치하지 **않으므로**, 접두사 없는 호스트를 항상 명시적으로 포함하세요.

`auth: "github-cli"`를 사용할 때는 엔터프라이즈 인스턴스의 GitHub CLI 로그인
호스트 이름을 맨 앞에 두고, 그 뒤에 같은 자격 증명을 받아야 하는 요청 호스트나
와일드카드 패턴을 나열하세요.

```json
{
  "hosts": [
    "ghes.example.com",
    "raw.ghes.example.com",
    "codeload.ghes.example.com"
  ],
  "provider": "github",
  "auth": "github-cli"
}
```

Specify는 첫 번째 호스트 이름을 설정된 그대로 사용해
`gh auth token --hostname ghes.example.com`을 실행합니다. 별개의 GitHub
인스턴스에는 별도의 제공자 항목을 사용하세요. 선택된 호스트는 GitHub CLI에서 이미
인증되어 있어야 합니다. 실패 시에는 GitHub CLI 명령 출력을 포함하지 않고
`gh auth login --hostname <host>`를 제안합니다.

### 데이터 상주를 지원하는 GitHub Enterprise Cloud (GHE.com)

GHE.com 테넌트는 웹과 REST API에 별도의 호스트 이름을 사용합니다. Specify가 릴리스
메타데이터 조회와 자산 다운로드를 인증할 수 있도록 두 호스트를 모두 나열하세요.

```json
{
  "providers": [
    {
      "hosts": ["tenant.ghe.com", "api.tenant.ghe.com"],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_ENTERPRISE_TOKEN"
    }
  ]
}
```

`tenant.ghe.com`의 릴리스 URL에 대해 Specify는 `api.tenant.ghe.com`을 통해
메타데이터와 릴리스 자산을 해석합니다. 반환된 자산은 같은 소유자와 저장소를
식별해야 하며 정확한 숫자 `/repos/.../releases/assets/...` 엔드포인트를 사용해야
합니다. `*.tenant.ghe.com` 같은 와일드카드는 API 호스트 이름과 일치하지만 접두사
없는 테넌트 웹 호스트 이름과는 일치하지 않으므로, 웹 호스트 이름을 명시적으로
나열하세요.

### Azure DevOps (`azure-devops`)

| 방식 | 헤더 | 용도 |
|---|---|---|
| `basic-pat` | `Authorization: Basic base64(:<PAT>)` | 개인 액세스 토큰 |
| `bearer` | `Authorization: Bearer <token>` | 미리 획득한 OAuth / Azure AD 토큰 |
| `azure-cli` | `Authorization: Bearer <token>` | `az account get-access-token`으로 획득한 토큰 |
| `azure-ad` | `Authorization: Bearer <token>` | OAuth2 클라이언트 자격 증명 흐름으로 획득한 토큰 |

**예시 — 환경 변수를 통한 PAT:**

```json
{
  "hosts": ["dev.azure.com"],
  "provider": "azure-devops",
  "auth": "basic-pat",
  "token_env": "AZURE_DEVOPS_PAT"
}
```

**예시 — Azure CLI(대화형 로그인):**

```json
{
  "hosts": ["dev.azure.com"],
  "provider": "azure-devops",
  "auth": "azure-cli"
}
```

사전에 `az login`이 실행되어 있어야 합니다.

**예시 — Azure AD 서비스 주체(CI/자동화):**

```json
{
  "hosts": ["dev.azure.com"],
  "provider": "azure-devops",
  "auth": "azure-ad",
  "tenant_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "client_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "client_secret_env": "AZURE_CLIENT_SECRET"
}
```

### Bitbucket (`bitbucket`)

| 방식 | 헤더 | 용도 |
|---|---|---|
| `bearer` | `Authorization: Bearer <token>` | 저장소/프로젝트/워크스페이스 액세스 토큰, 또는 계정 이메일 없이 보내는 Atlassian API 토큰(Bitbucket Cloud, **`api.bitbucket.org` 전용** — 아래 참고). HTTP 액세스 토큰(Bitbucket Data Center) |
| `basic` | `Authorization: Basic base64(<username>:<token>)` | `api.bitbucket.org`에 대한 Atlassian API 토큰(사용자 이름 = Atlassian 계정 이메일). Bitbucket Cloud 앱 비밀번호는 2026년 7월 28일에 Atlassian이 제거했습니다. 대신 API 토큰이나 액세스 토큰을 사용하세요. |

> **호스트 참고:** Bitbucket Cloud는 표면을 호스트별로 나눕니다. `api.bitbucket.org`는
> 모든 REST 호출에서 저장소/프로젝트/워크스페이스 액세스 토큰을 Bearer로 받고,
> Atlassian API 토큰을 Basic(계정 이메일을 사용자 이름으로)이나 Bearer(이메일 불필요)로
> 받습니다. 여기에는
> `GET /2.0/repositories/<workspace>/<repo>/src/<ref>/<path>`를 통한 카탈로그 파일
> 가져오기도 포함됩니다. 일반 `bitbucket.org` 웹 호스트(브라우저 페이지,
> `.../raw/...` 링크, HTTPS를 통한 `git clone`)는 이 둘 중 어느 것도 받지
> **않습니다**. 그곳은 문자 그대로의 사용자 이름 `x-token-auth`와 액세스 토큰을
> 비밀번호로 쓰는 HTTP Basic으로 HTTPS 기반 git을 인증합니다. `auth.json`의 자격
> 증명이 실제로 적용되도록 카탈로그와 `download_url` 항목이
> `bitbucket.org/.../raw/...`가 아니라 `api.bitbucket.org`(아래)를 가리키게 하세요.

**예시 — Bitbucket Cloud 액세스 토큰(권장):**

```json
{
  "hosts": ["api.bitbucket.org"],
  "provider": "bitbucket",
  "auth": "bearer",
  "token_env": "BITBUCKET_ACCESS_TOKEN"
}
```

카탈로그와 아카이브를 호스팅하는 저장소(또는 프로젝트/워크스페이스)에 대해
**Repositories: Read** 범위로 토큰을 만드세요. 이 자격 증명으로 카탈로그 파일을
가져올 때는 `bitbucket.org/.../raw/...` URL이 아니라
`https://api.bitbucket.org/2.0/repositories/<workspace>/<repo>/src/<ref>/catalog.json`을
사용하세요.

**예시 — Atlassian API 토큰(Basic 인증):**

```json
{
  "hosts": ["api.bitbucket.org"],
  "provider": "bitbucket",
  "auth": "basic",
  "username": "you@example.com",
  "token_env": "ATLASSIAN_API_TOKEN"
}
```

**예시 — Bitbucket Data Center HTTP 액세스 토큰:**

```json
{
  "hosts": ["bitbucket.example.com"],
  "provider": "bitbucket",
  "auth": "bearer",
  "token_env": "BITBUCKET_DC_TOKEN"
}
```

> **참고:** Bitbucket Cloud는 파일 다운로드(저장소의 **Downloads** 섹션)를 사전
> 서명된 Amazon S3 URL로의 리디렉션을 통해 제공합니다. 대상이 선언된 호스트를
> 벗어나므로 Specify는 그 리디렉션에서 `Authorization` 헤더를 제거합니다. 이는
> 예상된 동작이며, S3 URL이 자체 인가되므로 다운로드는 여전히 성공합니다. 인증
> 없는 마지막 구간도 무결성 검사를 받도록 카탈로그 항목에 `sha256`을 고정하세요.

## 여러 항목

서로 다른 호스트나 조직에 대해 여러 항목을 설정할 수 있습니다.

```json
{
  "providers": [
    {
      "hosts": ["github.com", "api.github.com", "raw.githubusercontent.com", "codeload.github.com"],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_TOKEN"
    },
    {
      "hosts": ["dev.azure.com"],
      "provider": "azure-devops",
      "auth": "basic-pat",
      "token_env": "AZURE_DEVOPS_PAT"
    }
  ]
}
```

## 동작 방식

공유 HTTP 헬퍼는 최초 요청 URL이 호스트 이름을 가진 HTTPS를 사용하도록
요구합니다. HTTP는 `localhost`, `127.0.0.1`, `::1` 같은 명백한 루프백 호스트에만
허용합니다. 카탈로그 소스에는 더 엄격한 정책이 있어, 루프백 호스트에서도 HTTPS를
요구합니다.
안전하지 않은 URL은 인증이 설정되지 않은 경우를 포함해, 자격 증명이 해석되거나
요청이 이루어지기 전에 거부됩니다.
온라인 번들 카탈로그 가져오기와 참조 검증은 URL 정책에 따른 거부를 패키지된
스냅샷이나 확인되지 않은 참조로의 가용성 폴백이 아니라 확정적인 오류로
취급합니다.

1. 나가는 HTTP 요청마다 URL 호스트 이름을 `auth.json`의 `hosts` 패턴과
   대조합니다.
2. 일치하는 항목이 있으면 해당 제공자가 토큰을 해석해 알맞은 `Authorization`
   헤더를 붙입니다. `github-cli` 획득 실패는 이후의 일치 항목을 시도하거나
   비인증 요청을 하지 않고 요청을 즉시 종료합니다.
3. 토큰 획득에 성공한 뒤 요청이 401 또는 403을 받으면 다음 일치 항목을
   시도합니다.
4. 획득 실패 없이 일치하는 항목을 모두 소진하면 마지막 폴백으로 비인증 요청을
   시도합니다.
5. 리디렉션 시 대상이 항목의 선언된 호스트를 벗어나면 `Authorization` 헤더가
   제거되어, CDN이나 서드파티 서비스로 자격 증명이 유출되는 것을 막습니다.

<a id="tls-certificates"></a>

## TLS 인증서

HTTPS 카탈로그 요청은 Python의 `urllib`과 `ssl` 인증서 검증을 사용합니다. 신뢰되는
인증 기관은 Python 배포판과 그 OpenSSL 설정에 따라 달라집니다. 개발용 CA를
브라우저나 macOS 키체인에서만 신뢰하도록 해도, Specify를 실행하는 Python
인터프리터가 반드시 그것을 신뢰하게 되지는 않습니다.

비공개 또는 로컬 개발 CA의 경우, Specify를 실행하기 전에 `SSL_CERT_FILE`을 그
CA를 포함하는 PEM CA 번들의 절대 경로로 설정하세요.

```bash
export SSL_CERT_FILE="/absolute/path/to/ca-bundle.pem"
```

이는 Python의 기본 CA 파일에 덧붙이는 것이 아니라 CA 파일을 선택하는 것입니다.
공개 카탈로그와 다운로드에 필요한 것을 포함해, 필요한 다른 신뢰 루트도 번들에
유지하세요. 서버 인증서는 여전히 요청된 호스트 이름이나 IP 주소에 대해 유효해야
합니다. 인증서 검증은 활성화된 상태로 유지하세요.

## 템플릿

GitHub가 미리 설정된 참고용 `auth.json`입니다.

```json
{
  "providers": [
    {
      "hosts": [
        "github.com",
        "api.github.com",
        "raw.githubusercontent.com",
        "codeload.github.com"
      ],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_TOKEN"
    }
  ]
}
```

사용하려면 다음과 같이 합니다.

```bash
mkdir -p ~/.specify
# 위의 JSON을 ~/.specify/auth.json에 복사
chmod 600 ~/.specify/auth.json
```
