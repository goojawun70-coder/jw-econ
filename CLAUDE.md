# Jw경제 — 일일 발행 가이드

이 저장소(`gh-pages` 브랜치)는 "Jw경제" 경제 블로그의 실제 배포 사이트입니다. 매일 아침 자동 세션이 그날의 세계 경제·증시 요약과 종목 4개 심층 분석을 조사·작성해 이 브랜치에 직접 커밋·푸시합니다.

## 발행 전 필수 확인

- **중복 발행 방지**: 작업 시작 전 `git fetch origin gh-pages && git log -1`로 오늘 날짜 커밋이 이미 있는지 확인. 동시에 여러 세션이 이 저장소를 다룰 수 있으므로, 이미 오늘자 글이 발행돼 있다면 임의로 덮어쓰지 말고 상황을 보고할 것.
- **템플릿 재사용**: 신규 파일은 처음부터 새로 만들지 말고 가장 최근 `posts/*.html`을 읽어 CSS(:root 토큰, 라이트/다크 테마)와 클래스 구조를 그대로 복사해 내용만 교체.
- **종목 중복 회피**: `posts/` 디렉토리의 최근 5~7개 글에서 다룬 종목과 겹치지 않게 오늘의 심층분석 종목(인기 테마 국내1·미국1 + 비인기 업종 국내1·미국1)을 선정.
- **종목코드 먼저 확인**: 가격 검색 전에 "{회사명} 종목코드"로 정확한 코드를 확인하고, 시가총액·업종으로 교차 확인 후 가격을 검색 (지주회사·계열사 혼동 주의 — 예: 아모레퍼시픽 090430 vs 아모레퍼시픽홀딩스 002790).
- **VIX 공포지수 섹션 필수 포함** (2026-09-04 추가): 아래 "VIX(공포지수) 섹션" 항목을 매일 반드시 포함할 것. 상세 절차는 해당 섹션 참조.

## 증권사 리포트 소스 — 한경컨센서스 대신 네이버금융 리서치 사용

`markets.hankyung.com/consensus`의 리포트 테이블은 날짜/카테고리 필터를 **클릭해야** 채워지는 클라이언트 렌더링 구조라, URL 방문만으로는 항상 빈 목록(`검색결과가 없습니다`)이 나온다. **이 URL은 리포트 소스로 쓰지 말 것.**

대신 네이버금융 리서치 게시판을 사용한다 — 서버사이드 렌더링이라 그대로 스크래핑되고, 목록에 종목명·제목·증권사·**작성일**이 바로 노출된다:

- 종목분석: `https://finance.naver.com/research/company_list.naver`
- 시황정보: `https://finance.naver.com/research/market_info_list.naver`
- 산업분석: `https://finance.naver.com/research/industry_list.naver`
- 경제분석: `https://finance.naver.com/research/economy_list.naver`

**절차**:
1. 위 목록 페이지를 `apify--rag-web-browser` (query=URL, maxResults=1)로 가져와 작성일이 **오늘 또는 어제**인 리포트만 골라낸다 (`nid=...` 값과 함께).
2. 관심 종목·주제의 리포트는 상세 페이지 `https://finance.naver.com/research/company_read.naver?nid={nid}&page=1`를 다시 스크래핑한다 — 여기에 증권사·작성일·목표가·투자의견(Buy/Hold 등)·애널리스트가 쓴 요약 본문이 그대로 들어있다.
3. 이 요약 본문에서 실제 수치(매출·영업이익·YoY 성장률 등)와 목표가를 뽑아 "증권사 리포트 하이라이트" 섹션에 인용한다. 출처(증권사명 + 게시일)를 반드시 명시.
4. 한경컨센서스 스크래핑을 다시 시도하지 말 것 — 이미 실패가 확인된 방법이다. 네이버금융조차 안 되면 그때만 WebSearch로 대체하되, 검색 결과의 날짜가 실제 오늘/어제인지 반드시 대조 확인(오래된 기사가 섞여 나오는 경우가 많음).

## 종목 가격·재무 소스 — WebSearch로 찾지 말 것 (필수)

