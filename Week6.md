# Python 6주차 프로젝트 과제

이번 주 목표는 **시각화를 통해 분석 결과를 설득력 있게 보여주는 것**입니다.  
그래프를 많이 만드는 것보다, 각 그래프가 어떤 메시지를 전달하는지 설명하는 데 집중하세요.

**필수:** 본인의 노트북에서 진행한 내용과 실행 결과를 스크린샷으로 첨부해주세요.  
👀 수행 인증샷은 필수입니다.

노트북은 반드시 위에서 아래로 순서대로 실행해주세요. 중간 셀만 실행하면 이전에 만든 변수가 없어 오류가 날 수 있습니다.

---

## 참고 교재

- 『파이썬 라이브러리를 활용한 데이터 분석』
  - 6장 데이터 로딩과 저장, 파일 형식: p.247~309
  - 9장 그래프와 시각화: p.381~465
  - 10장 데이터 집계와 그룹 연산: p.381~465

---

## 이번 주 과제 목차

| 구분 | 내용 |
| --- | --- |
| 필수 1 | 그래프 3개 만들기 |
| 필수 2 | 그래프별 해석 작성 |
| 필수 3 | 최종 주장 후보 정리 |
| 선택 | 그래프 종류 비교 및 개선 |

---

<br>

<!-- 여기까진 그대로 둬 주세요-->

---

# 1️⃣ 필수 과제

먼저 정제 데이터를 불러오고, 그래프를 그릴 준비를 하세요. 파일명은 본인의 저장 방식에 맞게 바꿔주세요.

```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker

plt.rcParams["font.family"] = "Malgun Gothic"
plt.rcParams["axes.unicode_minus"] = False
plt.rcParams["figure.facecolor"] = "white"
plt.rcParams["axes.facecolor"] = "white"
plt.rcParams["savefig.facecolor"] = "white"
plt.rcParams["axes.spines.top"] = False
plt.rcParams["axes.spines.right"] = False
plt.rcParams["axes.titlesize"] = 11
plt.rcParams["axes.labelsize"] = 9
plt.rcParams["xtick.labelsize"] = 8
plt.rcParams["ytick.labelsize"] = 8

YELLOW = "#F2C94C"   # 단일 시리즈
BLUE, CORAL = "#7FB3D5", "#E8A798"   # 두 범주 비교
GRID = "#BDB8AE"

df_clean = pd.read_csv("data/cleaned_지하철_시간대별_승하차.csv")
daily = pd.read_csv("data/서울시 지하철호선별 역별 승하차 인원 정보.csv", encoding="cp949")
events = pd.read_csv("data/서울시 문화행사 정보.csv", encoding="cp949", encoding_errors="replace")

boarding_cols = [c for c in df_clean.columns if c.endswith("승차인원")]
df_clean["총승차인원"] = df_clean[boarding_cols].sum(axis=1)
df_clean["날짜"] = pd.to_datetime(df_clean["사용월"].astype(str), format="%Y%m")
daily["날짜"] = pd.to_datetime(daily["사용일자"].astype(str), format="%Y%m%d")

print(df_clean.shape, daily.shape, events.shape)
```

## 1. 그래프 3개 만들기

분석 질문과 관련 있는 그래프를 3개 이상 만드세요. 막대그래프, 선그래프, 히스토그램, 산점도 중 데이터에 맞는 것을 선택하면 됩니다.

### 그래프 1 — 월별 지하철 총승차인원 추이 (선그래프)

```python
monthly = df_clean.groupby("날짜")["총승차인원"].sum() / 1e6

fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(monthly.index, monthly.values, color=YELLOW, lw=2)
ax.axhline(monthly[:"2019-12"].mean(), color=GRID, ls="--", lw=1)
ax.text(monthly.index[-1], monthly[:"2019-12"].mean() + 2, "2015~2019 평균", fontsize=8, color="gray", ha="right")
ax.axvspan(pd.Timestamp("2020-02"), pd.Timestamp("2022-04"), color=GRID, alpha=0.15)
ax.text(pd.Timestamp("2021-03"), monthly.max(), "코로나19 기간", fontsize=8, color="gray", ha="center", va="top")
ax.set_title("월별 지하철 총승차인원 추이")
ax.set_xlabel("사용월")
ax.set_ylabel("총승차인원 (백만 명)")
plt.show()

print("2015~2019 월평균:", round(monthly[:"2019-12"].mean(), 1), "백만")
print("최저월:", monthly.idxmin().strftime("%Y-%m"), round(monthly.min(), 1), "백만")
print("2025~2026 월평균:", round(monthly["2025-01":].mean(), 1), "백만")
```

