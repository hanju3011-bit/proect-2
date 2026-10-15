# Firebase Google 로그인 전환 계획

## 구현 방향

신규 가입 경로를 Firebase Authentication의 Google 로그인이 담당하도록 바꾼다. Firebase JavaScript SDK가 Google 팝업 또는 redirect 인증을 수행하고, 브라우저가 받은 Firebase ID token을 서버의 tRPC mutation으로 전달한다. 서버는 Firebase 프로젝트 ID에 맞춰 Google 공개 키와 ID token 서명·만료·issuer·audience를 검증한 후 Firebase UID에 연결된 앱 사용자와 기존 30일 앱 세션을 발급한다. DB에는 Firebase UID용 nullable 고유키를 추가하는 비파괴 migration을 적용한다. Firebase ID token은 애플리케이션 세션 쿠키나 사용자 입력값으로 신뢰하지 않는다.

기존 이메일·비밀번호 계정은 계속 로그인할 수 있게 유지한다. 신규 이메일·비밀번호 등록 UI와 공개 API는 제거한다. 기존 계정은 이메일 일치만으로 Firebase 계정에 자동 병합하지 않는다. Firebase 웹 앱 설정(apiKey, authDomain, projectId, appId)은 서버의 명시적 공개 설정 응답에서 필요한 값만 브라우저로 전달한다. 비공개 서비스 계정 키는 사용하지 않는다. Firebase 설정이 없으면 오류 원인을 명확하게 보여 주고, `Failed to fetch`처럼 모호하게 처리하지 않는다.

Firebase Authentication에서 Google 공급자를 켜고 앱을 Authorized domains에 추가하는 작업은 Firebase 프로젝트 소유자가 콘솔에서 해야 한다. 구현 코드는 이 사용자의 Firebase 구성값을 받기 전에도 빌드할 수 있도록 준비하되, 실제 Google 로그인을 성공했다고 주장하지 않는다.

## 구성 및 주요 모듈

- `client/src/lib/firebase-auth.ts`: 공개 Firebase 설정을 이용해 SDK를 초기화하고 Google 팝업/redirect, redirect 결과, sign-out을 캡슐화한다.
- `client/src/pages/AuthPage.tsx`: 새 가입의 Google 우선 흐름과 기존 사용자의 이메일·비밀번호 로그인 경로를 제공한다.
- `server/services/firebase-auth-service.ts`: Firebase ID token을 검증하고 앱 세션을 발급한다.
- `server/repositories/user-repository.ts`: 검증된 Firebase UID에 연결한 사용자를 조회·생성한다. 기존 프로필은 재로그인 시 덮어쓰지 않는다.
- `server/routers.ts` 및 `server/_core/publicConfig.ts`: 검증 mutation과 제한된 공개 설정 응답을 연결한다.
- `drizzle/schema.ts` 및 `drizzle/` migration: nullable Firebase UID 컬럼과 고유 인덱스를 추가하며 기존 계정 행과 소비 데이터를 변경·삭제하지 않는다.
- `client/src/platform.d.ts`: 브라우저 공개 설정 타입을 정의한다.

## 디자인

- **Design Movement:** 기존 앱의 다크 에디토리얼 웰니스 디자인을 유지하고, Google 인증 버튼은 익숙하고 명확한 서비스 로그인 관례로 표현한다.
- **Core Principles:** 새 가입은 한 번의 명확한 선택으로 완료한다. 기존 사용자 접근을 보존한다. 인증 실패는 실행 가능한 원인으로 설명한다. 외부 identity와 앱 내부 세션을 분리한다.
- **Color Philosophy:** 차콜 배경은 소비 기록을 차분히 바라보게 하고, 기존 만끽 그린은 주요 행동을 구분한다. Google 버튼은 브랜드별 컬러를 덧씌우지 않고 원래의 중립적 구성을 사용한다.
- **Layout Paradigm:** 현재 인증 페이지의 좌측 브랜드 스토리와 우측 행동 패널 레이아웃을 유지하며 모바일에서는 세로 흐름으로 접는다.
- **Signature Elements:** 기존 심볼 락업, 얇은 차콜 경계선, 초록색 진행·행동 강조를 계속 사용한다.
- **Interaction Philosophy:** 로그인 공급자와 기존 사용자 로그인 선택을 명시한다. 성공 시 현재 계정으로 이동하고, 취소·도메인 미허용·설정 미완료는 한국어 안내로 분리한다.
- **Animation:** 기존의 절제된 로딩 인디케이터만 사용한다. 팝업 대기 중 버튼 spinner를 표시하고, 성공 화면에는 과한 전환 효과를 쓰지 않는다.
- **Typography System:** 현재 한글 UI 서체 계층과 화면 제목/보조 설명/버튼의 위계를 유지한다. Google 문구는 일반 본문 크기로 읽기 쉽게 둔다.
- **Brand Essence:** 친구와 함께 소비 습관을 만드는 사람이 안전하게 기록을 이어가는 서비스. 성격은 다정하고, 명료하며, 믿음직하다.
- **Brand Voice:** 차분하고 부담이 적은 한국어를 쓴다. 예: “Google 계정으로 만끽을 시작해요.”, “기존 계정은 이메일로 로그인할 수 있어요.”
- **Wordmark & Logo:** 제공된 왕관·이니셜 심볼과 기존 만끽 워드마크를 유지한다.
- **Signature Brand Color:** 기존 만끽의 비비드 그린 `#00E676`을 계속 사용한다.

## 의존성과 제공 방식

기존 React/Vite·Express·tRPC·Drizzle/MySQL 구조에 `firebase` 웹 SDK를 추가하고, 이미 제공된 `jose`를 서버 ID-token 검증에 재사용한다. 애플리케이션은 기존 서버(포트 3000)와 DB에서 유지된다. 로그인 쿠키는 Preview의 cross-site HTTPS 임베딩을 위해 `SameSite=None; Secure` 옵션을 유지한다. 실제 로그인 활성화에는 Firebase 웹 앱의 네 공개 설정 필드, Firebase Console의 Google provider 활성화, 현재 Preview 도메인의 authorized-domain 등록이 필요하다.
