# 🏠 부동산 계약서 AI 분석

등기부등본·임대차계약서 등 부동산 서류를 AI가 분석해 **위험 요소를 점검**하고, **실거래가 기반 깡통전세 판정**을 해주며, **워드(.docx) 임시 계약서를 자동 생성**해주는 단일 HTML 웹앱입니다.

Claude API(Anthropic)를 브라우저에서 직접 호출하는 클라이언트 사이드 앱으로, 별도 백엔드 없이 정적 호스팅만으로 동작합니다.

---

## ✨ 주요 기능

### 1. 계약서 AI 분석
- **서류 업로드** — PDF / JPG / PNG 다중 업로드 지원 (등기부등본, 건축물대장, 임대차계약서, 현장 사진 등)
- **5MB 초과 이미지 자동 압축** — Canvas API로 리사이즈(최대 2000px) + JPEG 품질 점진 하향
- **계약 조건 점수화** — 임대인 신뢰도 / 권리 안전성 / 계약 조건 / 특약 완성도 / 서류 완비 5개 항목
- **실질 비용 계산** — 보증금, 월세, 관리비, 공과금, 전세보증보험료 포함 월 부담액
- **특약 패턴 분석** — 계약서 내 특약을 위험/주의/안전 3단계로 분류
- **유사 분쟁 케이스** — 판례나 유사 사례를 기반으로 한 주의점 제시

### 2. 실거래가 웹서치 기반 깡통전세 판정
- 서류에서 주소·건물명·면적을 추출 후, Claude의 **web_search 도구**로 국토부 실거래가·네이버부동산 조회
- **전세가율** 계산: (전세금 + 선순위채권) / 매매가
  - 80% 이상 🔴 위험 / 70~80% 🟡 주의 / 70% 미만 🟢 안전
- 조회된 실거래 내역 테이블로 표시

### 3. 상황별 자동 특약 생성
- 10가지 프리셋 태그(반려동물, 전세대출, 외국인, 재건축 우려 등) + 자유 입력
- 상황에 맞는 법적 특약 문장을 AI가 생성, 클립보드 복사 지원

### 4. 🆕 워드(.docx) 임시 계약서 자동 생성
- **템플릿 내장** — GitHub 레포의 `.docx` 파일을 raw URL로 자동 다운로드
- **빈칸 자동 채우기** — AI가 템플릿 내부 XML을 분석해 임대인/임차인 정보, 금액, 날짜 등을 적절한 위치에 삽입
- **필수 특약 누락 감지 & 자동 추가** — 아래 5개 특약이 기존 계약서에 없으면 자동 보완
  1. 근저당 고지 (국세 체납, 근저당 이자, 전세대출 협조)
  2. 매매 계약 고지 (소유권 변동 사전 통보)
  3. 임대인 계좌 (국민 김홍식 99290417067)
  4. 기타 사항 (주임법·민법·개인정보)
  5. 입주 전 수리 관련 (하자 vs 소모품)

---

## 🗂️ 프로젝트 구조

```
.
├── 부동산계약서AI분석.html   ← 메인 앱 (단일 파일, 모든 로직 포함)
├── templates/
│   └── lease-template.docx   ← 계약서 템플릿 (빈칸/밑줄 형태)
└── README.md
```

---

## 🚀 설치 & 배포

### 1. GitHub에 템플릿 업로드
레포를 만들고 `templates/lease-template.docx` 경로에 **빈칸이 있는 계약서 양식**을 올립니다.

### 2. HTML 내 `TEMPLATE_URL` 수정
`부동산계약서AI분석.html` 파일 상단에서:

```javascript
const TEMPLATE_URL = 'https://raw.githubusercontent.com/YOUR_GITHUB_ID/YOUR_REPO/main/templates/lease-template.docx';
```

`YOUR_GITHUB_ID`와 `YOUR_REPO`를 실제 값으로 교체하세요.

