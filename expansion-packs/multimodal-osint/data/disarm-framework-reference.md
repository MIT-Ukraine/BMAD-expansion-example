# DISARM Framework Reference

完整 DISARM (Disinformation Analysis and Risk Management) 框架參考，供 LLM 識別調查中觀察到的資訊操弄技術。

**來源**: https://github.com/DISARMFoundation/DISARMframeworks

---

## Framework Structure

```
Phase → Tactic → Technique → Sub-technique
P01   → TA01   → T0073    → T0073.001
```

---

## Phases (階段)

| ID  | Name    | Description                        |
| --- | ------- | ---------------------------------- |
| P01 | Plan    | 規劃階段：設想期望的結果和戰略     |
| P02 | Prepare | 準備階段：建立資產、發展內容和敘事 |
| P03 | Execute | 執行階段：實際操作和內容傳遞       |
| P04 | Assess  | 評估階段：評估效果以改進未來行動   |

---

## Tactics (戰術)

### P01 - Plan

| ID   | Name            | Description  |
| ---- | --------------- | ------------ |
| TA01 | Plan Strategy   | 規劃整體戰略 |
| TA02 | Plan Objectives | 規劃具體目標 |

### P02 - Prepare

| ID   | Name                     | Description                  |
| ---- | ------------------------ | ---------------------------- |
| TA05 | Microtarget              | 微目標定位，細分受眾         |
| TA06 | Develop Content          | 開發內容（文字、圖片、影片） |
| TA13 | Target Audience Analysis | 目標受眾分析                 |
| TA14 | Develop Narratives       | 發展敘事                     |
| TA15 | Establish Assets         | 建立資產（帳號、網站、網路） |
| TA16 | Establish Legitimacy     | 建立可信度                   |

### P03 - Execute

| ID   | Name                            | Description        |
| ---- | ------------------------------- | ------------------ |
| TA07 | Select Channels and Affordances | 選擇渠道和平台功能 |
| TA08 | Conduct Pump Priming            | 預熱操作           |
| TA09 | Deliver Content                 | 傳遞內容           |
| TA10 | Drive Offline Activity          | 推動線下活動       |
| TA17 | Maximise Exposure               | 最大化曝光         |
| TA18 | Drive Online Harms              | 推動線上傷害       |

### P04 - Assess

| ID   | Name                                   | Description        |
| ---- | -------------------------------------- | ------------------ |
| TA11 | Persist in the Information Environment | 持續存在於資訊環境 |
| TA12 | Assess Effectiveness                   | 評估效果           |

---

## Techniques (技術)

### TA01 - Plan Strategy

| ID    | Name                       | Detection Indicators |
| ----- | -------------------------- | -------------------- |
| T0073 | Determine Target Audiences | 目標受眾分析痕跡     |
| T0074 | Determine Strategic Ends   | 戰略目標設定         |

### TA02 - Plan Objectives

| ID    | Name     | Detection Indicators |
| ----- | -------- | -------------------- |
| T0075 | Dismiss  | 否認/忽視策略        |
| T0076 | Distort  | 扭曲事實             |
| T0077 | Distract | 轉移注意力           |
| T0078 | Dismay   | 製造恐慌             |
| T0079 | Divide   | 製造分裂             |

### TA05 - Microtarget

| ID        | Name                       | Detection Indicators |
| --------- | -------------------------- | -------------------- |
| T0072     | Segment Audiences          | 受眾細分             |
| T0072.001 | Geographic Segmentation    | 地理區域分類         |
| T0072.002 | Demographic Segmentation   | 人口統計分類         |
| T0072.003 | Economic Segmentation      | 經濟條件分類         |
| T0072.004 | Psychographic Segmentation | 心理特徵分類         |
| T0072.005 | Political Segmentation     | 政治傾向分類         |

### TA06 - Develop Content

| ID        | Name                                                     | Detection Indicators       |
| --------- | -------------------------------------------------------- | -------------------------- |
| T0082     | Develop New Narratives                                   | 新敘事開發                 |
| T0083     | Integrate Target Audience Vulnerabilities into Narrative | 利用受眾弱點               |
| T0084     | Reuse Existing Content                                   | 重複使用內容               |
| T0085     | Develop Text-Based Content                               | 文字內容生成               |
| T0085.001 | Develop AI-Generated Text                                | AI 生成文字 - 語言模式異常 |
| T0086     | Develop Image-Based Content                              | 圖片內容生成               |
| T0086.001 | Develop AI-Generated Images                              | AI 生成圖片 (Deepfakes)    |
| T0087     | Develop Video-Based Content                              | 影片內容生成               |
| T0087.001 | Develop AI-Generated Video                               | AI 生成影片 (Deepfakes)    |
| T0088     | Develop Audio-Based Content                              | 音訊內容生成               |
| T0088.001 | Develop AI-Generated Audio                               | AI 生成音訊                |
| T0089     | Obtain Private Documents                                 | 取得私人文件               |

