# index.html 코드맵 (1,851줄)

> 최종 업데이트: 2026-05-07 (2차) | 단일 파일 SPA (HTML + CSS + JS)

---

## 전체 구조 개요

```
L1-50     HTML <head> + CSS 스타일
L52-177   HTML <body> — 4개 탭 UI
L179-1851 <script> — 전체 JavaScript 로직
L1848     initSettings() 호출 (앱 시작점)
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

## B. 상수 (L180-263)

| 라인 | 상수명 | 내용 |
|------|--------|------|
| 183 | `COMPANIES_RAW` | 원육 업체 4곳: 참프레시, 영미트, 인천공장, 스마일푸드 |
| 184 | `COMPANIES_PROC` | 가공육 업체 4곳 (원육과 동일) |
| 186 | `RAW_ITEMS` | 원육 품목 13종 |
| 188-193 | `PROC_ITEMS` | 가공육 품목 19종 (선물용 4종 포함) |
| 197-217 | `PROC_BOM` | 가공육 BOM — 품목별 사용 원육/표준무게/사용비율 (25년12월 기준). 선물용 4종 BOM 포함 |
| 220 | `SAUCE_RAW` | 소스 원재료 11종 |
| 221 | `SAUCE_PRODUCTS` | 소스 제품 2종: 돼지갈비양념, 마늘갈비양념 |
| 223-236 | `SAUCE_BOM` | 소스 BOM — 제품별 원재료 표준무게 |
| 238 | `SHINE_ITEMS` | 샤인스카이(원육) 4종: 양념갈비/소불고기/LA갈비/양념구이[선물] |
| 240-244 | `GIFT_SET_ITEMS` | 선물세트 15종 (명품갈비세트~실속스페셜2호2구) |
| 246-262 | `GIFT_SET_BOM` | 선물세트 BOM — 세트별 구성 원육 품목/수량 |

---

## C. 전역 상태 STATE (L264-381)

| 라인 | 필드 | 타입 | 설명 |
|------|------|------|------|
| 268-293 | `DEFAULT_INIT_RAW` | 하드코딩 | 원육 25년12월 기말 (참프레시/영미트/인천공장) |
| 294-328 | `DEFAULT_INIT_PROC` | 하드코딩 | 가공육 25년12월 기말 (참프레시/영미트/인천공장 + 샤인스카이 4종) |
| 329-350 | `DEFAULT_INIT_SAUCE_*` | 하드코딩 | 소스 원재료/제품 25년12월 기말 (영미트 마늘갈비양념 포함) |
| 357 | `startMonth/endMonth` | string | 작업 기간 (기본 2601~2612) |
| 359-362 | `initRaw/initProc/initSauceRaw/initSauceProd` | object | 기초재고 (하드코딩 값 복사) |
| 364 | `purchase` | `{month:{item:{qty,amount}}}` | 매입자료 파싱 결과 |
| 365 | `worklog` | `{company:{month:{item:{incoming,outgoing}}}}` | 원육 작업일지 파싱 결과 |
| 366 | `procWorklog` | `{company:{month:{item:{production,outgoing,outByDest}}}}` | 가공육 작업일지 파싱 결과 |
| 367 | `sauceInput` | `{month:{item:{incQty,incAmt,outQty}}}` | 소스 원재료 입출고 |
| 368 | `sauceProdInput` | `{month:{product:qty}}` | 소스 제품 생산수량 |
| 369 | `shineTransfer` | `{company:{month:{item:qty}}}` | 샤인스카이 이전 수량 |
| 370 | `etcTransfer` | `{company:{month:{item:qty}}}` | 기타 이전 수량 |
| 371 | `sauceProdOutput` | `{month:{'company_product':qty}}` | 소스 제품 출고(배분처별) |
| 373-376 | `purchaseRawWS/purchaseColMap/purchaseHeaderRow/purchaseDataLen` | WS/obj/int | **매입원본 WS 보존 (엑셀 SUMIFS 연결용)** |
| 378-380 | `rawResult/procResult/sauceResult` | object | 계산 결과 저장 |

---

## D. 유틸리티 (L384-416)

| 라인 | 함수 | 역할 |
|------|------|------|
| 354 | `deepClone(o)` | 깊은 복사 |
| 386 | `fmt(n)` | 정수 천단위 콤마 |
| 390 | `fmtDec(n, d)` | 소수점 d자리 포맷 |
| 394 | `safe(v)` | NaN→0 변환 |
| 396 | `getMonths()` | startMonth~endMonth 배열 반환 |
| 406 | `monthLabel(code)` | '2601'→'26년 1월' |
| 411 | `prevMonth(code)` | 이전 월 코드 반환 |

---

## E. 탭 전환 (L418-429)

`.tab` 클릭 이벤트 → `.panel` active 토글

---

## F. 설정 탭 (L430-496)

| 라인 | 함수 | 역할 |
|------|------|------|
| 432 | `initSettings()` | 월 드롭다운 생성, 기초재고 테이블 렌더 |
| 448 | `refreshMonthSelects()` | 전체 월 드롭다운 갱신 |
| 460 | `fmtInit(n)` | 기초재고 입력값 포맷 |
| 461 | `initInput(co,item,field,val,type)` | 기초재고 input 태그 생성 |
| 469 | `renderInitialTables()` | 원육 기초재고 편집 테이블 렌더 |
| 491 | `updInitRaw(co,item,field,val)` | 기초재고 값 STATE 업데이트 |

---

## G. 핵심 계산 함수 (L498-527)

| 라인 | 함수 | 역할 |
|------|------|------|
| 500 | `weightedAvg(begQty,begAmt,incQty,incAmt)` | 총평균단가 계산 |
| 507 | `calcInventoryItem(beg,incQty,incPrice,incAmt,outQty)` | **단일 품목 수불부 계산** — 기초→입고→총평균단가→출고→기말 |

**반환 구조**: `{beg, inc, out, end, avgPrice}` — 각각 `{qty, price, amount}`

---

## H. 원육 수불부 계산 엔진 (L528-607)

### `calculateRawMeat()` (L530)

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

## I. 가공육 수불부 계산 엔진 (L608-792)

### `calculateProcessed()` (L610)

```
사전조건: rawResult 존재 필요
월별 루프:
  각 업체(4곳):
    1. 원육출고금액 수집 (rawResult에서)
    2. 생산수량 수집 (procWorklog에서)
    3. 원육별 표준무게합계 = Σ(생산수량 × BOM 표준무게)
    4. BOM 비례배분 → 투입원가(입고금액)
    5. 출고 = procWorklog 출고수량 × 총평균단가
  본푸드 (L684):
    입고 = 다른 업체의 본푸드 출고행, 단가 = 출고업체 총평균단가
    출고 = 입고와 동일 (pass-through)
  샤인스카이(원육) (L706):
    입고 = 다른 업체의 '샤인' 출고행, 단가 = 출고업체 총평균단가
    출고 = 입고와 동일 (pass-through)
  샤인선물세트 (L728):
    선물세트 15종, 조립/출하 데이터 미확보 → 현재 빈 값
  기타선물세트 (L740):
    선물세트 15종, 현재 빈 값
  전사 (L752):
    ALL_PROC = 생산업체4곳 + 본푸드 + 샤인스카이 합산
