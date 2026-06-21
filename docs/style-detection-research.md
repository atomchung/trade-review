# 研究:怎麼從交易紀錄「找風格」(style detection)

> 狀態:研究統整(獨立 research track,先研究後實作)。資料來源見文末。
> 目的:為 v2c 誠實閥找出第一個可實作的【風格】機械維。北極星不變:**找「不同投資哲學會給相反判決」的行為軸,不是給人貼分型標籤。** 風格是「從交易浮現的傾向」,不是「這個人是 X 型」。
> 方法:5 角度並行檢索 + 對抗式查核(動能/逆勢學術定義、處置效應、持有/換手、加碼、其他指標+方法論陷阱)。

---

## 0. 為什麼這份研究是 v2c 的關鍵前提

`docs/v2c-lens-selection.md` 已釘死:誠實閥要能觸發,前提是存在【風格】型的機械維,而**現有 5 維(出場/sizing/分散/持有/攤平)全部對映【普世】單元** → 閥結構上是空的。所以整條路徑的第一步是:**先建一個真【風格】維**(各派會分歧的),閥才有對象可判。

這份研究回答:**哪些行為訊號是真風格(哲學分歧)、哪些其實是普世(人人該守)、哪個最值得先做。**

---

## 1. 最重要的統整:每個訊號都要先拆「普世 vs 風格」

研究最一致的結論:**多數行為指標同時含普世成分和風格成分,混在一起判就會錯。** 拆法如下(這張表直接餵 v2c §5 的普世/風格軸):

| 行為訊號 | 普世成分(閥免疫,一律判) | 風格成分(閥適用,哲學分歧) | 對立的兩派 |
|---|---|---|---|
| **進場相對位置** | 幾乎沒有 → **最純的風格軸** | 追高(買在高點/突破)vs 抄底(買在回檔/低點) | 動能/順勢 ⟷ 價值/逆勢 |
| **處置效應 / ride-vs-cut** | 在「成本價/回本點」附近做決定(錨定)、為了不認賠而抱、只在 12 月認賠 | 贏家該奔跑還是該收(賣贏家方向) | 動能(讓利潤跑)⟷ 均值回歸(見好就收) |
| **加碼方向** | 金額逐次放大(martingale)、以回本為目標、無停損/無 thesis | 跌價是買點還是賣點(往下加) | 價值/逆勢(跌破內在值→加)⟷ 動能/趨勢(跌=錯→砍) |
| **換手率 turnover** | **淨成本洩漏(普世逆風)** | 僅極窄的「已證實技巧」例外 | (基本不是風格) |
| **持有期 holding period** | 幾乎沒有 | **風格標籤**(scalp..buy&hold);本身無對錯 | (分類用,非判決用) |
| **集中度 concentration** | 過度自信/風險代理(偏普世) | 弱次級訊號 | (不單獨當風格) |

> **設計含義**:閥只在「風格成分」上 fork;「普世成分」一律判、不進閥。所以一個維要當【風格】維,得能**把自己的普世成分和風格成分分開輸出**(例如加碼維:martingale/回本錨定 → 普世判;單純跌價往下加且非升額 → 風格 fork)。

---

## 2. 候選風格維逐項評估

每項列:能不能從 `(ticker, side, qty, price, date)` + 日線算、對漂移容錯、樣本需求、偏誤防護、哪兩派對立。

### 2.1 進場相對位置(追高 vs 抄底)— ★ 最值得先做

**學術依據(對立天然成立):**
- 動能:Jegadeesh & Titman 1993,過去 3–12 月贏家續贏(6/6 約 +1%/月);George & Hwang 2004「52 週高」更強——`PRILAG = 進場價 / 過去 252 日最高`,愈近 1 愈強,且**它的超額報酬長期不反轉**(不像 JT/DBT 會回吐),最適合當穩定錨。
- 逆勢:De Bondt & Thaler 1985,36 月形成期的輸家在未來 3–5 年反超;Jegadeesh 1990 短期(1 月)反轉——所以動能研究要**跳過最近 1 個月**(skip-month),否則量到的是反轉不是動能。

