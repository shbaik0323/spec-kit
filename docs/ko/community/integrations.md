# 커뮤니티 통합

커뮤니티 통합은 독립적으로 유지 관리되는 외부 어댑터를 통해 Spec Kit을 추가
코딩 에이전트에 연결합니다.

> [!NOTE]
> 메인테이너는 어댑터 구현이 아니라 카탈로그 항목 메타데이터를 확인합니다.
> 등재는 코드 감사, 보증, 지원 약속, 설치 허가가 아닙니다. 설치하고 사용하기 전에
> 어댑터의 소스와 릴리스를 검토하세요.

## 사용 가능한 통합

[커뮤니티 통합 카탈로그](https://github.com/github/spec-kit/blob/main/integrations/catalog.community.json)에는
현재 항목이 없습니다. 내장 통합은 CLI와 함께 제공되며 통합 목록에 따로
나열됩니다.

프로젝트에서 활성인 소스를 확인하고 그 메타데이터를 검색하세요.

```bash
specify integration catalog list
specify integration search
specify integration info <id> --versions
```

검색은 커뮤니티 소스만이 아니라 활성 카탈로그 설정을 사용합니다. 기본 커뮤니티
소스는 탐색 전용입니다. 외부 어댑터를 설치하려면 명시적으로 설치가 허용된
카탈로그와 접근 가능한 패키지 아카이브가 필요합니다. 직접 URL이나 로컬 설치
경로는 없습니다.

## 통합 기여하기

외부 어댑터를 개발하고 테스트하고, 그 디스크립터와 고정된 릴리스 아카이브를 게시한
다음, `integrations/catalog.community.json`을 편집하는 카탈로그 PR을 제출하세요.
패키지 계약, 필수 메타데이터, 검증 근거, 버전 업데이트는 통합 기여 가이드를
따르세요. 내장 통합 기여는 그 가이드의 별도 경로를 따릅니다.

## 함께 보기

- [통합 목록](../integrations/index.md)
- [외부 어댑터 설치](../integrations/cli.md#catalog-installed-external-adapters)
- [카탈로그 스키마와 설정](../integrations/catalog.md)
- [통합 개발 및 제출](../integrations/contributing.md)
- [카탈로그 탐색과 신뢰](../overview/catalogs.md)
