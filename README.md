# Inventory Control — Amazon + TikTok Shop (Next.js + TypeScript)

SaaS 스타일 재고 관제 대시보드. **Next.js 14 (App Router) + TypeScript**, 서버에서 서비스 계정으로 Google Sheet를 읽고(API Route), 클라이언트에서 브랜드 필터·KPI·차트·CSV Export를 처리합니다.

## 라우트
| URL | 화면 |
|---|---|
| `/` | Home Dashboard (헬스 요약 + 메뉴 카드) |
| `/amazon-inventory` | Amazon Inventory (CCONMA·FBA·회전일·Status) |
| `/inventory-planning` | 발주 필요 SKU + CCONMA→FBA 이동 보드 |
| `/tiktok-inventory` | TikTok Inventory (CCONMA·FBT) + 업로드 |
| `/amazon-sales` | Amazon Sales Dashboard (8 KPI + 3 차트 + Live Orders) |
| `/settings` | 연동 상태 · 데이터 소스 업로드 · 데모/테마 |
| `/api/health`, `/api/config`, `/api/data` | 서버 API (Node 런타임) |

상단 **Brand Filter**(데이터에서 자동 추출)가 모든 KPI·차트·테이블에 일괄 적용됩니다.

## 계산식
- Daily Avg = 30 Day Sales / 30
- Coverage Days = (CCONMA + FBA) / Daily Avg
- Status: `<30` Critical · `30–60` Warning · `>60` Healthy
- 발주(Coverage < 45): Order Qty = Daily Avg × 90 − 현재고
- 이동(FBA Coverage < 30 & CCONMA > 0): Transfer Qty = Daily Avg × 60 − FBA

## 로컬 실행
```bash
npm install
cp .env.example .env     # 값 채우기
npm run dev              # http://localhost:8080 (개발)
# 또는 프로덕션 동일 검증:
npm run build && npm start
```

## 환경변수
| 변수 | 설명 |
|---|---|
| `GOOGLE_SHEET_ID` | Amazon 소스 스프레드시트 ID |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | 서비스 계정 키 (JSON 원문 또는 base64) |
| `PORT` | Railway 자동 주입 (로컬만 수동) |

서비스 계정: GCP에서 JSON 키 생성 → **Google Sheets API 활성화** → 대상 스프레드시트를 서비스 계정 이메일에 **Viewer로 공유**(필수). 변수가 없으면 서버는 정상 기동하되 데이터는 비고, 프론트는 시드 마스터(152 SKU) + 데모/업로드로 동작합니다.

## Railway 배포
1. 이 폴더를 GitHub 레포 루트로 push (Dockerfile이 루트에 있어야 함)
2. Railway → New Project → Deploy from GitHub repo → 레포 선택 (`railway.json`이 Dockerfile 빌더 사용)
3. Variables 에 `GOOGLE_SHEET_ID`, `GOOGLE_SERVICE_ACCOUNT_JSON` 입력
4. Settings → Networking → Generate Domain → 포트 8080
5. 헬스체크 `/api/health`

Dockerfile은 멀티스테이지(빌드 → standalone 런타임)이며 `node:20-slim`(glibc) 기반이라 Alpine의 Vite/rollup·SWC musl 문제를 피합니다.

## 시트 탭 자동 감지
서버가 탭 제목으로 역할 매핑: `…CCONMA`→재고, `…FBA`→재고, `…일별/SALES HIST`→판매, `PRODUCT INFO/Master`→마스터. 컬럼 헤더는 클라이언트 퍼지 매칭(SKU/SAP, available/qty/재고, 7D/30D, date/units/매출 등). FBT·TikTok은 수동 업로드.

## 기술 검증 (이 빌드에서 통과 확인)
- `npm install` 정상
- `npm run build` 통과 (TS 오류 0, ESLint 오류·경고 0)
- standalone 서버 기동 + 전 라우트 200 + API graceful 응답