WebSearch로 "{종목명} 주가"를 검색하면 날짜가 뒤섞인 과거 데이터가 섞여 나온다 (몇 달 전 종가가 "오늘 가격"처럼 반환되는 사고가 실제로 발생했다). 심층분석 종목의 **전일 종가·PER·PBR·ROE·부채비율·목표주가는 반드시 아래 전용 페이지에서 직접 가져올 것** — 둘 다 서버사이드 렌더링이라 `apify--rag-web-browser`로 그대로 스크래핑된다.

- **국내 종목**: `https://finance.naver.com/item/main.naver?code={종목코드}`
  한 페이지에 전일가·현재가·PER·추정PER·PBR·ROE·부채비율·목표주가(컨센서스)·투자의견·52주 최고/최저가 표가 모두 들어있다. "전일" 필드를 종목 카드의 "종가"로 쓴다.
- **해외(미국) 종목**: `https://stockanalysis.com/stocks/{티커소문자}/`
  Previous Close, PE Ratio, Forward PE, Price Target, Market Cap, EPS, Dividend, 52-Week Range가 한 번에 나온다. "At close: ..." 시각의 가격이 아니라 "Previous Close" 필드가 전일 종가임에 유의 — 페이지 상단 큰 숫자는 그날 장중/애프터마켓 가격일 수 있다.

이 두 소스를 확인하지 않고 WebSearch로 찾은 가격을 그대로 쓰지 말 것. 2026-08-19자 초판에서 LS(006260)와 유한양행(000100)의 종가를 WebSearch로 잘못 가져와(각각 실제보다 10만원 이상 차이) 정정 발행한 사고가 있었다.

## 원/달러·원/엔 환율 소스 — Naver 일별 환율 페이지 사용 (2026-08-29 확인, 신뢰 가능)

기존에는 원/달러·원/100엔 환율을 신뢰 가능한 소스로 당일 확인하지 못해 지난 검증일(예: 8/21) 수치를 그대로 이월하는 방식을 썼으나, 아래 네이버금융 페이지가 서버사이드 렌더링으로 일별 매매기준율을 정확히 제공하는 것을 확인했다. **앞으로는 이 페이지로 당일(또는 최근 거래일) 환율을 직접 확인할 것 — 더 이상 구 baseline을 이월하지 말 것.**

- 원/달러: `https://finance.naver.com/marketindex/exchangeDailyQuote.naver?marketindexCd=FX_USDKRW`
- 원/100엔: `https://finance.naver.com/marketindex/exchangeDailyQuote.naver?marketindexCd=FX_JPYKRW`

두 페이지 모두 "매매기준율" 열의 최상단 행이 가장 최근 거래일(공휴일·주말 다음날은 그 이전 마지막 거래일) 종가 기준 환율이다. "전일대비" 열도 함께 있어 등락 방향을 바로 확인할 수 있다.

## 원자재(금·유가) 가격 소스 — 이것도 WebSearch 금지

금·유가는 종목이 아니라서 위 방법이 안 통하지만, WebSearch는 여기서도 똑같이 신뢰할 수 없다. 특히 금값은 스팟 가격을 보도하는 매체마다 시차·통화선물 계약월이 달라 같은 날인데도 기사마다 수백 달러씩 차이가 난다 (2026-08-20자 초판에서 이 문제로 실제 $4,500대였던 금값을 $4,151.9로 잘못 게재해 정정한 사고가 있었다). 대신 관련 ETF 페이지로 대체 확인한다 — 둘 다 stockanalysis.com에서 서버사이드 렌더링되고, "News" 섹션에 Kitco 등 1차 소스 헤드라인이 날짜와 함께 그대로 노출되어 스팟가·등락 원인을 같이 확인할 수 있다:

- **금**: `https://stockanalysis.com/etf/gld/` (SPDR Gold Shares, LBMA Gold Price 추종) — Previous Close·등락률과 News 섹션의 Kitco PM/AM Report 헤드라인(예: "Gold surges above $4,500")을 함께 인용.
- **유가(WTI)**: `https://stockanalysis.com/etf/uso/` (Front Month WTI 선물 추종) — Previous Close·등락률과 News 섹션 헤드라인을 함께 확인.

ETF 가격 자체를 "금 온스당 가격"으로 직접 환산하지 말 것(운용보수·리밸런싱으로 정확한 배율이 아님) — News 섹션에 있는 1차 소스(Kitco 등) 기사가 실제로 인용한 스팟 가격·등락 이유를 그대로 가져와 쓴다.