### TA07 - Select Channels and Affordances

| ID        | Name                             | Detection Indicators           |
| --------- | -------------------------------- | ------------------------------ |
| T0104     | Social Networks                  | 社群網路操作                   |
| T0104.001 | Mainstream Social Networks       | 主流社群平台 - 多帳號協調操作  |
| T0104.002 | Niche Social Networks            | 小眾社群平台                   |
| T0104.003 | Private/Closed Networks          | 私人/封閉群組                  |
| T0104.004 | Interest-Based Networks          | 興趣導向社群                   |
| T0104.005 | Use Hashtags                     | 使用 Hashtags - 短時間大量使用 |
| T0104.006 | Create Dedicated Hashtag         | 創建專屬 Hashtag               |
| T0107     | Bookmarking and Content Curation | 書籤和內容策展                 |
| T0109     | Consumer Review Networks         | 消費者評論平台                 |
| T0110     | Formal Diplomatic Channels       | 正式外交渠道                   |
| T0111     | Traditional Media                | 傳統媒體                       |
| T0113     | Employ Commercial Analytic Firms | 商業分析公司                   |

### TA08 - Conduct Pump Priming

| ID    | Name                 | Detection Indicators |
| ----- | -------------------- | -------------------- |
| T0020 | Trial Content        | 測試內容             |
| T0039 | Bait Influencer      | 誘餌 KOL             |
| T0042 | Seed Kernel of Truth | 植入真實核心         |
| T0044 | Seed Distortions     | 植入扭曲資訊         |

### TA09 - Deliver Content

| ID    | Name                          | Detection Indicators            |
| ----- | ----------------------------- | ------------------------------- |
| T0114 | Deliver Ads                   | 投放廣告                        |
| T0115 | Post Content                  | 發布內容                        |
| T0116 | Comment or Reply on Content   | 留言回覆 - 留言時間、內容相似度 |
| T0117 | Attract Traditional Media     | 吸引傳統媒體                    |
| T0118 | Amplify Existing Narrative    | 放大現有敘事 - 多帳號同步放大   |
| T0119 | Cross-Posting                 | 跨平台發布 - 相同內容跨平台     |
| T0120 | Incentivize Sharing           | 激勵分享                        |
| T0121 | Manipulate Platform Algorithm | 操縱演算法 - 異常互動模式       |

### TA10 - Drive Offline Activity

| ID    | Name                           | Detection Indicators |
| ----- | ------------------------------ | -------------------- |
| T0057 | Organise Events                | 組織活動             |
| T0061 | Sell Merchandise               | 販售商品             |
| T0126 | Encourage Attendance at Events | 鼓勵參與活動         |
| T0127 | Physical Violence              | 實體暴力             |

### TA13 - Target Audience Analysis

| ID    | Name                                          | Detection Indicators |
| ----- | --------------------------------------------- | -------------------- |
| T0080 | Map Target Audience Information Environment   | 繪製目標受眾資訊環境 |
| T0081 | Identify Social and Technical Vulnerabilities | 識別社會和技術弱點   |

### TA14 - Develop Narratives

| ID    | Name                                  | Detection Indicators |
| ----- | ------------------------------------- | -------------------- |
| T0002 | Facilitate State Propaganda           | 促進國家宣傳         |
| T0003 | Leverage Existing Narratives          | 利用現有敘事         |
| T0004 | Develop Competing Narratives          | 發展競爭敘事         |
| T0022 | Leverage Conspiracy Theory Narratives | 利用陰謀論           |
| T0023 | Distort Facts                         | 扭曲事實             |

### TA15 - Establish Assets

