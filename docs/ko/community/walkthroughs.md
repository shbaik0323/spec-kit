# 커뮤니티 워크스루

> [!NOTE]
> 커뮤니티 워크스루는 각 작성자가 독립적으로 만들고 유지 관리합니다. GitHub가 **검토하거나, 보증하거나, 지원하지 않습니다**. 따라 하기 전에 내용을 검토하고 여러분의 판단에 따라 사용하세요.

커뮤니티가 기여한 이 워크스루들로 다양한 시나리오에서 스펙 주도 개발이 실제로 동작하는 모습을 살펴보세요. 완료된 흐름을 보여 주는 유용한 읽기 전용 예시이지만, 생성되는 스펙, 계획, 태스크의 공식적인 모범 출력은 아닙니다.

- **[그린필드 .NET CLI 도구](https://github.com/mnriem/spec-kit-dotnet-cli-demo)** — 빈 디렉터리에서 Timezone Utility를 .NET 단일 바이너리 CLI 도구로 만들며, GitHub Copilot 에이전트를 사용해 constitution, specify, plan, tasks, 여러 차례의 implement까지 spec-kit 워크플로 전체를 다룹니다.

- **[그린필드 Spring Boot + React 플랫폼](https://github.com/mnriem/spec-kit-spring-react-demo)** — Spring Boot, 내장 React, PostgreSQL, Docker Compose를 사용해 LLM 성능 분석 플랫폼(REST API, 그래프, 반복 추적)을 처음부터 만들며, clarify 단계와 아티팩트 간 일관성 분석 패스를 포함합니다.

- **[브라운필드 ASP.NET CMS 확장](https://github.com/mnriem/spec-kit-aspnet-brownfield-demo)** — 기존 오픈소스 .NET CMS(CarrotCakeCMS-Core, C#, Razor, SQL, JavaScript, 설정 파일 약 307,000줄)에 크로스 플랫폼 Docker Compose 인프라와 토큰 인증 헤드리스 REST API라는 두 가지 새 기능을 추가하며, 기존 스펙이나 컨스티튜션이 없는 기존 코드베이스에 spec-kit이 어떻게 들어맞는지 보여 줍니다.

- **[브라운필드 Java 런타임 확장](https://github.com/mnriem/spec-kit-java-brownfield-demo)** — 기존 오픈소스 Jakarta EE 런타임(Piranha, 180개 Maven 모듈에 걸친 Java, XML, JSP, HTML, 설정 파일 약 420,000줄)에 비밀번호로 보호되는 Server Admin Console을 추가하며, 기존 스펙이나 컨스티튜션이 없는 대규모 다중 모듈 Java 프로젝트에서의 spec-kit을 보여 줍니다.

- **[브라운필드 Go / React 대시보드 데모](https://github.com/mnriem/spec-kit-go-brownfield-demo)** — **GitHub Copilot CLI를 사용해 터미널에서만** 구동되는 spec-kit을 보여 줍니다. NASA의 오픈소스 Hermes 지상 지원 시스템(Go)에 가벼운 React 기반 웹 텔레메트리 대시보드를 추가하며, constitution → specify → plan → tasks → implement 워크플로 전체가 터미널에서 동작함을 보여 줍니다.

- **[사용자 정의 프리셋을 쓴 그린필드 Spring Boot MVC](https://github.com/mnriem/spec-kit-pirate-speak-preset-demo)** — 사용자 정의 해적 말투 프리셋으로 Spring Boot MVC 애플리케이션을 처음부터 만들며, 프리셋이 spec-kit 경험 전체를 어떻게 바꿀 수 있는지 보여 줍니다. 명세는 "Voyage Manifests"가, 계획은 "Battle Plans"가, 태스크는 "Crew Assignments"가 되며, 도구를 하나도 바꾸지 않고 모두 해적 말투로 생성됩니다.

- **[사용자 정의 확장을 쓴 그린필드 Spring Boot + React](https://github.com/mnriem/spec-kit-aide-extension-demo)** — spec-kit에 대안적인 스펙 주도 워크플로를 추가하는 커뮤니티 확장인 **AIDE 확장**을 안내합니다. 상위 수준 스펙(비전)과 하위 수준 스펙(작업 항목)을 7단계 반복 라이프사이클(비전 → 로드맵 → 진행 추적 → 작업 큐 → 작업 항목 → 실행 → 피드백 루프)로 구성합니다. 가족 거래 플랫폼(Spring Boot 4, React 19, PostgreSQL, Docker Compose)을 시나리오로 사용해, 확장 메커니즘이 코어 도구를 바꾸지 않고도 다른 스타일의 스펙 주도 개발을 끼워 넣게 해 준다는 것, 즉 Spec Kit의 "Kit"을 제대로 활용하는 모습을 보여 줍니다.
