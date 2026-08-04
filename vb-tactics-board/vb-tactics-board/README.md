# 배구 전술 보드 (Volleyball Tactics Board)

6인제·9인제 배구 전술을 시각적으로 만들고 팀별로 공유할 수 있는 웹 도구입니다.
정적 HTML 파일 하나(`index.html`)로 동작하며, 빌드 과정이 필요 없습니다.

## 주요 기능

- 6인제 / 9인제 로테이션 및 상황별(서브·리시브·공격·블로킹·수비) 기본 배치
- 선수·상대팀·교체선수·공을 자유롭게 드래그해서 포지셔닝
- 선수 방향(정면) 조정 — 토큰 테두리를 끌면 회전
- 전술 저장/불러오기/이름변경/삭제 (관리자만 저장·수정 가능)
- 팀 선택 화면 — 팀마다 비밀번호로 접근 제어, 관리자는 모든 팀 접근 가능
- 코트 확대/축소, 가로보기(90도 회전), 전체화면 보기
- 관리자 비밀번호 시스템 (기본 비밀번호: `coach1234`, 로그인 후 변경 가능)

## 데이터 저장 방식

이 앱은 아래 우선순위로 데이터를 저장합니다.

1. **Firebase (Firestore)** — 연결하면 기기·브라우저 관계없이 데이터가 공유됩니다. (권장)
2. **Claude 아티팩트 저장소** — Claude.ai 안에서 열었을 때만 자동으로 사용됩니다.
3. **localStorage** — 위 두 가지가 없을 때, 이 브라우저에만 저장됩니다 (기기 간 공유 안 됨).

### Firebase 연결 방법

1. [Firebase 콘솔](https://console.firebase.google.com)에서 프로젝트 생성
2. Firestore Database 생성 (테스트 모드로 시작)
3. Firestore 규칙을 아래와 같이 설정:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /vbBoardData/{document} {
         allow read, write: if true;
       }
     }
   }
   ```

   > ⚠️ 이 앱은 자체 서버 인증이 없어서 Firestore 규칙을 열어둬야 동작합니다. 내부 팀 도구용으로는 보통 문제없는 수준이지만, 설정값(firebaseConfig)을 아는 사람은 앱을 거치지 않고도 데이터에 접근할 수 있다는 점은 알아두세요.

4. 프로젝트 설정 → 웹 앱 추가 → `firebaseConfig` 객체 복사
5. 배포된 사이트에서 **관리자 로그인 → "Firebase 연결하기"** → 복사한 값 붙여넣기

## 로컬에서 열어보기

빌드 없이 `index.html`을 브라우저로 바로 열면 됩니다. 다만 파일을 더블클릭해서(`file://`) 열면 일부 브라우저에서 `localStorage`가 제한될 수 있으니, 가능하면 로컬 서버로 열어보는 걸 추천합니다.

```bash
npx serve .
# 또는
python3 -m http.server 8080
```

## Vercel로 배포하기

### 방법 A — 파일 직접 업로드
1. [vercel.com](https://vercel.com) 가입 (무료)
2. "Add New... → Project" → 이 폴더(또는 `index.html`)를 드래그 앤 드롭
3. Deploy

### 방법 B — GitHub 연동 (파일 수정할 때마다 자동 배포)
1. 이 저장소를 본인의 GitHub 계정으로 push (아래 "GitHub에 올리기" 참고)
2. Vercel 대시보드 → "Add New... → Project" → "Import Git Repository"
3. 방금 올린 저장소 선택 → Deploy
4. 이후로는 GitHub에 `git push`만 하면 Vercel이 자동으로 재배포합니다

## GitHub에 올리기

```bash
# 1. GitHub에서 새 저장소를 먼저 만든 뒤 (github.com/new), 아래 URL을 본인 것으로 바꿔서 실행
git remote add origin https://github.com/사용자명/저장소이름.git
git branch -M main
git push -u origin main
```

## 관리자 비밀번호

- 팀 접근 기본 비밀번호: `1234` (팀별로 변경 가능)
- 관리자 기본 비밀번호: `coach1234` (로그인 후 반드시 변경 권장)

## 주의사항

이 앱은 완전한 클라이언트 사이드 앱입니다. 관리자 권한 검증도 브라우저 안에서 이루어지므로, 은행 수준의 보안이 필요한 데이터에는 적합하지 않습니다. 학교·동호회 등 내부 코칭 도구로 사용하는 것을 전제로 설계되었습니다.