| ID        | Name                               | Detection Indicators |
| --------- | ---------------------------------- | -------------------- |
| T0010     | Cultivate Ignorant Agents          | 培養不知情代理人     |
| T0014     | Prepare Fundraising Campaigns      | 準備募款活動         |
| T0091     | Recruit Malign Actors              | 招募惡意行為者       |
| T0092     | Build Network                      | 建立網路             |
| T0093     | Acquire/Recruit Network            | 取得/招募網路        |
| T0094     | Infiltrate Existing Networks       | 滲透現有網路         |
| T0095     | Develop Owned Media Assets         | 發展自有媒體資產     |
| T0096     | Leverage Content Farms             | 利用內容農場         |
| T0097     | Present Persona                    | 呈現人設             |
| T0097.100 | Individual Persona                 | 個人人設             |
| T0097.101 | Fake Expert Persona                | 假專家人設           |
| T0097.102 | Journalist Persona                 | 記者人設             |
| T0097.103 | Activist Persona                   | 活動家人設           |
| T0097.104 | Partisan Persona                   | 黨派人設             |
| T0097.105 | Institutional Persona              | 機構人設             |
| T0097.106 | Government Official Persona        | 政府官員人設         |
| T0097.200 | Institutional Persona              | 機構人設             |
| T0097.201 | NGO Persona                        | NGO 人設             |
| T0097.202 | News Outlet Persona                | 新聞媒體人設         |
| T0097.203 | Fact-Checking Organisation Persona | 事實查核組織人設     |
| T0097.204 | Think Tank Persona                 | 智庫人設             |
| T0097.205 | Business Persona                   | 企業人設             |
| T0097.206 | Government Institution Persona     | 政府機構人設         |
| T0097.207 | Hacktivist Group Persona           | 駭客組織人設         |
| T0097.208 | Social Cause Persona               | 社會議題人設         |
| T0097.209 | Religious Organisation Persona     | 宗教組織人設         |
| T0097.210 | Interest Group Persona             | 利益團體人設         |
| T0098     | Establish Inauthentic News Sites   | 建立假新聞網站       |
| T0100     | Co-Opt Trusted Sources             | 拉攏可信來源         |
| T0101     | Create Localised Content           | 創建本地化內容       |

### TA16 - Establish Legitimacy

| ID    | Name                                 | Detection Indicators     |
| ----- | ------------------------------------ | ------------------------ |
| T0015 | Create Hashtags and Search Artefacts | 創建 Hashtags 和搜尋痕跡 |
| T0045 | Use Fake Experts                     | 使用假專家               |
| T0046 | Use Search Engine Optimisation       | SEO 操作                 |

### TA17 - Maximise Exposure

| ID    | Name                                  | Detection Indicators          |
| ----- | ------------------------------------- | ----------------------------- |
| T0016 | Create Clickbait                      | 創建點擊誘餌                  |
| T0018 | Purchase Targeted Advertisements      | 購買目標廣告                  |
| T0049 | Flood Information Space               | 資訊洪水 - 短時間大量相似內容 |
| T0060 | Continue to Amplify                   | 持續放大                      |
| T0102 | Leverage Echo Chambers/Filter Bubbles | 利用同溫層/過濾泡泡           |
| T0122 | Direct Users to Alternative Platforms | 引導用戶到替代平台            |

### TA18 - Drive Online Harms

| ID    | Name                                                                    | Detection Indicators |
| ----- | ----------------------------------------------------------------------- | -------------------- |
| T0048 | Harass                                                                  | 騷擾                 |
| T0040 | Demand Insurmountable Proof                                             | 要求不可能的證明     |
| T0047 | Censor Social Media as Political Force                                  | 審查社群媒體         |
| T0066 | Degrade Adversary                                                       | 貶低對手             |
| T0124 | Suppress Opposition                                                     | 壓制反對聲音         |
| T0123 | Control Information Environment through Offensive Cyberspace Operations | 網路攻擊控制資訊環境 |

### TA11 - Persist in Information Environment

| ID    | Name                                            | Detection Indicators |
| ----- | ----------------------------------------------- | -------------------- |
| T0059 | Play the Long Game                              | 長期經營             |
| T0068 | Respond to Breaking News Event or Active Crisis | 回應突發新聞         |
| T0125 | Platform Filtering                              | 平台過濾             |

### TA12 - Assess Effectiveness

| ID    | Name                | Detection Indicators |
| ----- | ------------------- | -------------------- |
| T0029 | Online Polls        | 線上投票             |
| T0017 | Conduct Fundraising | 進行募款             |

---

## Common Detection Patterns

### Coordinated Behavior Indicators

適用技術: T0104, T0118, T0119, T0121

- 多帳號同時發布相似內容
- 異常的互動模式（同時按讚、分享）
- 跨平台同步操作
- 短時間內大量相同 hashtag 使用

### Content Manipulation Indicators

適用技術: T0049, T0084, T0085, T0086, T0087

- 內容高度相似或重複
- AI 生成內容特徵
- 大量轉貼原始內容
- 異常的內容產出速度

### Temporal Anomaly Indicators

適用技術: T0049, T0115, T0116

- 非正常時段發布高峰
- 突發性的內容爆發
- 規律性的發布時間模式
- 與事件時間點的可疑相關性

### Network Structure Indicators

適用技術: T0092, T0093, T0094, T0097

- 異常的追蹤/被追蹤比例
- 帳號建立時間集中
- 命名模式相似
- 互動網路結構異常

---

## Reference Links

- **Official Repository**: https://github.com/DISARMFoundation/DISARMframeworks
- **Interactive Framework**: https://disarmframework.herokuapp.com/
- **STIX Data**: https://github.com/DISARMFoundation/DISARMframeworks/tree/main/generated_files/DISARM_STIX