```

### `emptyRow()` (L786) — 빈 수불부 행

---

## J. 소스 수불부 계산 엔진 (L794-888)

### `calculateSauce()` (L796)

```
월별 루프:
  1. 원재료 총평균법 (sauceInput → calcInventoryItem)
  2. BOM 비례배분 (가공육과 동일 로직, matTotalStdWt 기반)
  3. 인천공장 제품 계산
  4. 참프레시/영미트 제품 (L858):
     입고 = 인천공장에서 이전받은 수량 × 인천 총평균단가
     출고 = 입고와 동일 (pass-through)
```

---

## K. 결과 렌더링 (L889-1004)

| 라인 | 함수 | 대상 |
|------|------|------|
| 891 | `renderRawResult()` | 원육 — 전사+컨테이너+4업체, 13품목 테이블 |
| 931 | `renderProcResult()` | 가공육 — 전사+4업체+본푸드+샤인스카이(원육)+샤인선물세트+기타선물세트 |
| 967 | `renderSauceResult()` | 소스 — 원재료 + 인천공장/참프레시/영미트 제품 테이블 |

공통 구조: 기초/입고/출고/기말 × 수량/단가/금액 (12열)
음수재고는 `.neg` 클래스 (빨강)

---

## L. 수동 입력 — 원육 (L1005-1058)

| 라인 | 함수 | 역할 |
|------|------|------|
| 1007 | `showManualRawInput()` | 수동입력 패널 토글 |
| 1013 | `renderManualRawTables()` | 매입(품목×수량/금액) + 업체별(품목×입고/출고) 테이블 |
| 1044 | `updPurchase(month,item,field,val)` | STATE.purchase 업데이트 |
| 1050 | `updWorklog(co,month,item,field,val)` | STATE.worklog 업데이트 |

소스 수동입력은 Q절 참조

---

## M. 엑셀 출력 (L1060-1285)

### 스타일 & 헬퍼 (L1063-1180)

| 라인 | 함수/상수 | 역할 |
|------|-----------|------|
| 1065 | `XS` 객체 | 엑셀 셀 스타일 정의 (테마색 #00338D) |
| 1084 | `bs(s)` | 스타일에 border 추가 |
| 1086 | `xC(ws,r,c,v,s)` | 셀에 값+스타일 쓰기 |
| 1091 | `xF(ws,r,c,f,s)` | 셀에 수식+스타일 쓰기 |
| 1095 | `xEmpty(ws,r,c,s)` | 빈 셀(스타일만) |
| 1099 | `addMerge(ws,r1,c1,r2,c2)` | 셀 병합 |
| 1103 | `setRange(ws,r,c)` | 시트 범위 확장 |
| 1110 | `writeSectionHeader(ws,r,name)` | 섹션 헤더행 (업체명 + 기초/입고/출고/기말) |
| 1121 | `writeColHeader(ws,r)` | 열 헤더행 (품목/단위/수량/단가/금액...) |
| 1126 | `writeDataRow(ws,r,item,unit,d)` | 데이터행 (값+수식) — 단가/출고/기말은 엑셀 수식 |
| 1144 | `writeSumRow(ws,r,dataRows)` | 합계행 (SUM 수식) |
| 1162 | `writeInvSection(ws,r0,name,items,data,unitFn)` | 하나의 섹션(헤더+데이터+합계) 일괄 작성. 0값 품목도 출력 |

### 엑셀 다운로드 함수

| 라인 | 함수 | 파일명 | 비고 |
|------|------|--------|------|
| 1182 | `exportRawMeatExcel()` | 원육수불부_2601-2612.xlsx | **매입원본 + 작업일지 시트 추가, 전 섹션 수식 연결** |
| 1283 | `exportProcessedExcel()` | 가공육수불부_2601-2612.xlsx | 본푸드+샤인스카이(원육)+샤인선물세트+기타선물세트 포함 |
| 1314 | `exportSauceExcel()` | 소스수불부_2601-2612.xlsx | 참프레시/영미트 제품 포함 |

### 원본 시트 + SUMIFS 수식 연결 (L1186-1278)

**매입원본 시트** (L1187):
```
원본 데이터 그대로 + 헬퍼 컬럼 2개 (매핑품목, 월코드)
parsePurchaseFile()에서 파싱 시 자동 추가
→ STATE.purchaseRawWS에 보존
```

**작업일지 시트** (L1191):
```
엑셀 출력 시점에 STATE.worklog에서 정규화 테이블 동적 생성
컬럼: 업체 / 월코드 / 품목 / 입고수량 / 출고수량
COMPANIES_RAW 4개사 × 월별 × 품목별 flat 구조
```

**섹션별 수식 연결**:
```
컨테이너 (L1232):
  F(입고수량) = SUMIFS('매입원본'!수량, 월코드=월, 매핑품목=품목)
  H(입고금액) = SUMIFS('매입원본'!금액, 월코드=월, 매핑품목=품목)
  I(출고수량) = F 참조 (전량출고)

