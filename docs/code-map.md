# index.html 코드맵 (1,577줄)

> 최종 업데이트: 2026-05-04 | 단일 파일 SPA (HTML + CSS + JS)

---

## 전체 구조 개요

```
L1-50     HTML <head> + CSS 스타일
L52-177   HTML <body> — 4개 탭 UI
L179-1574 <script> — 전체 JavaScript 로직
L1573     initSettings() 호출 (앱 시작점)
```

---

## A. HTML 구조 (L1-177)

| 라인 | 영역 | 설명 |
|------|------|------|
| 1-50 | `<head>` | CSS 변수(`--main:#00338D`, `--gold:#C5A44E`), 스타일 정의. xlsx-js-style CDN |
| 52-57 | 헤더 | 앱 제목 "JSY Food 재고수불부 자동생성기" |
| 59-64 | 탭 바 | 설정 / 원육 / 가공육 / 소스 (4탭) |
| 66-95 | **설정 탭** `#panel-settings` | 작업기간 선택(startMonth/endMonth), 기초재고 입력 테이블(원육/가공육) |
| 97-131 | **원육 탭** `#panel-rawmeat` | 파일업로드(매입+작업일지3), 수동입력, 계산/결과 |
| 133-144 | **가공육 탭** `#panel-processed` | 계산버튼, 결과 (입력은 원육 탭에서 함께 파싱) |
| 146-177 | **소스 탭** `#panel-sauce` | 작업일지(소스) 업로드, 수동입력, 계산/결과 |

### 파일 업로드 입력 요소

| ID | 탭 | 용도 |
|----|-----|------|
| `filePurchase` | 원육 | 매입자료 (품목별 입고현황.xlsx) |
| `fileWL_CP` | 원육 | 작업일지 참프레시 |
| `fileWL_YM` | 원육 | 작업일지 영미트 |
| `fileWL_IC` | 원육 | 작업일지 인천공장(제이에스와이푸드) |
| `fileWL_Sauce` | 소스 | 작업일지 소스(인천공장 소스 수불부) |
| `fileInitInv` | 설정 | 25년 수불부 기초재고 (미구현, hidden) |

---

## B. 상수 (L179-236)

| 라인 | 상수명 | 내용 |
|------|--------|------|
| 183 | `COMPANIES_RAW` | 원육 업체 4곳: 참프레시, 영미트, 인천공장, 스마일푸드 |
| 184 | `COMPANIES_PROC` | 가공육 업체 4곳 (원육과 동일) |
| 186 | `RAW_ITEMS` | 원육 품목 13종 |
| 188-193 | `PROC_ITEMS` | 가공육 품목 19종 (선물세트 포함) |
| 197-217 | `PROC_BOM` | 가공육 BOM — 품목별 사용 원육/표준무게/사용비율 (25년12월 기준) |
| 220 | `SAUCE_RAW` | 소스 원재료 11종 |
| 221 | `SAUCE_PRODUCTS` | 소스 제품 2종: 돼지갈비양념, 마늘갈비양념 |
| 223-236 | `SAUCE_BOM` | 소스 BOM — 제품별 원재료 표준무게 |

---

## C. 전역 상태 STATE (L238-339)

| 라인 | 필드 | 타입 | 설명 |
|------|------|------|------|
| 242-267 | `DEFAULT_INIT_RAW` | 하드코딩 | 원육 25년12월 기말 (참프레시/영미트/인천공장) |
| 268-295 | `DEFAULT_INIT_PROC` | 하드코딩 | 가공육 25년12월 기말 |
| 296-317 | `DEFAULT_INIT_SAUCE_*` | 하드코딩 | 소스 원재료/제품 25년12월 기말 |
| 322 | `startMonth/endMonth` | string | 작업 기간 (기본 2601~2612) |
| 324-327 | `initRaw/initProc/initSauceRaw/initSauceProd` | object | 기초재고 (하드코딩 값 복사) |
| 329 | `purchase` | `{month:{item:{qty,amount}}}` | 매입자료 파싱 결과 |
| 330 | `worklog` | `{company:{month:{item:{incoming,outgoing}}}}` | 원육 작업일지 파싱 결과 |
| 331 | `procWorklog` | `{company:{month:{item:{production,outgoing}}}}` | 가공육 작업일지 파싱 결과 |
| 332 | `sauceInput` | `{month:{item:{incQty,incAmt,outQty}}}` | 소스 원재료 입출고 |
| 333 | `sauceProdInput` | `{month:{product:qty}}` | 소스 제품 생산수량 |
| 334 | `sauceProdOutput` | `{month:{'company_product':qty}}` | 소스 제품 출고(배분처별) |
| 336-338 | `rawResult/procResult/sauceResult` | object | 계산 결과 저장 |