### 3. GitHub Pages 배포 (권장)
- 레포 **Settings → Pages**
- Source: `main` 브랜치 / root 선택
- 몇 분 후 `https://YOUR_ID.github.io/YOUR_REPO/부동산계약서AI분석.html`에서 접속 가능

### 4. Anthropic API 준비
- [console.anthropic.com](https://console.anthropic.com) 가입
- **Plans & Billing**에서 **최소 $5 크레딧 충전** (⚠️ Claude.ai Pro 월정액과는 별개 과금!)
- **Settings → API Keys**에서 키 발급 (`sk-ant-api03-...`)

---

## 🎯 사용법

1. 배포된 URL 또는 로컬서버(아래 참고)로 접속
2. **Anthropic API Key** 입력 — 브라우저 메모리에만 저장되며 외부로 전송되지 않음
3. **서류 업로드** — 등기부등본, 계약서, 사진 등을 드래그&드롭 또는 클릭
4. **계약 유형** 선택 (월세/전세) — 필수 특약 매칭에 사용
5. **건물 종류** 선택 (아파트/연립다세대/오피스텔/단독다가구) — 실거래가 조회 정확도 향상
6. (선택) **자동 특약 생성** — 상황 태그 선택 후 생성
7. **🔍 AI 분석 시작** 버튼 클릭
8. 분석 완료 후 결과 카드 상단의 **📄 임시 계약서 다운로드** 버튼으로 워드 파일 생성

---

## 🔧 로컬 실행 (개발용)

⚠️ **`file:///` 경로로 HTML을 직접 열면 CORS로 API 호출이 차단됩니다.** 반드시 웹서버를 거쳐야 합니다.

### 방법 A — VS Code Live Server (추천)
1. VS Code에 `Live Server` 확장 설치
2. HTML 파일 우클릭 → **"Open with Live Server"**
3. `http://127.0.0.1:5500/부동산계약서AI분석.html`로 열림

### 방법 B — Python 간단 서버
```bash
cd 프로젝트폴더
python -m http.server 8000
# → http://localhost:8000/부동산계약서AI분석.html
```

### 방법 C — Node.js
```bash
npx http-server -p 8000
```

---

## 🛠️ 기술 스택

| 분류 | 기술 |
|---|---|
| **프론트엔드** | HTML + CSS + Vanilla JS (프레임워크 X) |
| **LLM** | Claude Sonnet 4 (`claude-sonnet-4-20250514`) via Anthropic Messages API |
| **웹서치** | Anthropic `web_search_20250305` 도구 (실거래가 조회) |
| **파일 처리** | JSZip 3.10.1 (.docx 압축 해제/재압축), FileSaver.js 2.0.5 (다운로드) |
| **이미지 압축** | Canvas API (리사이즈 + JPEG 품질 조절) |
| **폰트** | Noto Sans KR |

---

## ⚙️ 내부 동작 원리

### 분석 플로우
```
[파일 업로드]
    ↓
[이미지 자동 압축 (5MB 초과 시)]
    ↓
[STEP 1] Claude에게 서류 전송 → 주소/임대인/금액 등 JSON 추출
    ↓
[STEP 2] 웹서치 도구로 국토부 실거래가 조회 (agentic loop, 최대 5회 반복)
    ↓
[STEP 3] 전세가율 계산 + 깡통전세 판정
    ↓
[STEP 4] 종합 분석 → 점수 / 위험 요소 / 특약 / 조언 JSON 생성
    ↓
[결과 렌더링]
```

### 계약서 생성 플로우
```
[GitHub raw URL에서 템플릿 .docx fetch]
    ↓
[JSZip으로 압축 해제 → word/document.xml 추출]
    ↓
[AI에게 원본 XML + 추출 데이터 + 누락 특약 전달]
    ↓
[AI가 XML 내 빈칸 판단 + 특약 <w:p> 문단 삽입]
    ↓
[수정된 XML을 다시 zip에 삽입 → .docx 재압축]
    ↓
[FileSaver로 다운로드 트리거]
```

---

## 🐛 문제 해결 (Troubleshooting)

### `image exceeds 5 MB maximum` 에러
✅ **해결됨** — 업로드 시 자동 리사이즈/압축됩니다. 파일 리스트에 "📦 압축 예정" 뱃지가 표시되면 정상.

### `Failed to fetch` / `401 Unauthorized` 에러
**원인**: API 키 또는 크레딧 문제

확인 순서:
1. `console.anthropic.com` → **Plans & Billing**에서 **크레딧 잔액** 확인
   - ⚠️ **Claude.ai Pro 구독과 API는 별개 과금**입니다. Pro 구독 중이어도 API 크레딧이 $0이면 401이 뜹니다
   - 최소 $5 충전 필요 (Tier 1 진입)
2. API 키를 다시 발급받아 앞뒤 공백 없이 붙여넣기
3. PowerShell에서 키 단독 테스트:
   ```powershell
   curl.exe https://api.anthropic.com/v1/messages `
     -H "x-api-key: sk-ant-api03-..." `
     -H "anthropic-version: 2023-06-01" `
     -H "content-type: application/json" `
     -d '{\"model\":\"claude-haiku-4-5-20251001\",\"max_tokens\":10,\"messages\":[{\"role\":\"user\",\"content\":\"hi\"}]}'
   ```
   - 200 OK → 키 정상
   - 401 → 크레딧/키 문제
   - 400 "credit balance too low" → 크레딧 충전

### `Access to fetch ... has been blocked by CORS policy` 에러
**원인**: `file:///` 경로로 HTML을 직접 열어서 발생

**해결**: 위 "로컬 실행" 섹션의 웹서버 방법 중 하나 사용. GitHub Pages로 배포했다면 자동 해결.

### 템플릿 로드 실패
- 상단 **템플릿 상태 표시**가 빨간색이면 `TEMPLATE_URL` 확인
- GitHub raw URL이 정확한지, 파일이 public 레포에 있는지 확인
- 네트워크 탭에서 템플릿 fetch 응답 코드 확인 (404면 경로 문제)

### .docx 생성 시 XML 손상
- AI가 가끔 XML 구조를 미세하게 망가뜨려 Word에서 안 열리는 경우가 있음
- 재시도하거나, 템플릿에 `{{임대인명}}` 같은 명시적 플레이스홀더를 일부 추가하면 안정성 향상

---

## ⚠️ 주의사항

- **법적 자문 대체 불가**: 본 앱은 참고용이며, 실제 계약 전 반드시 공인중개사·법무사·변호사 등 전문가 상담을 권장합니다
- **생성된 임시 계약서**는 초안일 뿐, 반드시 양 당사자가 내용을 검토하고 수정·서명해야 합니다
- **API 키 보안**: API 키는 브라우저 메모리에만 저장되지만, 공용 PC에서 사용 시 주의하세요
- **크레딧 소모**: 분석 1회당 대략 $0.05~0.20 정도 소모됩니다 (이미지 수·서류 크기에 따라 변동)

---

## 🔐 필수 특약 커스터마이즈

`REQUIRED_CLAUSES` 배열을 수정하면 자동 추가되는 특약을 변경할 수 있습니다:

```javascript
const REQUIRED_CLAUSES = [
  {
    id: 'unique_id',
    category: ['월세', '전세'],     // 적용 계약 유형
    title: '특약 제목',
    keywords: ['검색어1', '검색어2'],  // 기존 계약서에 이 단어가 없으면 누락으로 판단
    content: `- 특약 본문 여러 줄`
  },
  // ...
];
```

---

## 📝 라이선스

개인 용도 프로젝트 — 자유롭게 수정·사용하세요. Claude API 사용료는 본인 부담입니다.

---

## 🙏 도움이 필요하면

- Anthropic API 문서: https://docs.claude.com
- Anthropic Console: https://console.anthropic.com
- API 크레딧 충전 가이드: Console → Plans & Billing