## 美 지수(다우·S&P500·나스닥) 소스 — WebSearch 합성 답변 신뢰 금지

WebSearch로 "다우존스 S&P500 나스닥 {날짜} 마감"류를 검색하면, 검색엔진이 여러 기사를 뒤섞어 "답"을 합성해주는데 이게 날짜를 하루~이틀씩 밀려서 틀리는 사고가 실제로 발생했다 (2026-08-21자 초판에서 "8/20 마감"이라며 실제로는 8/19 데이터였던 수치(다우 53,463·S&P500 7,708·나스닥 26,331, 모두 상승)를 게재 — 실제 8/20은 국채금리 재반등으로 3대 지수가 모두 하락한 날이었다). 같은 질의를 다르게 표현해서 여러 번 검색하면 서로 모순되는 "답"이 나오기도 하므로, 어느 한 번의 WebSearch 결과만 보고 확정하지 말 것.

대신 지수를 추종하는 ETF의 **일별 히스토리 표**(날짜별 시가·고가·저가·종가·등락률이 표로 그대로 노출, 서버사이드 렌더링)로 교차 확인한다:

- **다우존스**: `https://stockanalysis.com/etf/dia/history/` (DIA, 종가×100 ≈ 다우 지수)
- **S&P500**: `https://stockanalysis.com/etf/spy/history/` (SPY, 종가×10.02 ≈ S&P500 지수)
- **나스닥종합**: `https://stockanalysis.com/etf/oneq/history/` (ONEQ, Fidelity Nasdaq Composite Index ETF — QQQ는 나스닥100이라 다른 지수이니 쓰지 말 것)

이 표에서 원하는 날짜의 등락률(%)을 확인하고, 직전 영업일의 (이미 검증된) 지수 종가에 그 등락률을 곱해서 지수 값을 산출한다. ETF 종가를 배율로 환산한 값과, WebSearch로 찾은 명시적 지수 수치("OOO에 마감", "OOO까지 하락" 등 기사 인용문)가 서로 맞아떨어지는지 반드시 교차 검증할 것 — 두 값이 크게 어긋나면(50pt 이상) WebSearch 쪽을 버리고 ETF 히스토리 기준으로 쓴다.

## VIX(공포지수) 섹션 — 매일 필수 포함 (2026-09-04 추가)

"증시 동향" 섹션 바로 다음, "환율·원자재·금리" 섹션 앞에 VIX(CBOE Volatility Index, 변동성지수) 신호등 카드를 매일 추가한다. 사용자가 명시적으로 요청한 상시 섹션이므로 절대 생략하지 말 것.

### 데이터 소스 (WebSearch 합성 답변 신뢰 금지 — 위 美 지수 섹션과 동일한 이유)

VIX를 그대로 추종하는 ETF는 없다(VIXY·UVXY 등은 선물 만기 롤오버로 인한 콘탱고 감가 때문에 지수 수준과 크게 괴리되므로 절대 대용치로 쓰지 말 것 — 금/유가 ETF와는 다른 케이스). 대신 아래 두 지수 전용 페이지를 `apify--rag-web-browser`로 스크래핑해서 서로 교차 확인한다:

- `https://www.cnbc.com/quotes/.VIX`
- `https://finance.yahoo.com/quote/%5EVIX/`

두 값이 1pt 이상 차이 나면(장중 변동 때문일 수 있음) 더 최근 갱신 시각의 값을 쓰고, 두 소스 다 스크래핑이 안 되면(빈 페이지 등) WebSearch로 대체하되 기사 날짜가 오늘/최근 영업일인지 반드시 대조 확인 후 "정확도 낮음"을 내부적으로 인지하고 보수적으로 (전일 대비 급변 언급 자제) 서술할 것. 이 방법으로 새로운 신뢰 가능한 소스를 찾으면 이 섹션에 갱신해서 기록할 것 (다른 섹션들처럼).

### 등급 구간 및 표시 문구 (고정 — 임의로 바꾸지 말 것)