---

## D. 유틸리티 (L342-373)

| 라인 | 함수 | 역할 |
|------|------|------|
| 344 | `fmt(n)` | 정수 천단위 콤마 |
| 348 | `fmtDec(n, d)` | 소수점 d자리 포맷 |
| 352 | `safe(v)` | NaN→0 변환 |
| 354 | `getMonths()` | startMonth~endMonth 배열 반환 |
| 364 | `monthLabel(code)` | '2601'→'26년 1월' |
| 369 | `prevMonth(code)` | 이전 월 코드 반환 |

---

## E. 탭 전환 (L376-385)

`.tab` 클릭 이벤트 → `.panel` active 토글

---

## F. 설정 탭 (L387-453)

| 라인 | 함수 | 역할 |
|------|------|------|
| 390 | `initSettings()` | 월 드롭다운 생성, 기초재고 테이블 렌더 |
| 406 | `refreshMonthSelects()` | 전체 월 드롭다운 갱신 |
| 418 | `fmtInit(n)` | 기초재고 입력값 포맷 |
| 419 | `initInput(co,item,field,val)` | 기초재고 input 태그 생성 |
| 427 | `renderInitialTables()` | 원육 기초재고 편집 테이블 렌더 |
| 449 | `updInitRaw(co,item,field,val)` | 기초재고 값 STATE 업데이트 |

---

## G. 핵심 계산 함수 (L455-483)

| 라인 | 함수 | 역할 |
|------|------|------|
| 458 | `weightedAvg(begQty,begAmt,incQty,incAmt)` | 총평균단가 계산 |
| 465 | `calcInventoryItem(beg,incQty,incPrice,incAmt,outQty)` | **단일 품목 수불부 계산** — 기초→입고→총평균단가→출고→기말 |

**반환 구조**: `{beg, inc, out, end, avgPrice}` — 각각 `{qty, price, amount}`

---

## H. 원육 수불부 계산 엔진 (L486-563)

### `calculateRawMeat()` (L488)

```
월별 루프:
  1. 컨테이너 = 매입자료 (입고=출고, 기초/기말=0)
  2. 각 업체(4곳):
     기초 = 전월기말 or initRaw
     입고 = 작업일지 입고수량 × 컨테이너 입고단가
     출고 = 작업일지 출고수량
     → calcInventoryItem() 호출
  3. 전사 = 4개 업체 합산
```

**주의**: 스마일푸드용 작업일지 파서 없음 — worklog['스마일푸드']는 항상 빈 값

---

## I. 가공육 수불부 계산 엔진 (L566-678)

### `calculateProcessed()` (L568)

```
사전조건: rawResult 존재 필요
월별 루프:
  각 업체:
    1. 원육출고금액 수집 (rawResult에서)
    2. 생산수량 수집 (procWorklog에서)
    3. 원육별 표준무게합계 = Σ(생산수량 × BOM 표준무게)
    4. BOM 비례배분 → 투입원가(입고금액)
       부위별비율 = (품목 표준무게합) / (전체 표준무게합계)
       투입원가 = 부위별비율 × 원육출고금액
    5. 출고 = procWorklog 출고수량 × 총평균단가
  전사 = 4개 업체 합산
```

### `emptyRow()` (L673) — 빈 수불부 행

---

## J. 소스 수불부 계산 엔진 (L680-753)

### `calculateSauce()` (L683)

```
월별 루프:
  1. 원재료 총평균법 (sauceInput → calcInventoryItem)
  2. BOM 비례배분 (가공육과 동일 로직, matTotalStdWt 기반)
  3. 인천공장 제품만 계산 (참프레시/영미트 미구현)
```