각 업체 (L1244):
  F(입고수량) = SUMIFS('작업일지'!입고수량, 업체=업체명, 월코드=월, 품목=품목)
  G(입고단가) = 컨테이너 동일품목 G셀 참조
  H(입고금액) = F×G 수식
  I(출고수량) = SUMIFS('작업일지'!출고수량, 업체=업체명, 월코드=월, 품목=품목)

전사 (L1262):
  C(기초수량), E(기초금액), F(입고수량), H(입고금액), I(출고수량) = SUM(4개사)
  D,G,J,K,L,M,N = 기존 수식 자동계산
```

---

## N. 파일 파싱 — 매입자료 (L1338-1530)

### `parseRawMeatInputs()` (L1340)

오케스트레이터: 4개 파일(매입+작업일지3) 순차 파싱, 3초 후 결과 표시

### `PURCHASE_ITEM_MAP` (L1394-1406)

매입자료 품목코드/명 → 내부 품목명 매핑 (예: '수입갈비'→'갈비', '8025'→'갈비')

### `parsePurchaseFile(file)` (L1408)

```
헤더행 자동탐지 → 품목코드/품명/일자/수량/공급가 열 매핑
헬퍼 컬럼 추가 (매핑품목/월코드) → 원본 WS 끝에 2열 추가
행별: 일자→월코드 변환, 품목명 매핑, 월별/품목별 수량+금액 누적
→ STATE.purchase + STATE.purchaseRawWS에 저장
```

---

## O. 파일 파싱 — 작업일지 (L1531-1676)

### 매핑 테이블

| 라인 | 상수 | 설명 |
|------|------|------|
| 1513-1523 | `WORKLOG_RAW_MAP` | 작업일지 원육 품목명→내부명 (예: '소갈비'→'갈비') |
| 1525-1530 | `WORKLOG_PROC_MAP` | 작업일지 가공육 품목명→내부명 (예: '돼지갈비(동그랑떙)'→'돼지갈비(동그랑땡)') |

### `cleanWorklogName(raw, mapObj)` (L1533)

줄바꿈·공백 정리, 2줄 합성 시도 (예: "돼지갈비\n(동그랑떙)" → "돼지갈비(동그랑떙)")

### `parseWorklogFile(file, company)` (L1546)

```
시트별 (4자리 숫자 = 월):
  섹션 감지: "원육" → raw, "가공육" → proc, "작업비" → 무시
  품목명 매핑 → 입고/출고 수량 추출 (col4=합계)
  돼지갈비 단위 기반 리매핑: 팩/pack → 돼지갈비(동그랑땡)
  '본푸드'/'샤인'/'기타' 행 파싱 → bonpoodTransfer/shineTransfer/etcTransfer
  → STATE.worklog[company][month] / STATE.procWorklog[company][month]

