# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 성격

스프링 MVC 학습용 실습 리포지터리다. 강의 진도에 따라 커밋이 하나씩 쌓이며, 커밋 메시지 제목은 강의 섹션 번호와 제목을 그대로 쓴다 (예: `feat: 70. RedirectAttributes`). 프로덕션 코드가 아니라 **단계별 학습 예제**이므로, 학습 흐름과 무관한 리팩터링이나 "개선"을 임의로 제안하지 않는다.

## 명령어

Gradle 래퍼를 사용한다 (Java 25 toolchain, Spring Boot 4.1.1).

```bash
./gradlew bootRun            # 앱 실행 (http://localhost:8080)
./gradlew build              # 빌드 + 전체 테스트
./gradlew test               # 전체 테스트
./gradlew test --tests 'hello.itemservice.domain.item.ItemRepositoryTest'          # 클래스 단위
./gradlew test --tests 'hello.itemservice.domain.item.ItemRepositoryTest.save'     # 메서드 단위
```

## 아키텍처

`hello.itemservice` 아래 3계층:

- `domain.item.Item` — Lombok `@Data` POJO. `id`는 저장 시점에 리포지터리가 채운다.
- `domain.item.ItemRepository` — `@Repository`, `static Map<Long, Item>` + `static long sequence` 기반 메모리 저장소. 학습 단계라 동시성 처리는 없다(실무라면 `ConcurrentHashMap`/`AtomicLong`). `clearStore()`는 테스트 전용.
- `web.basic.BasicItemController` — `@RequestMapping("/basic/items")`. `@PostConstruct init()`에서 테스트 데이터 2건을 넣는다.

### 컨트롤러의 Vn 메서드 규칙

`addItemV1` ~ `addItemV6`처럼 같은 기능을 파라미터 바인딩 방식별로 여러 버전 보관한다 (`@RequestParam` → `@ModelAttribute` → 생략 → PRG → `RedirectAttributes`). **활성 버전 하나만 `@PostMapping`이 살아 있고 나머지는 주석 처리된다.** 다음 단계로 넘어갈 때는 이전 버전을 지우지 말고 매핑 애너테이션만 주석 처리한 뒤 새 버전에 붙인다.

### 템플릿 vs 정적 HTML

같은 화면이 두 벌 존재하며 둘 다 유지한다:

- `src/main/resources/static/html/*.html` — 순수 HTML 목업. 서버 없이 열어보는 용도.
- `src/main/resources/templates/basic/*.html` — 타임리프 버전. 실제 렌더링 대상.
- 부트스트랩 CSS도 `static/css/`와 `templates/css/` 양쪽에 있다.

타임리프 템플릿은 `th:href="@{...}"`와 `href="../css/..."`를 함께 쓰는 **내추럴 템플릿** 방식을 지킨다 — 서버 렌더링 시엔 `th:*`가, 파일로 직접 열면 순수 HTML 속성이 동작한다.

## 커밋

전역 지침의 Conventional Commits를 따르되, description은 학습 목차의 번호와 제목을 그대로 사용한다: `feat: 68. 상품 수정`.