**한계**: 인천공장 제품만 계산, 참프레시/영미트 소스제품 섹션 미구현

---

## K. 결과 렌더링 (L755-860)

| 라인 | 함수 | 대상 |
|------|------|------|
| 758 | `renderRawResult()` | 원육 — 전사+컨테이너+4업체, 13품목 테이블 |
| 798 | `renderProcResult()` | 가공육 — 전사+4업체, 19품목 테이블 |
| 827 | `renderSauceResult()` | 소스 — 원재료 + 인천공장 제품 테이블 |

공통 구조: 기초/입고/출고/기말 × 수량/단가/금액 (12열)
음수재고는 `.neg` 클래스 (빨강)

---

## L. 수동 입력 모드 (L862-915, L1489-1545)

### 원육 수동입력

| 라인 | 함수 | 역할 |
|------|------|------|
| 865 | `showManualRawInput()` | 수동입력 패널 토글 |
| 871 | `renderManualRawTables()` | 매입(품목×수량/금액) + 업체별(품목×입고/출고) 테이블 |
| 902 | `updPurchase(month,item,field,val)` | STATE.purchase 업데이트 |
| 908 | `updWorklog(co,month,item,field,val)` | STATE.worklog 업데이트 |

### 소스 수동입력

| 라인 | 함수 | 역할 |
|------|------|------|
| 1489 | `showManualSauceInput()` | 소스 수동입력 패널 토글 |
| 1499 | `renderSauceManualTables()` | 원재료(incQty/incAmt/outQty) + 제품(생산/출고) 테이블 |
| 1533 | `updSauceRaw(month,item,field,val)` | STATE.sauceInput 업데이트 |
| 1538 | `updSauceProd(month,prod,val)` | STATE.sauceProdInput 업데이트 |
| 1542 | `updSauceProdOut(month,prod,val)` | STATE.sauceProdOutput 업데이트 |

---

## M. 엑셀 출력 (L917-1099)

### 스타일 & 헬퍼 (L920-1039)