후처리:
  - 안창살/우설: 원육출고 = 가공육 입출고 (소분판매 pass-through)
  - 깐양: 막내장출고 + 깐양출고 = 가공육 깐양 생산수량
```

---

## P. 파일 파싱 — 소스 작업일지 (L1678-1762)

### 매핑 테이블

| 라인 | 상수 | 설명 |
|------|------|------|
| 1681-1693 | `SAUCE_WL_RAW_MAP` | 소스 원재료명 매핑 (예: '정종(백화수복)'→'정종', '후추가루'→'후추') |
| 1694-1697 | `SAUCE_WL_PROD_MAP` | 소스 제품명 매핑 (2종) |

### `parseSauceInputs()` (L1678) — 오케스트레이터

### `parseSauceWorklogFile(file, callback)` (L1692)

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

## Q. 수동 입력 — 소스 (L1764-1821)

| 라인 | 함수 | 역할 |
|------|------|------|
| 1764 | `showManualSauceInput()` | 소스 수동입력 패널 토글 |
| 1774 | `renderSauceManualTables()` | 원재료(incQty/incAmt/outQty) + 제품(생산/출고) 테이블 |
| 1808 | `updSauceRaw(month,item,field,val)` | STATE.sauceInput 업데이트 |
| 1813 | `updSauceProd(month,prod,val)` | STATE.sauceProdInput 업데이트 |
| 1817 | `updSauceProdOut(month,prod,val)` | STATE.sauceProdOutput 업데이트 |

---

## R. 기초재고 불러오기 (L1823-1843)

| 라인 | 함수 | 상태 |
|------|------|------|
| 1825 | `importInitialInventory()` | 파일선택 다이얼로그 열기 |
| 1829 | 파일 change 이벤트 | **미구현** — alert로 안내만 |
| 1837 | `clearInitialInventory()` | 하드코딩 기본값으로 초기화 |

---

## S. 앱 시작점 (L1846-1851)

`initSettings()` 호출 → 월 드롭다운 생성, 기초재고 테이블 렌더

---

## 변경 이력 (코드맵 기준)

### 2026-05-07

| 커밋 | 변경 내용 |
|------|----------|
| `0e6b3fc` | **매입원본 시트 + SUMIFS 수식 연결**: 파싱 시 원본 WS 보존 + 헬퍼 컬럼(매핑품목/월코드), 엑셀 출력 시 매입원본 시트 추가 + 컨테이너 입고를 SUMIFS 수식으로 연결 |
| `c696910` | **업체/전사 수식 연결 확장**: 각 업체 입고단가→컨테이너 참조, 입고금액→F×G 수식. 전사 기초/입고/출고 수량·금액 → 4개사 SUM 수식 |
| `b69f18b` | **작업일지 정규화 시트 추가**: STATE.worklog에서 [업체/월코드/품목/입고수량/출고수량] flat 테이블 생성. 각 업체 입고수량(F)·출고수량(I) → 작업일지 시트 SUMIFS 연결 |

### 2026-05-06

| 커밋 | 변경 내용 |
|------|----------|
| `c0dec98` | 인천공장 돼지갈비(동그랑땡) 단위(팩/pack) 기반 리매핑 |
| `409e4f5` | 엑셀 다운로드 시 0값 품목도 모두 표시 (월별 비교 편의) |
| `3c96ff9` | 가공육에 **본푸드** 섹션 추가 — 작업일지 본푸드 출고행 파싱, 내부이전 입고 처리 |
| `76c7cfb` | **샤인스카이(원육)** 4품목 + **샤인선물세트** 15품목 + **기타선물세트** 추가. 소스에 **참프레시/영미트** 제품 섹션 추가 |

---

## 미구현 / 한계

| 항목 | 설명 |
|------|------|
| **스마일푸드 작업일지** | 파서 없음. 25년에 안창살/홍두깨 소분 실적 있었으나 데이터 입력 경로 없음 |
| **선물세트 조립 로직** | GIFT_SET_BOM 정의만 존재, 실제 조립/원가배분 계산 미구현 (빈 값) |
| **기초재고 자동 불러오기** | 파일 업로드 UI는 있으나 파서 미구현 (하드코딩 값 사용) |
| **가공육 수동입력** | 원육만 수동입력 UI 있음, 가공육은 없음 |
| **소스 입고금액** | 작업일지에서 수량만 추출, 금액은 수동입력 필요 |
| **샤인/본푸드 출고 상세** | 현재 pass-through (입고=출고), 실제 매장 출고 데이터 확보 시 전환 필요 |