### 그래프 2 — 일별 총승차인원, 평일 vs 주말·공휴일 (막대그래프)

```python
holidays = pd.to_datetime(["2026-08-15", "2026-08-17"])   # 광복절, 대체공휴일
day_total = daily.groupby("날짜")["승차총승객수"].sum() / 1e6
is_off = (day_total.index.dayofweek >= 5) | day_total.index.isin(holidays)

fig, ax = plt.subplots(figsize=(10, 4))
ax.bar(day_total.index, day_total.values, color=[CORAL if o else BLUE for o in is_off], width=0.8)
ax.axhline(day_total[~is_off].mean(), color=GRID, ls="--", lw=1)
ax.set_title("일별 지하철 총승차인원 (평일 vs 주말·공휴일)")
ax.set_xlabel("날짜")
ax.set_ylabel("총승차인원 (백만 명)")
ax.xaxis.set_major_formatter(plt.matplotlib.dates.DateFormatter("%m/%d"))
ax.legend(handles=[plt.Rectangle((0, 0), 1, 1, color=BLUE), plt.Rectangle((0, 0), 1, 1, color=CORAL)],
          labels=["평일", "주말·공휴일"], fontsize=8, frameon=False)
plt.show()

print("평일 평균:", round(day_total[~is_off].mean(), 2), "백만 / 주말·공휴일 평균:", round(day_total[is_off].mean(), 2), "백만")
```

### 그래프 3 — 평소 대비 하차인원 급증 TOP 10 역·날짜 (가로 막대그래프)

역마다 평소 수준이 달라서 절대 인원이 아니라 "같은 요일유형(평일/휴일) 중앙값 대비 몇 배인가"(`하차배율`)로 비교하고,
서울시 문화행사 정보에 그 날짜·장소의 행사가 있는지 표시했다.

```python
st = daily.groupby(["역명", "날짜"])["하차총승객수"].sum().reset_index()
st["휴일"] = (st["날짜"].dt.dayofweek >= 5) | st["날짜"].isin(holidays)
st["하차배율"] = st["하차총승객수"] / st.groupby(["역명", "휴일"])["하차총승객수"].transform("median")

top = st[st["하차총승객수"] >= 8000].nlargest(10, "하차배율").copy()

# 문화행사 데이터에 같은 날짜, 역 이름이 들어간 장소가 있는지 확인 (행사명까지 보면 "대화" 같은 일반 단어가 잘못 걸림)
events["시작"] = pd.to_datetime(events["시작일"].str[:10], errors="coerce")
events["종료"] = pd.to_datetime(events["종료일"].str[:10], errors="coerce")
place_text = events["장소"].astype(str)

def has_event(row):
    keyword = row["역명"].split("(")[0]
    on_day = (events["시작"] <= row["날짜"]) & (events["종료"] >= row["날짜"])
    return (on_day & place_text.str.contains(keyword, regex=False)).any()

top["행사데이터"] = top.apply(has_event, axis=1)
top["라벨"] = top["역명"] + " " + top["날짜"].dt.strftime("%m/%d")
top[["역명", "날짜", "휴일", "하차총승객수", "하차배율", "행사데이터"]]
```

```python
plot_df = top.sort_values("하차배율")

fig, ax = plt.subplots(figsize=(8, 4.5))
bars = ax.barh(plot_df["라벨"], plot_df["하차배율"],
               color=[BLUE if e else CORAL for e in plot_df["행사데이터"]])
ax.axvline(1, color=GRID, ls="--", lw=1)
for b, n in zip(bars, plot_df["하차총승객수"]):
    ax.text(b.get_width() + 0.05, b.get_y() + b.get_height() / 2, f"{n:,}명", va="center", fontsize=7, color="gray")
ax.set_title("평소 대비 하차인원 급증 TOP 10 (역·날짜)")
ax.set_xlabel("하차배율 (같은 요일유형 중앙값 = 1)")
ax.set_ylabel("역 · 날짜")
ax.legend(handles=[plt.Rectangle((0, 0), 1, 1, color=BLUE), plt.Rectangle((0, 0), 1, 1, color=CORAL)],
          labels=["문화행사 데이터에 있음", "문화행사 데이터에 없음"], fontsize=8, frameon=False, loc="lower right")
plt.show()
```

### 그래프 4 — 여의나루역 21~23시 승차인원, 연도 × 월 (히트맵)