| 라인 | 함수/상수 | 역할 |
|------|-----------|------|
| 923 | `XS` 객체 | 엑셀 셀 스타일 정의 (테마색 #00338D) |
| 942 | `bs(s)` | 스타일에 border 추가 |
| 944 | `xC(ws,r,c,v,s)` | 셀에 값+스타일 쓰기 |
| 949 | `xF(ws,r,c,f,s)` | 셀에 수식+스타일 쓰기 |
| 953 | `xEmpty(ws,r,c,s)` | 빈 셀(스타일만) |
| 957 | `addMerge(ws,r1,c1,r2,c2)` | 셀 병합 |
| 961 | `setRange(ws,r,c)` | 시트 범위 확장 |
| 968 | `writeSectionHeader(ws,r,name)` | 섹션 헤더행 (업체명 + 기초/입고/출고/기말) |
| 979 | `writeColHeader(ws,r)` | 열 헤더행 (품목/단위/수량/단가/금액...) |
| 984 | `writeDataRow(ws,r,item,unit,d)` | 데이터행 (값+수식) — **단가/출고/기말은 엑셀 수식** |
| 1002 | `writeSumRow(ws,r,dataRows)` | 합계행 (SUM 수식) |
| 1020 | `writeInvSection(ws,r0,name,items,data,unitFn)` | 하나의 섹션(헤더+데이터+합계) 일괄 작성 |

### 엑셀 다운로드 함수

| 라인 | 함수 | 파일명 |
|------|------|--------|
| 1041 | `exportRawMeatExcel()` | 원육수불부_2601-2612.xlsx |
| 1058 | `exportProcessedExcel()` | 가공육수불부_2601-2612.xlsx |
| 1080 | `exportSauceExcel()` | 소스수불부_2601-2612.xlsx |

---

## N. 파일 파싱 — 매입자료 (L1101-1244)

### `parseRawMeatInputs()` (L1104)

오케스트레이터: 4개 파일(매입+작업일지3) 순차 파싱, 3초 후 결과 표시

### `PURCHASE_ITEM_MAP` (L1158-1170)

매입자료 품목코드/명 → 내부 품목명 매핑 (예: '수입갈비'→'갈비', '8025'→'갈비')

### `parsePurchaseFile(file)` (L1172)

```
헤더행 자동탐지 → 품목코드/품명/일자/수량/공급가 열 매핑
행별: 일자→월코드 변환, 품목명 매핑, 월별/품목별 수량+금액 누적
→ STATE.purchase에 저장
```

---

## O. 파일 파싱 — 작업일지 (L1246-1380)

### 매핑 테이블

| 라인 | 상수 | 설명 |
|------|------|------|
| 1247-1257 | `WORKLOG_RAW_MAP` | 작업일지 원육 품목명→내부명 (예: '소갈비'→'갈비') |
| 1259-1271 | `WORKLOG_PROC_MAP` | 작업일지 가공육 품목명→내부명 (예: '돼지갈비(동그랑떙)'→'돼지갈비(동그랑땡)') |

### `cleanWorklogName(raw, mapObj)` (L1273)

줄바꿈·공백 정리, 2줄 합성 시도 (예: "돼지갈비\n(동그랑떙)" → "돼지갈비(동그랑떙)")

### `parseWorklogFile(file, company)` (L1286)

```
시트별 (4자리 숫자 = 월):
  섹션 감지: "원육" → raw, "가공육" → proc, "작업비" → 무시
  품목명 매핑 → 입고/출고 수량 추출 (col4=합계)
  → STATE.worklog[company][month] / STATE.procWorklog[company][month]

후처리 (L1358-1376):
  - 안창살/우설: 원육출고 = 가공육 입출고 (소분판매 pass-through)
  - 깐양: 막내장출고 + 깐양출고 = 가공육 깐양 생산수량
```

---

## P. 파일 파싱 — 소스 작업일지 (L1382-1487)

### 매핑 테이블

| 라인 | 상수 | 설명 |
|------|------|------|
| 1385-1397 | `SAUCE_WL_RAW_MAP` | 소스 원재료명 매핑 (예: '정종(백화수복)'→'정종', '후추가루'→'후추') |
| 1398-1401 | `SAUCE_WL_PROD_MAP` | 소스 제품명 매핑 (2종) |

### `parseSauceInputs()` (L1403) — 오케스트레이터

### `parseSauceWorklogFile(file, callback)` (L1417)

```
시트별 (4자리 숫자 = 월):
  섹션 감지: "1. 원재료" → raw, "2. 제품" → product
  원재료: 입고수량/출고수량 추출 → STATE.sauceInput
  제품:
    입고 → STATE.sauceProdInput (생산수량)
    출고 → STATE.sauceProdOutput['인천공장_제품명']
    참프레시/영미트 배분행 → STATE.sauceProdOutput['업체_제품명']
```

---

## Q. 기초재고 불러오기 (L1547-1568)

| 라인 | 함수 | 상태 |
|------|------|------|
| 1550 | `importInitialInventory()` | 파일선택 다이얼로그 열기 |
| 1554 | 파일 change 이벤트 | **미구현** — alert로 안내만 |
| 1562 | `clearInitialInventory()` | 하드코딩 기본값으로 초기화 |

---

## R. 앱 시작점 (L1573)

`initSettings()` 호출 → 월 드롭다운 생성, 기초재고 테이블 렌더

---

## 미구현 / 한계

| 항목 | 설명 |
|------|------|
| **스마일푸드 작업일지** | 파서 없음. 25년에 안창살/홍두깨 소분 실적 있었으나 데이터 입력 경로 없음 |
| **소스 참프레시/영미트 제품** | calculateSauce()가 인천공장만 계산, 타 업체 제품 수불부 미생성 |
| **기초재고 자동 불러오기** | 파일 업로드 UI는 있으나 파서 미구현 (하드코딩 값 사용) |
| **공장간 이동** | 가공육 내부이동(참→인천 등) 처리 로직 없음 |
| **가공육 수동입력** | 원육만 수동입력 UI 있음, 가공육은 없음 |
| **소스 입고금액** | 작업일지에서 수량만 추출, 금액은 수동입력 필요 |
