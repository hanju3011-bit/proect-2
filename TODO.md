# 만끽 구현 진행 항목

## 계정·프로필
- [x] 기존 이메일·비밀번호 계정 사용자는 비밀번호를 안전한 해시로 저장한 상태로 계속 로그인할 수 있으며 애플리케이션 세션으로 인증한다.
- [x] Firebase Google 계정은 검증된 Firebase UID로만 연결하며, 이메일이 같은 기존 이메일·비밀번호 계정과 자동 병합하지 않는다.
- [ ] 신규 가입은 Firebase Authentication의 Google 로그인을 사용한다. Firebase JavaScript SDK로 Google 팝업 또는 리다이렉트 인증을 수행하고, 이메일·비밀번호 신규 가입 UI와 공개 가입 API는 제공하지 않는다. Manus OAuth 계정 생성 콜백과 Preview identity fallback도 앱 인증 경로로 등록하지 않는다.
- [ ] 서버는 브라우저가 보낸 Firebase ID token의 서명, 만료, Firebase 프로젝트의 issuer와 audience를 검증한 후에만 사용자 계정과 기존 앱 세션을 발급한다. Firebase UID는 DB 고유 키로 연결하고 기존 이메일·비밀번호 계정과 이메일 일치만으로 자동 병합하지 않는다.
- [ ] Firebase의 apiKey, authDomain, projectId, appId만 브라우저 공개 설정으로 전달한다. Firebase 비공개 서비스 계정 키는 사용하지 않으며 설정이 누락되면 명확한 설정 안내를 표시한다.
- [ ] 기존 이메일·비밀번호 로그인은 기존 계정 사용자용으로 계속 접근 가능하고, 새 계정 생성은 Google 로그인으로만 시작한다.
- [x] 사용자는 닉네임과 프로필 사진을 설정·변경할 수 있다.
- [x] `User(id,email,password_hash,nickname,profile_img)` 데이터를 관리하고, `User` 도메인 모델에 등록·프로필 검증 규칙을 둔다.

## 목표 관리
- [x] 목표를 생성할 때 목표 이름, 카테고리, 목표 금액을 설정한다.
- [x] 목표 시작일·종료일을 지정하거나, 종료일 없이 상시 관리하는 무제한 기간 모드를 설정한다.
- [x] 현재 소비 금액을 목표 금액과 비교하여 초록색 프로그레스 바와 퍼센트로 표시한다.
- [x] `Goal(id,user_id,title,target_amount,current_amount,is_unlimited,start_date,end_date)` 데이터를 저장하고, `Goal` 도메인 모델이 기간 및 진행률 규칙을 관리한다.

## 소비 기록·사진
- [x] 소비 기록에 금액, 카테고리, 메모, 지출일을 입력한다.
- [x] 영수증 또는 구매 물품·절약 순간을 보여주는 JPG, PNG, WebP 사진을 첨부한다.
- [x] 기록에 연결한 목표가 있으면 기록 저장 트랜잭션에서 목표 누적 소비액을 갱신한다.
- [x] `Record(id,user_id,goal_id,amount,category,memo,image_url,created_at)` 데이터를 저장하고, `ConsumptionRecord` 도메인 모델이 기록 필드를 검증한다.
- [x] 개인 소비 기록은 오래된 항목도 50건 단위 페이지네이션으로 탐색할 수 있다.

## 친구·소셜 피드
- [x] 닉네임 또는 유저 ID로 사용자를 검색하고 친구 신청을 보낸다.
- [x] 받은 친구 신청을 승인하거나 거절한다.
- [x] 친구가 올린 사진과 소비 기록을 공유 피드에서 확인하며, 피드는 10초 간격으로 갱신된다.
- [x] 친구 기록에 좋아요/피드백 반응과 응원 댓글을 남긴다.
- [x] `Friendship(id,requester_id,receiver_id,status)`와 `Interaction(id,record_id,user_id,type,comment)` 데이터를 저장한다.
- [x] `Friend`, `Friendship`, `Interaction` 도메인 모델이 관계·상호작용 규칙을 관리하고, 좋아요는 DB 유일 키로 사용자별 중복을 방지한다.

## 객체지향 구조·제품 경험
- [x] User, Goal, Record, Friendship, Interaction 및 세션 데이터를 관계형 DB에 저장하고 도메인·서비스·저장소 계층으로 분리한다.
- [x] 배경 #0D0D0D/#121212 계열, 다크 그레이 카드, 비비드 그린 #00E676 포인트와 만끽 심볼을 적용한다.
- [x] 모바일 우선 반응형 카드 UI를 제공한다.