**計算(主訊號):**
- `52週高比` = 進場價 / 過去 252 日最高。逐筆算,投資人風格 = 所有 BUY 的中位數。參數少、最穩、不反轉 → **首選**。
- `形成期報酬(skip 月)` = `價_{t-21} / 價_{t-21-L} - 1`,L=126/252。正=追動能、負=抄逆勢。**符號就是判決**。
- (次)`區間百分位` N=60/252;(次)`距均線 z 分數` `(價-SMA)/σ`,最受波動/區間影響,只當佐證。

**對立判決(這就是閥要的岔路):** 52週高比近 1 = 買在高點 → **動能/順勢派稱讚(強者續強)**、**價值/逆勢派斥責(買貴了、長期會反轉)**。每個指標的「高」極都是一派褒、一派貶。

**陷阱:** ① **回看窗主導結論**——同一筆買單在 6 月窗是動能、1 月或 3 年窗可能是逆勢;**必須把 lookback 跟判決一起報**,否則判決無定義。② 沒 skip 月會把短期反轉誤當動能。③ 樣本:每筆給一個 (0,1) 百分位,σ≈0.29,要把 0.65(動能)和 0.50(中性)在 95% 分開約需 **n≳15 筆**,0.60 vs 0.50 需 **n≳35**;單檔集中會讓筆數不獨立、再放大。<20 筆視為弱讀。

### 2.2 處置效應 / ride-vs-cut — 普世與風格的混合體(最易誤判)

**普世(該幫所有人修):**
- 處置效應本體:`PGR = 已實現獲利 /(已實現獲利 + 帳面獲利)`、`PLR = 已實現虧損 /(已實現虧損 + 帳面虧損)`,`PGR > PLR` 即有偏(Odean 1998:PGR≈0.148、PLR≈0.098,賣贏家的傾向約賠家 1.5 倍)。Odean 證實這**不是**稅、再平衡、或理性均值回歸能解釋的(12 月才反轉 → 全年行為非稅務理性)。Frazzini 2006:這偏誤系統到能留下可套利的動能溢酬(>200bps/月)。
- 普世錯誤的具體簽名:**賣點剛好群聚在回本線**(Ben-David & Hirshleifer 的 V-shape:賣出機率隨盈虧幅度兩側上升,在零附近沒有跳升 → 不是「翻黑就跑」的純參考點偏好,而是信念修正)、**為了不認賠而抱輸家**、**只在 12 月認賠**。

**風格(各派合法對立):**
- 贏家「該奔跑還是該收」**真的因派而異**:趨勢/動能該讓贏家跑、砍輸家(股票會 trend,對股票宇宙是理性的);均值回歸/價值該把贏家修回權重、加碼輸家(賭反轉,對會回歸的資產理性)。**「對的不對稱」會隨資產的報酬自相關翻轉**(Frontiers 2023:專業交易者在均值回歸商品上呈處置效應、在趨勢股票上呈反處置效應,兩者都理性)。
- 所以**不知道投資人的 thesis 和資產的 trend/mean-revert 性質,就不能把高 PGR/PLR 讀成錯**。

**含義:** 出場維要拆兩半——「回本錨定 + 只在 12 月認賠 + 抱著惡化中的輸家」走普世判;「賣贏家的方向(跑 vs 收)」走風格 fork,且要對著投資人風格 + 資產性質算,用容差帶、別用裸 PGR/PLR。

### 2.3 加碼:金字塔 vs 攤平 — 真風格 fork + 可分離的普世錯誤

**風格 fork(真對立):** 跌價是買點還是賣點。趨勢/動能:往下加是大忌(「Average Up, Never Down」,只加贏家、砍輸家);價值/逆勢:跌破內在值往下加**同時提高期望報酬又降風險**(Graham 邊際安全)。同一份成交、相反判決——因為兩派作用在不同狀態變數(動能看價格本身、價值看價格 vs 內在值)。

**可分離的普世錯誤(不分派都該修):**
1. **逐次放大加碼金額**(martingale)= 數學上必爆,不是風格。
2. **觸發點是自己的成本/回本**(沉沒成本/處置)= 純偏誤,前一次買價對前瞻 thesis 無關。
3. **無預設停損/thesis 失效規則** = 把有界損失變無界。