```python
yn = df_clean[df_clean["지하철역"] == "여의나루"].copy()
yn["야간승차"] = yn[["21시-22시 승차인원", "22시-23시 승차인원"]].sum(axis=1)
yn["연도"] = yn["날짜"].dt.year
yn["월"] = yn["날짜"].dt.month
heat = yn.pivot_table(index="연도", columns="월", values="야간승차", aggfunc="sum") / 1000

fig, ax = plt.subplots(figsize=(9, 4.5))
im = ax.imshow(heat.values, cmap="Blues", aspect="auto")
ax.set_xticks(range(12), [f"{m}월" for m in heat.columns])
ax.set_yticks(range(len(heat)), heat.index)
for i in range(heat.shape[0]):
    for j in range(heat.shape[1]):
        v = heat.values[i, j]
        if pd.notna(v):
            ax.text(j, i, f"{v:.0f}", ha="center", va="center", fontsize=7,
                    color="white" if v > heat.values[~pd.isna(heat.values)].max() * 0.6 else "black")
ax.spines[["left", "bottom"]].set_visible(False)
fig.colorbar(im, ax=ax, label="야간 승차 (천 명)")
ax.set_title("여의나루역 21~23시 승차인원 (연도 × 월)")
ax.set_xlabel("월")
ax.set_ylabel("연도")
plt.show()

print(yn.groupby("월")["야간승차"].mean().round().astype(int).to_string())
```

## 2. 그래프별 해석 작성

각 그래프가 무엇을 보여주는지, 어떤 패턴이 보이는지 적으세요.

```md
그래프 1: 월별 지하철 총승차인원 추이 (선그래프)
보여주는 내용: 2015년 1월 ~ 2026년 8월, 전 노선·전 역의 월별 총승차인원 합계.
해석: 2015~2019년에는 월 2.2억 명 안팎에서 안정적으로 움직이다가, 2020년 3월 1.4억 명으로 급락했다.
코로나19 기간(2020~2022 초) 내내 평소의 70~80% 수준에 머물렀고, 2023년 이후 회복했지만
2025~2026년 월평균(2.14억 명)도 아직 코로나 이전 평균(2.22억 명)에 조금 못 미친다.
5주차 피벗에서 본 "2020년에 전 노선이 동시에 꺾인 구간"의 정체는 대형 행사가 아니라 코로나19였다.
→ 행사 효과를 볼 때 2020~2022년은 비교 기준에서 빼거나 따로 다뤄야 한다.

그래프 2: 일별 지하철 총승차인원, 평일 vs 주말·공휴일 (막대그래프)
보여주는 내용: 2026-07-27 ~ 09-04 하루 단위 전체 승차인원. 파란색은 평일, 주황색은 주말·공휴일.
해석: 평일은 하루 약 776만 명, 주말·공휴일은 약 496만 명으로 휴일이 평일의 64% 수준이다.
8/15(광복절, 토)과 8/17(대체공휴일, 월)도 일요일과 거의 같은 수준으로 떨어진다.
요일 효과가 이렇게 크기 때문에, 행사 날짜의 승하차를 "전체 평균"과 비교하면 안 되고
같은 요일유형(평일끼리, 휴일끼리)과 비교해야 한다. 그래프 3의 하차배율을 이 기준으로 만든 이유다.

그래프 3: 평소 대비 하차인원 급증 TOP 10 (가로 막대그래프)
보여주는 내용: 역별로 "같은 요일유형의 중앙값 대비 하차인원 배율"이 가장 큰 역·날짜 10개.
색은 서울시 문화행사 정보에 같은 날짜·같은 장소의 행사가 등록돼 있는지 여부.
해석: 월드컵경기장(성산)역 8/9 하차가 평소 휴일의 6.7배(26,896명)로 가장 크게 튀었고,
같은 역이 8/5(4.1배), 8/15(3.5배)에도 TOP 10에 들었다. 대화역(킨텍스) 8/21~23도 사흘 연속 3배 안팎이다.
이런 급증은 경기장·전시장 앞 역에서, 특정 날짜에만 나타나므로 대형 행사의 흔적으로 볼 수 있다.
그런데 TOP 10 중 서울시 문화행사 데이터로 설명되는 건 대공원(서울대공원)과 독립문(광복절 영천시장 축제)
2건뿐이다. 가장 크게 튄 월드컵경기장은 공연·스포츠 경기 같은 민간 행사라 이 데이터에 없고,
대화역은 고양시라 서울시 데이터 범위 밖이다.

그래프 4: 여의나루역 21~23시 승차인원, 연도 × 월 (히트맵)
보여주는 내용: 한강공원 앞 여의나루역에서 밤 9~11시에 지하철을 탄 인원을 연도·월별로 색칠한 표.
해석: 4~10월이 진하고 11~2월은 2만 명대로 옅어서, 야외 활동 계절에 따라 밤 수요가 3~4배 차이 난다.
2020년 4월~2021년은 거의 겨울 수준으로 빠져 코로나 영향이 그대로 보인다.
반면 서울세계불꽃축제가 열리는 10월은 오히려 9월보다 낮은 해가 많다. 하루에 100만 명이 모이는 행사라도
월 합계로 묶으면 계절 효과에 묻혀 보이지 않는다는 뜻이고, 행사 효과는 일 단위 이상으로 봐야 잡힌다.
```