| VIX 값 | 등급 | 신호등 색 | 문구 (그대로 사용) |
|---|---|---|---|
| 30 초과 | 공포 (Fear) | 빨강 (`--rise`) | "공포 지수가 매우 높습니다. 매수하기에 매력 있습니다." |
| 20 이상 30 이하 | 경계 (Caution) | 노랑/골드 (`--gold`) | "변동성이 확대되고 있습니다. 시장 불확실성에 유의하세요." |
| 15 이상 20 미만 | 안정 (Stable) | 초록 (`--brand`) | "시장 공포 지수가 안정적입니다." |
| 15 미만 | 저변동 (Low) | 파랑 (`--fall`) | "변동성이 매우 낮은 구간입니다. 시장이 과도하게 낙관적일 수 있어 단기 되돌림에 유의하세요." |

문구는 위 표를 그대로 쓰고, 그 뒤에 그날의 구체적 수치·전일 대비 등락만 한 문장 덧붙인다 (예: "VIX 15.2(전일比 -0.8pt)로 시장 공포 지수가 안정적입니다.").

### HTML/CSS 템플릿 (그대로 복사해서 값만 채울 것)

CSS — 매일 발행 시 `<style>` 안에 아래 블록을 (없으면) 추가:

```css
.vix-card{background:var(--surface); border:1px solid var(--line); border-left:4px solid var(--line); border-radius:8px; padding:20px 22px; box-shadow:var(--shadow);}
.vix-top{display:flex; align-items:center; gap:12px; flex-wrap:wrap;}
.vix-light{width:14px; height:14px; border-radius:50%; display:inline-block; flex:0 0 auto;}
.vix-value{font-size:28px; font-weight:800; letter-spacing:-0.01em;}
.vix-label{font-size:12.5px; font-weight:700; padding:4px 10px; border-radius:99px; background:var(--surface-2); letter-spacing:0.02em;}
.vix-msg{margin:14px 0 0; font-size:15px; font-weight:600;}
.vix-note{margin:10px 0 0; font-size:13px; color:var(--ink-faint); line-height:1.6;}
.vix-low{border-left-color:var(--fall);} .vix-low .vix-light{background:var(--fall);} .vix-low .vix-label,.vix-low .vix-msg{color:var(--fall);}
.vix-stable{border-left-color:var(--brand);} .vix-stable .vix-light{background:var(--brand);} .vix-stable .vix-label,.vix-stable .vix-msg{color:var(--brand-ink);}
.vix-caution{border-left-color:var(--gold);} .vix-caution .vix-light{background:var(--gold);} .vix-caution .vix-label,.vix-caution .vix-msg{color:var(--gold);}
.vix-fear{border-left-color:var(--rise);} .vix-fear .vix-light{background:var(--rise);} .vix-fear .vix-label,.vix-fear .vix-msg{color:var(--rise);}
```

HTML — "증시 동향" `</section>` 바로 다음에 삽입 (등급에 맞는 클래스 하나만 `vix-low`/`vix-stable`/`vix-caution`/`vix-fear` 중 골라서 `.vix-card`에 적용, 라벨 텍스트도 등급에 맞게 교체):

```html
<section>
  <div class="eyebrow"><div class="label-kr">VIX 공포지수</div><div class="label-en">Fear &amp; Greed Signal</div></div>
  <div class="vix-card vix-stable">
    <div class="vix-top">
      <span class="vix-light"></span>
      <span class="vix-value mono">15.2</span>
      <span class="vix-label">안정</span>
    </div>
    <p class="vix-msg">VIX 15.2(전일比 -0.8pt)로 시장 공포 지수가 안정적입니다.</p>
    <p class="vix-note">VIX(변동성지수, CBOE Volatility Index)는 S&amp;P500 옵션 가격에 내재된 향후 30일간 예상 변동성을 수치화한 지표로, 흔히 "공포지수"라 불립니다. 수치가 높을수록 투자자들의 불안 심리가 크다는 뜻이며, 역사적으로 30을 넘는 극단적 수준은 시장이 단기 바닥을 형성하는 국면과 자주 맞물려 역발상 매수 신호로도 해석됩니다. 반대로 지나치게 낮은 수준은 시장의 과도한 낙관을 시사하기도 합니다.</p>
  </div>
</section>
```

라벨 텍스트: 빨강="공포", 노랑="경계", 초록="안정", 파랑="저변동". `.vix-value`는 항상 소수 첫째 자리까지 표기.