**可計算的分離訊號**(走一遍持倉,對每筆 add 算):`price_vs_avg = 加碼價/當時均成本-1`(+金字塔 / -攤平);`size_ratio = 本次量/上次量`(>1 連續=升額紅旗);`time_gap` 規律度(低變異=DCA);`loss_only_fraction`(只在虧損買的比例≈1=攤平);`drawdown_at_buy`。
- 高信心普世錯誤 = **只在虧損 + 升額 + 成本錨定**三者並現(與該次是否剛好回本無關 → 防 outcome bias)。
- 非升額的跌價加碼 = **模稜兩可(風格 or 錯)**,不可自動判錯。

> 引擎已有 `classify_adds()`(疑似定投/凹單/待確認)正是這個方向的雛形;要做的是讓它**額外吐出一個風格 lean**(往上加=strength / 往下加=weakness),並把普世錯誤簽名與風格 fork 分開。

### 2.4 換手率 turnover vs 持有期 holding — 一個是普世成本、一個是風格標籤

- **turnover = 普世成本洩漏**:Barber & Odean 2000——交易最兇的家戶 11.4%/年 vs 大盤 17.9%、平均家戶 16.4%,但**毛報酬在各 turnover 分位幾乎持平、只有淨報酬隨 turnover 下滑** → 高換手不是選股差,是被摩擦成本/短期稅吃掉,**對所有人都是逆風**(算術,非哲學)。唯一例外:已證實的技巧尾(台灣當沖 >80% 賠錢但 <1% 持續獲利;部分高換手基金淨正)——要用**淨成本後績效證明**,不能用嘴宣稱。
- **holding period = 風格標籤**:scalp(秒)/day(分時)/swing(天週)/position(月年)/buy&hold(年),文獻只分類不排好壞。短持有**本身是風格不是病**,只有跟負淨績效綁在一起才是病。
- **穩健性**:turnover **更穩**——用「股數流量 / 平均持股」算,**與配對規則無關**(不必把買賣配對)、近乎不受股價漂移。holding period **與配對規則相關**(FIFO/LIFO/指定批給不同答案)、被部分成交切碎、右偏(用中位數別用平均)。
- **持有期離散度(同檔又當沖又長抱)**:本身**模稜兩可**——可能是紀律的多策略(核心+衛星),也可能是框架漂移(套牢就改口長期)。**裂解測試**:離散若與盈虧符號獨立(贏輸都當沖)=刻意多策略;若短持有群聚在贏家、長持有群聚在輸家 = 處置效應/框架壞掉。引擎現有 `dim_hold` 的「同檔不一致框架」正是這個,但要補這個 P&L-vs-duration 裂解測試才站得住。

### 2.5 其他候選(次級)

- **交易方向 vs 趨勢前報酬(順勢外推 vs 逆勢)**:最直接、最揭露意圖的 ex-ante 軸,直接對映動能 vs 價值(Lakonishok-Shleifer-Vishny;Chicago Fed 2023:散戶總體偏逆勢但異質)。其實是 2.1 的廣義版,可當聚合層訊號。
- **勝率 × 賠率 × 盈虧偏度(return signature)**:動能=低勝率(~20–40%)高賠率正偏;均值回歸=高勝率(~60–70%)低賠率負偏。理論分離強,但**是已實現結果指標、最受 outcome/look-ahead 偏誤污染**,只當佐證別當主訊號。
- **因子傾斜持續度(HML/growth/WML, Sharpe 1992 + Fama-French)**:基金上很持續、對映價值/成長/動能哲學;散戶較雜訊。
- **集中度/特質波動暴險**:主要當**過度自信/風險代理**(Goetzmann & Kumar:集中→換手更高、報酬更低),別單獨當風格,免得把過度自信的集中客誤標成「高信念價值投資人」。

---

## 3. 方法論護欄(任何 style detector 必守,否則退回「分型」老路)

研究最強硬的共識——這些是**硬約束**,不是 nice-to-have:

1. **最低樣本**:每投資人估計需 **≥30–50 round-trip**,有信心標籤前best ≥100。不足 → 回「資料不足」,不是猜一個。(對映引擎現有 `alpha_credible`、`low_conf` 的閘門做法)
2. **報不確定性**:每個指標附信賴區間/容差;**永遠不端硬分類「你是 X 型」**。
3. **只用 ex-ante 特徵**:用決策當下資訊(進場方向、趨勢前報酬、進場估值)分類,**絕不用已實現盈虧/這筆有沒有賺**判風格(防 outcome bias,Baron & Hershey 1988)。
4. **point-in-time 資料**:特徵只能用成交時點可得的資料;決策價與成交價分開;不可用事後價標進出場(防 look-ahead)。
5. **不做存活過濾**:要含未平倉、被放棄的輸家(bag-holding)、下市標的——這些最能診斷風格,濾掉會高估品質。
6. **漂移容差 / 穩定性測試**:把交易史前後對半,驗風格標籤穩不穩;不穩就 flag,別硬塞單一標籤(散戶風格存在但雜訊大、部分、頻率相依)。
7. **風格與風險分離**:把集中度/特質波動當控制變數,別讓它污染風格判定。

> 這些護欄跟本專案既有設計天然契合:「對股價漂移容錯、只斷言方向不斷言精確值」(見 `tests/`)、`alpha_credible` 雙閘門、「對事不對人」。新風格維直接沿用同一套紀律。

---

## 4. 排序建議:先做哪個風格維