## 3. 최종 주장 후보 정리

최종 리포트에서 강조하고 싶은 핵심 주장을 2~3개 정리하세요.

```md
주장 후보 1: 대형 행사는 개최지 인근 역의 하차인원을 평소의 3~7배까지 끌어올린다.
근거: 그래프 3에서 월드컵경기장(성산)역 8/9 하차가 같은 요일유형 중앙값의 6.7배, 대화역(킨텍스)이 사흘 연속
약 3배였다. 급증이 경기장·전시장 앞 역, 특정 날짜에만 몰려 있어 평소 변동이 아니라 행사 수요로 보인다.

주장 후보 2: 행사 효과는 "같은 요일유형 + 일 단위" 기준선과 비교해야 보인다.
근거: 그래프 2에서 휴일은 평일의 64% 수준이라 요일만으로도 큰 차이가 생기고, 그래프 4에서 불꽃축제가 있는
10월이 월 합계로는 9월보다 낮게 나왔다. 그래프 1처럼 코로나19 같은 외부 충격도 있어서 2020~2022년은
기준선에서 빼야 한다.

주장 후보 3: 공공 문화행사 데이터만으로는 지하철 수요를 크게 흔드는 행사를 거의 설명하지 못한다.
근거: 그래프 3의 TOP 10 급증 중 서울시 문화행사 정보로 설명되는 건 2건뿐이었다. 가장 큰 급증(월드컵경기장)은
콘서트·스포츠 경기 같은 민간 행사라서, 공연 일정(KOPIS)·경기 일정 데이터를 추가해야 행사 영향을 제대로 측정할 수 있다.
```

---

# 2️⃣ 선택 과제

같은 데이터를 기준으로 그래프 종류를 2개 그려보고, 어떤 그래프가 더 적절한지 비교해보세요.

```python
summary = df_clean.groupby("호선명")["총승차인원"].mean().sort_values(ascending=False) / 1e4

fig, ax = plt.subplots(figsize=(10, 4))
summary.plot(kind="bar", ax=ax, color=YELLOW, width=0.8)
ax.set_title("호선별 평균 총승차인원 — 세로 막대")
ax.set_xlabel("호선")
ax.set_ylabel("평균 총승차인원 (만 명)")
plt.show()

fig, ax = plt.subplots(figsize=(7, 7))
summary.sort_values().plot(kind="barh", ax=ax, color=YELLOW, width=0.8)
ax.axvline(summary.mean(), color=GRID, ls="--", lw=1)
ax.set_title("호선별 평균 총승차인원 — 가로 막대")
ax.set_xlabel("평균 총승차인원 (만 명)")
ax.set_ylabel("호선")
plt.show()
```

```md
비교한 그래프 1: 호선별 평균 총승차인원 — 세로 막대그래프
비교한 그래프 2: 호선별 평균 총승차인원 — 가로 막대그래프 (+ 전체 평균 점선)
더 적절하다고 판단한 그래프: 가로 막대그래프
그 이유: 호선이 28개나 되고 "공항철도 1호선", "9호선2~3단계"처럼 이름이 길어서, 세로 막대에서는
x축 라벨을 90도로 눕혀야 하고 고개를 돌려 읽어야 한다. 가로 막대는 이름을 그대로 읽을 수 있고,
위에서 아래로 순위대로 내려가며 비교할 수 있다. 평균 점선을 넣으니 일산선까지가 평균 이상,
안산선부터 평균 이하라는 것도 한눈에 보인다. 항목 수가 적고(5개 이하) 이름이 짧을 때나
시간 순서가 있는 데이터일 때는 세로 막대가 더 자연스럽다.
```

---

# 3️⃣ 제출 체크리스트

- [✅] 그래프 3개 이상을 만들었다.
- [✅] 각 그래프의 제목과 축 이름을 작성했다.
- [✅] 그래프별 해석을 작성했다.
- [✅] 최종 주장 후보를 2개 이상 정리했다.

🎉 수고하셨습니다.  
다음 주에는 지금까지의 분석 결과를 하나의 최종 리포트로 완성합니다.
