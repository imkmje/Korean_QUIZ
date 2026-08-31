# 맞춤법 퀴즈 (카훗식) — 설정 가이드 (Firebase 버전)

## 구성
- `teacher.html` — 교실 화면(프로젝터)에 띄우는 파일. PIN으로 보호됨.
- `student.html` — 학생이 QR로 접속해 답을 제출하는 파일.
- `firestore.rules` — Firebase 콘솔에 붙여넣을 보안 규칙.

## 1. Firebase 프로젝트 준비
1. [console.firebase.google.com](https://console.firebase.google.com)에서 새 프로젝트를 만듭니다. (무료 Spark 플랜으로 충분합니다.)
2. 왼쪽 메뉴 **빌드 → Firestore Database** → "데이터베이스 만들기" → 위치는 `asia-northeast3(서울)` 추천 → 일단 "테스트 모드"로 시작해도 되고, 아래 3번에서 규칙을 바로 교체해도 됩니다.
3. **Firestore Database → 규칙(Rules)** 탭에서 `firestore.rules` 내용 전체를 붙여넣고 "게시".
4. 프로젝트 설정(톱니바퀴 아이콘) → **일반** 탭 맨 아래 "내 앱" → 웹 앱(`</>`) 추가 → 앱 닉네임 아무거나 입력 → **Firebase SDK 구성 정보**(firebaseConfig 객체)를 복사해둡니다.

## 2. 파일 설정값 수정
`teacher.html`과 `student.html` 맨 위 `<script>` 블록에서 `window.FIREBASE_CONFIG`를 1-4번에서 복사한 값으로 바꿔주세요.

```js
window.FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

`teacher.html`에는 추가로 이 두 값도 있습니다.

```js
window.TEACHER_PIN = "0802";                          // 원하는 숫자로 변경
window.STUDENT_PAGE_URL = "https://imkmje.github.io/Korean_QUIZ/student.html"; // 배포 후 실제 주소로 변경
```

**두 파일의 `FIREBASE_CONFIG`는 반드시 똑같아야 합니다.**

## 3. GitHub Pages 배포
1. 저장소에 `teacher.html`, `student.html`을 올립니다. (`firestore.rules`는 배포 안 해도 됨, 참고용)
2. Settings → Pages에서 배포를 켭니다.
3. 배포된 student.html 주소를 `teacher.html`의 `STUDENT_PAGE_URL`에 반영하고 다시 커밋하세요.

## 4. 사용 방법

### 교사
1. 교실 프로젝터에서 `teacher.html` 접속 → PIN 입력.
2. **참여 방식** 선택:
   - **개인 참여 모드**: 학생이 각자 폰으로 QR을 찍어 참여, 점수·순위 집계.
   - **다같이 풀이 모드**: 학생 기기 필요 없음. 문제만 화면에 띄우고 교사가 넘기며 다같이 풀이. 점수 집계 안 함.
3. **문제 진행 방식** 선택: 수동(버튼으로 직접 넘김) 또는 자동(제한 시간 후 자동 진행). 여기서 정하는 "문항당 기준 시간"은 자동 진행의 실제 제한 시간이자, 수동 진행에서도 빠른 정답 가산점 계산의 기준으로 쓰입니다.
4. 문제 목록(JSON)은 기본 25문항이 채워져 있습니다. 그대로 써도 되고 직접 수정해도 됩니다.
5. "세션 만들기" → 개인 참여 모드면 QR/방 코드가 뜹니다. 학생이 들어올 때마다 닉네임이 실시간으로 표시됩니다. 다 들어오면 "퀴즈 시작".
6. 문제 화면에서는 학생들이 몇 명 답변했는지, 누가 답변을 마쳤는지 실시간으로 표시됩니다. "정답 공개"를 누르면 정답·득표 현황과 함께 현재 스코어보드가 뜨고, 확인 후 "다음 문제"로 넘어갑니다.

### 학생
1. QR을 찍거나 방 코드 6자리를 직접 입력하고 닉네임 입력 후 참여.
2. 교실 화면과 같은 문제가 뜨면 답을 선택 → 제출.
3. 정답 공개 시 O/X 도장으로 정오답과 이번 문제 획득 점수·현재 점수가 표시됩니다.

### 점수 계산 (카훗식 — 빠를수록 고득점)
- 정답을 맞히면 **1000점(즉시 정답)~500점(기준 시간을 거의 다 써서 정답)** 사이에서 답변 속도에 비례해 차등 지급됩니다. 오답은 0점.
- 기준 시간은 세션 설정의 "문항당 기준 시간(초)" 값이며, 수동 진행에서도 동일하게 적용됩니다.
- 클라이언트에서 계산하는 방식이라 완벽한 부정행위 방지는 아닙니다 (아래 "알아두실 점" 참고).

## 5. 알아두실 점 (솔직한 안내)
- `TEACHER_PIN`은 화면 접근을 막는 정도의 **가벼운 보호**입니다. `firestore.rules`도 교사/학생을 구분하지 않고 열려 있어서, 학생이 브라우저 개발자도구를 열어 Firestore에 직접 접근하는 것까지 막지는 못합니다. 성적 반영용 평가에는 쓰지 마시고, 수업 중 활동용으로만 사용하세요.
- Firebase Spark(무료) 플랜은 Firestore 기준 **하루 read 5만 회 / write 2만 회 / 삭제 2만 회**까지 무료입니다. 반 하나(약 30명) 기준 25문항 게임 1회면 대략 write 1,000회 안팎이라 여유가 큽니다. 다만 하루에 여러 반을 연속으로 돌리시면 한 번씩 Firebase 콘솔의 사용량(Usage) 탭을 확인해보시길 권합니다.
- 학생 답변은 문서 ID를 "플레이어ID_문항번호"로 고정해뒀기 때문에, 같은 문제에 두 번 제출해도 마지막 값으로만 덮어써집니다(중복 걱정 없음). Supabase 버전에서 썼던 unique 제약과 동일한 효과입니다.
- 문제 수정은 매번 `teacher.html`의 JSON 텍스트 영역에서 직접 하시면 됩니다. 문제은행 UI는 아직 없습니다.

## 6. 배포 전 테스트 체크리스트
- [ ] 본인 폰 + 노트북(교사 화면) 2개 기기로 먼저 테스트
- [ ] QR 스캔 → 참여 → 답 제출 → 정답 공개까지 한 바퀴 확인
- [ ] 자동 진행 모드 쓸 경우 제한 시간이 너무 짧지 않은지 확인
- [ ] Wi-Fi가 불안정한 교실이면 수동 진행 + 넉넉한 시간 권장
- [ ] Firebase 콘솔에서 Firestore에 `quiz_sessions` 컬렉션이 실제로 생기는지 확인