1. **進場相對位置(2.1)— 強烈建議先做。** 最純的風格軸(幾乎無普世成分要拆)、學術依據最硬(George-Hwang 52週高不反轉)、直接對映 `compare_lenses` 既有的 `lean`(strength/weakness)、可從現有資料 + 日線算、漂移容錯(只報方向)。**且它依賴的價格對齊我們已經做好了**(`adjust_for_splits`,PR #13)——跨分割的進場百分位沒對齊會錯 10 倍。
2. **加碼風格 lean(2.3)— 第二。** 已有 `classify_adds` 雛形,擴成吐風格 lean + 分離普世錯誤簽名,投入小、對立真。
3. **出場 ride-vs-cut 的風格半(2.2)— 第三。** 價值高但最糾纏(普世/風格混合最深),要先有 thesis/資產性質輸入才站得住,留後。
4. turnover 維持當**普世成本**提示(不進閥);holding period 當**風格標籤 + 框架裂解測試**(不當判決)。

### 建議的落地 MVP(第一個可觸發的閥)
```
1. dim_entry_style(rows, px):算 52週高比 + 形成期報酬(skip 月),帶樣本閘門(<~20 筆→低信賴),
   回傳 {dim:"進場", lean:"strength|weakness|—", axis:"style", severity, n, lookback}
2. 確定性測試:合成價格 fixture(追高樣本→strength、抄底樣本→weakness),離線可重現
3. momentum / margin-of-safety 兩面 lens 補這維:axis:style + 對立 stance + lean(schema 見 v2c §11)
4. compare_lenses 接這維 → 端出第一個真岔路 = 誠實閥 MVP
```

---

## 5. 對引擎/鏡片的接線(總結)

- **引擎側**:新 `dim_entry_style` 照現有 `dim_*` 形狀(回 dict:dim/severity/triggered/tier + 數字欄位),沿用 `alpha_credible` 式樣本閘門、`adjust_for_splits` 的價格對齊、`tests/` 的「只斷言方向」容錯。
- **鏡片側**:這維標 `axis:"style"`(閥適用),各 lens 補對立 `stance`/`lean`。普世維仍 `axis:"universal"`(閥免疫)。
- **閥側**:`compare_lenses` 復活時,missing stance → 視為閥 OFF(非 aligned),counter-lens 用 valve 專用選法(固定 top_flaw、同維比、需 ≥1 面非-inverted)。

---

## 待驗 / 風險

- **進場百分位的可實作穩定度**:lookback 敏感 → 固定報 52週高比 + 形成期(L=252)雙主訊號,兩者不一致就標「橫跨橫期、模稜」不硬判。
- **樣本閘門門檻**:n≳15–35 是推導值(非引用),上線後用真實資料校。
- **資產 trend/mean-revert 性質**:ride-vs-cut 的風格判定需要它當輸入,目前沒有 → 所以 ride-vs-cut 排第三、先做進場。
- **散戶風格穩定度本就弱**:必做前後半穩定性測試,不穩就誠實標「風格未定」。

---

## 資料來源

**進場時機 / 動能 vs 逆勢**
- Jegadeesh & Titman 1993, *Returns to Buying Winners and Selling Losers* — https://www.bauer.uh.edu/rsusmel/phd/jegadeesh-titman93.pdf
- De Bondt & Thaler 1985, *Does the Stock Market Overreact?* — https://onlinelibrary.wiley.com/doi/10.1111/j.1540-6261.1985.tb05004.x
- George & Hwang 2004, *The 52-Week High and Momentum Investing* — https://www.bauer.uh.edu/tgeorge/papers/gh4-paper.pdf
- Jegadeesh 1990 短期反轉 / skip-month — https://alphaarchitect.com/quantitative-momentum-research-short-term-return-reversal/
- AQR, *Hold the Dip* (2025) — https://www.aqr.com/-/media/AQR/Documents/Alternative-Thinking/AQR-Alternative-Thinking---Hold-the-Dip.pdf
- JPMorgan Institute, returns-chasing vs dip-buying — https://www.jpmorganchase.com/institute/all-topics/financial-health-wealth-creation/returns-chasing-and-dip-buying-among-retail-investors

**處置效應 / ride-vs-cut**
- Odean 1998, *Are Investors Reluctant to Realize Their Losses?* — https://onlinelibrary.wiley.com/doi/abs/10.1111/0022-1082.00072
- Shefrin & Statman 1985 — https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1540-6261.1985.tb05002.x
- Frazzini 2006, *The Disposition Effect and Underreaction to News* — https://pages.stern.nyu.edu/~afrazzin/pdf/The%20Disposition%20Effect%20and%20Underreaction%20to%20news%20-%20Frazzini.pdf
- Ben-David & Hirshleifer 2012, V-shape — http://www.columbia.edu/~la2329/The%20V-shaped%20Disposition%20Effect.pdf
- *When the disposition effect proves to be rational* (Frontiers 2023) — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9996105/
- *Rational disposition effects: Theory and evidence* — https://www.sciencedirect.com/science/article/pii/S0378426623000821

**換手率 / 持有期**
- Barber & Odean 2000, *Trading Is Hazardous to Your Wealth* — https://onlinelibrary.wiley.com/doi/abs/10.1111/0022-1082.00226
- Barber, Lee, Liu & Odean, 台灣當沖技巧尾 — https://faculty.haas.berkeley.edu/odean/papers/day%20traders/The%20Cross-Section%20of%20Speculator%20Skill.pdf
- 換手-績效異質性 — https://www.sciencedirect.com/science/article/abs/pii/S0378426621000121

**加碼 / 金字塔 vs 攤平**
- Turtle/trend pyramiding — https://www.quantifiedstrategies.com/turtle-trading-strategy/
- Graham 邊際安全 — https://www.netnethunter.com/benjamin-graham-value-investing-principles/
- DCA vs averaging down — https://trendspider.com/learning-center/mastering-dollar-cost-averaging-averaging-up-and-averaging-down/
- Martingale 風險 — https://corporatefinanceinstitute.com/resources/career-map/sell-side/capital-markets/martingale-strategy/

**其他指標 / 方法論陷阱**
- Goetzmann & Kumar, *Equity Portfolio Diversification* — https://papers.ssrn.com/sol3/papers.cfm?abstract_id=627321
- Lakonishok, Shleifer & Vishny, *Contrarian Investment, Extrapolation, and Risk* — https://www.nber.org/system/files/working_papers/w4360/w4360.pdf
- Coval, Hirshleifer & Shumway, *Can Individual Investors Beat the Market?* — https://www.bus.umich.edu/pdf/mitsui/nttdocs/coval-shumway2.pdf
- Benhamou et al., *Testing Sharpe Ratio: Luck or Skill?*(小樣本)— https://arxiv.org/abs/1905.08042
- Baron & Hershey 1988, outcome bias — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12372742/
- 存活/前視偏誤 — https://www.quantifiedstrategies.com/survivorship-bias-in-backtesting/
- *Retail investors are not noise traders* (CEPR) — https://cepr.org/voxeu/columns/retail-investors-are-not-noise-traders

> **查核註記**:多數一手 PDF(SSRN/NBER/作者頁/AQR)對自動抓取回 403,部分精確數字(Odean PGR=0.148/PLR=0.098、Barber-Odean 11.4%/17.9%、Frazzini >200bps/月、小樣本 n 門檻)經多個獨立二手來源交叉確認、與經典引用一致,但若要逐字引用請回一手 PDF 核對。n≳15–35 樣本門檻為推導值非引用。
