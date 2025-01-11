<!--yml
category: 未分类
date: 2025-01-11 12:25:38
-->

# A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading

> 来源：[https://arxiv.org/html/2407.09546/](https://arxiv.org/html/2407.09546/)

Yuan Li, Bingqiao Luo¹¹footnotemark: 1, Qian Wang¹¹footnotemark: 1, Nuo Chen, Xu Liu, Bingsheng He
National University of Singapore
li.yuan@u.nus.edu, luo.bingqiao@u.nus.edu, qiansoc@nus.edu.sg
nuochen@u.nus.edu liuxu@comp.nus.edu.sg
hebs@comp.nus.edu.sg Equal contribution

###### Abstract

The utilization of Large Language Models (LLMs) in financial trading has primarily been concentrated within the stock market, aiding in economic and financial decisions. Yet, the unique opportunities presented by the cryptocurrency market, noted for its on-chain data’s transparency and the critical influence of off-chain signals like news, remain largely untapped by LLMs. This work aims to bridge the gap by developing an LLM-based trading agent, CryptoTrade, which uniquely combines the analysis of on-chain and off-chain data. This approach leverages the transparency and immutability of on-chain data, as well as the timeliness and influence of off-chain signals, providing a comprehensive overview of the cryptocurrency market. CryptoTrade incorporates a reflective mechanism specifically engineered to refine its daily trading decisions by analyzing the outcomes of prior trading decisions. This research makes two significant contributions. Firstly, it broadens the applicability of LLMs to the domain of cryptocurrency trading. Secondly, it establishes a benchmark for cryptocurrency trading strategies. Through extensive experiments, CryptoTrade has demonstrated superior performance in maximizing returns compared to traditional trading strategies and time-series baselines across various cryptocurrencies and market conditions. Our code and data are available at [https://anonymous.4open.science/r/CryptoTrade-Public-92FC/](https://anonymous.4open.science/r/CryptoTrade-Public-92FC/).

## 1 Introduction

Large Language Models (LLMs) have revolutionized financial decision-making and stock market prediction by excelling in tasks such as sentiment analysis Liang et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib16)) and explanation generation Pu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib25)). Specialized models like FinGPT and BloombergGPT Liu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib17)); Wu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib34)) demonstrate this capability. Recent research highlights their ability to interpret financial time-series and enhance cross-sequence reasoning Wei et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib30)); Yu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib39)); Zhang et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib40)); Zhao et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib41)); Yang et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib36)). Furthermore, the development of LLM-based trading agents like Sociodojo Cheng and Chin ([2024](https://arxiv.org/html/2407.09546v1#bib.bib5)) underscores the potential for innovating investment strategies.

![Refer to caption](img/c5b8646f3324f87fa06cf287b4dc443a.png)

Figure 1: CryptoTrade Framework. Our framework begins with the collection of various types of data, including on-chain transactions, market data, and off-chain data from multiple financial news sources. We extract on-chain statistics while summarizing off-chain news to provide comprehensive inputs for our agents’ analysis. We then deploy several LLM-based agents to make day-to-day trading decisions, utilizing a reflective mechanism to maximize total returns over different time periods.

However, the application of LLMs in the cryptocurrency market remains underexplored, yet this field holds great potential for future development for three main reasons. First, the cryptocurrency market is characterized by high market value, volatility, and uncertainty, which challenge traditional trading signals Drożdż et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib7)); Wei et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib31)). Second, LLMs have demonstrated their ability to understand and analyze financial markets by leveraging large volumes of multi-modal data, such as news and price information Wu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib34)). Third, the cryptocurrency market includes open-sourced on-chain data, such as gas prices and total transaction values, providing insights beyond just price movements Feichtinger et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib8)); Ren et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib26)). To bridge this gap, we introduce CryptoTrade. By integrating on-chain data, including market data and transaction records, with off-chain information like financial news, CryptoTrade leverages both dimensions to execute daily trading strategies, taking full advantage of the transparency of on-chain data and the immediacy of off-chain information. We detail the structure of CryptoTrade in [Figure 1](https://arxiv.org/html/2407.09546v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

CryptoTrade consists of a three-part framework. Initially, we collect data where on-chain details such as transactions and broader market data are aggregated alongside off-chain data from established financial news outlets like Bloomberg and Yahoo Finance. Following collection, we conduct statistical analyses to derive indicators such as moving averages and utilizing text processing for news summarization using GPT-3.5-turbo¹¹1[https://platform.openai.com/docs/models/gpt-3-5-turbo](https://platform.openai.com/docs/models/gpt-3-5-turbo). Finally, we enhance day-to-day decision-making with specialized analytical agents: market analyst agent evaluates market trends, news analyst agent interprets recent news impacts, and trading agent deliberates on investment actions. Concurrently, reflection agent reviews past performance, allowing CryptoTrade to refine its strategies to maximize returns.

Then, we conduct comprehensive experiments with CryptoTrade using GPT-4²²2[https://platform.openai.com/docs/models/gpt-4](https://platform.openai.com/docs/models/gpt-4), and GPT-4o³³3[https://platform.openai.com/docs/models/gpt-4o](https://platform.openai.com/docs/models/gpt-4o), evaluating its proficiency in making daily trading decisions for Bitcoin (BTC), Ethereum (ETH), and Solana (SOL). These three cryptocurrencies were selected for their prominence and market values of $134.14, $45.59, and $7.61 billion, respectively, as of June 2nd, 2024⁴⁴4[https://coinmarketcap.com/currencies/](https://coinmarketcap.com/currencies/). CryptoTrade significantly outperforms time-series baselines such as Informer Zhou et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib42)) and PatchTST Nie et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib20)), and achieves comparable performance to trading signals like Moving Average Convergence Divergence (MACD) Gencay ([1996](https://arxiv.org/html/2407.09546v1#bib.bib10)) in both return and sharpe ratio across bull, sideways, and bear market conditions. Notably, CryptoTrade operates in a zero-shot manner without fine-tuning based on validation sets, highlighting its potential for future applications. For instance, during the ETH bullish test period, the Buy and Hold strategy secured a 22.59% return, while CryptoTrade exceeded this by a remarkable 3%.

To summarize, we make the following three contributions:

*   •

    We introduce CryptoTrade, an innovative trading agent in the cryptocurrency domain, driven by LLMs. CryptoTrade is designed to generate optimized trading decisions specifically for the cryptocurrency market, setting a new benchmark in this field.

*   •

    We develop a comprehensive framework for cryptocurrency trading agents that encompasses the collection of both on-chain and off-chain data, along with the integration of a self-reflective component to enhance decision-making processes. This approach aggregates diverse information sources and establishes a new standard for data-driven trading strategies within the cryptocurrency domain.

*   •

    Through rigorous experiments, we present empirical evidence showcasing the efficacy of CryptoTrade compared to other baselines. CryptoTrade advances the frontier of cryptocurrency trading technologies and offers valuable insights for financial decision-making.

## 2 CryptoTrade Framework

This section details the components employed to develop the CryptoTrade agent, including data collection, market dynamics analysis, and agents development. [Figure 1](https://arxiv.org/html/2407.09546v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading") shows an overview of CryptoTrade.

### 2.1 Data Collection

The foundation of our methodology relies on a comprehensive collection of data from on-chain and off-chain sources, which is essential for making informed trading decisions in the cryptocurrency market. The data license is detailed in [Appendix A](https://arxiv.org/html/2407.09546v1#A1 "Appendix A License ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"). The data ethics are explained in [Appendix B](https://arxiv.org/html/2407.09546v1#A2 "Appendix B Data Ethics ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"). The data collection strategy is illustrated in [Figure 1](https://arxiv.org/html/2407.09546v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading")(a) and further detailed below:

*   •

    On-chain Data: We leverage historical data from CoinMarketCap⁵⁵5[https://coinmarketcap.com](https://coinmarketcap.com), which provides daily insights into prices, trading volumes, and market capitalization of various cryptocurrencies: BTC, ETH, SOL. This dataset forms the backbone of our market trend analysis, enabling us to decipher long-term trends and identify cycles in cryptocurrency valuations and investor behavior.

    Additionally, we incorporate detailed transaction statistics from on-chain activities. All blockchain transactions are transparent, traceable, and publicly accessible, achieved through securely linked blocks using cryptographic techniques Narayanan et al. ([2016](https://arxiv.org/html/2407.09546v1#bib.bib19)). As numerous prominent blockchain explorers provide tools for easy access to blockchain transaction data, we retrieve on-chain transaction data from the Dune Database⁶⁶6[https://dune.com/home](https://dune.com/home), a crypto analytics platform, and construct comprehensive statistics related to these transactions to include information on market dynamics. This includes comprehensive metrics such as daily number of transactions, number of active wallet, total value transferred, average gas price, and total gas consumed. These features are crucial for understanding the operational aspects of blockchains, such as network congestion times and cost efficiency, which directly impact trading strategies. Our daily collection of these metrics facilitates a nuanced analysis of market dynamics and liquidity, allowing for real-time adjustments to our trading algorithms based on current market conditions.

*   •

    Off-chain Data: We employ the Gnews API⁷⁷7[https://pypi.org/project/gnews/](https://pypi.org/project/gnews/) to systematically gather news articles related to each cryptocurrency. This tool enables us to access a wide array of sources through Google News, providing a comprehensive daily snapshot of market sentiment. Moreover, we particularly focus on filtering news from reputable financial and cryptocurrency-specific outlets such as Bloomberg, Yahoo Finance, and crypto.news⁸⁸8[https://crypto.news/](https://crypto.news/) to ensure the reliability and relevance of the information. The integration of analysis from these articles allows us to capture the market’s sentiment and response to developments, which is often a precursor to significant market movements.

By merging both on-chain data and off-chain news insights, our methodology offers a holistic view of the cryptocurrency market. This integration not only enhances our analytical capabilities but also significantly improves the precision of our trading decisions.

### 2.2 Market and News Analyst Agents

Upon collecting extensive on-chain and off-chain data, we analyze it through two key components of our CryptoTrade agent: (1) market analyst agent, (2) news analyst. By leveraging the capabilities of GPT-3.5-turbo, these analysts provide deep insights into the crypto market, enabling informed and strategic trading decisions.

Market Analyst Agent. The market analyst agent plays a crucial role in deciphering market dynamics through the statistical analysis of key trading signals from on-chain data, such as MA Gencay ([1996](https://arxiv.org/html/2407.09546v1#bib.bib10)), MACD Wang and Kim ([2018](https://arxiv.org/html/2407.09546v1#bib.bib29)), and Bollinger Bands Day et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib6)). Details of these trading signals are provided in [Appendix E](https://arxiv.org/html/2407.09546v1#A5 "Appendix E Baselines ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"). Armed with this information, the market analyst agent compiles reports on the market’s direction and momentum. An example is shown in [Figure 4](https://arxiv.org/html/2407.09546v1#A4.F4 "Figure 4 ‣ Appendix D Analysts Examples ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

News Analyst Agent. The news analyst agent is tasked with extracting and analyzing critical information from the latest news to assess the potential market impact of off-chain social hype. By sourcing news summaries from various trusted sources, the news analyst agent pinpoints relevant recent events and assesses the significance and implications of key topics, thus adding an extra dimension of insight. An example is provided in [Figure 5](https://arxiv.org/html/2407.09546v1#A4.F5 "Figure 5 ‣ Appendix D Analysts Examples ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

### 2.3 Trading Agent

Each day, the trading agent offers an investment suggestion based on reports from the market and news analyst agents. After analyzing the reports, the trading agent provides a concise rationale for its decisions. It also recommends allocating a certain portion of remaining cash to purchase cryptocurrency (with a range from $($$0$ to $1]$), selling a certain portion of owned cryptocurrency (with a range from $[-1$ to $0)$), or holding (neither buying nor selling). When a trading decision is made, a transaction fee is charged in proportion to the traded value. [Figure 6](https://arxiv.org/html/2407.09546v1#A4.F6 "Figure 6 ‣ Appendix D Analysts Examples ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading") illustrates an example of our trading agent’s operations.

### 2.4 Reflection Agent

The reflection agent reviews the trading agent’s recent activities to enhance future strategies. By analyzing the previous week’s prompts, decisions, and returns, the reflection agent identifies the most impactful information and the reasons behind its significance, providing feedback to the trading agent for future decisions. Consequently, CryptoTrade learns to focus on the most influential information for upcoming decisions. An example is illustrated in [Figure 7](https://arxiv.org/html/2407.09546v1#A4.F7 "Figure 7 ‣ Appendix D Analysts Examples ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

## 3 Experiments

In this section, we detail the experiments designed to evaluate the efficacy of our proprietary CryptoTrade agent in comparison to established baseline strategies within the trading domain.

### 3.1 Experimental Setup

Experiment Environments. We conduct all experiments using PyTorch on an NVIDIA GeForce RTX 3090 GPU. More details are in [Appendix C](https://arxiv.org/html/2407.09546v1#A3 "Appendix C Experimental Environment ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

Table 1: Dataset splits. Prices are in US dollars. In each split, the transaction days include the start date and exclude the end date. We evaluate the total profit on the end date.

| Type | Split | Start | End | Open | Close | Trend |
| --- | --- | --- | --- | --- | --- | --- |
| BTC | Validation | 2023-01-19 | 2023-03-13 | 20977.48 | 20628.03 | -1.67% |
| Test Bearish | 2023-04-12 | 2023-06-16 | 30462.48 | 25575.28 | -15.61% |
| Test Sideways | 2023-06-17 | 2023-08-25 | 26328.68 | 26163.68 | -0.83% |
| Test Bullish | 2023-10-01 | 2023-12-01 | 26967.40 | 37718.01 | 39.66% |
| ETH | Validation | 2023-01-13 | 2023-03-12 | 1417.13 | 1429.60 | 0.88% |
| Test Bearish | 2023-04-12 | 2023-06-16 | 1892.94 | 1664.98 | -12.24% |
| Test Sideways | 2023-06-20 | 2023-08-31 | 1734.79 | 1705.11 | -1.91% |
| Test Bullish | 2023-10-01 | 2023-12-01 | 1671.00 | 2051.76 | 22.59% |
| SOL | Validation | 2023-01-14 | 2023-03-12 | 18.29 | 18.24 | -0.27% |
| Test Bearish | 2023-04-12 | 2023-06-16 | 23.02 | 14.76 | -36.08% |
| Test Sideways | 2023-07-08 | 2023-08-31 | 21.49 | 20.83 | -3.23% |
| Test Bullish | 2023-10-01 | 2023-12-01 | 21.39 | 59.25 | 176.72% |

Datasets. To ensure our experiments are robust across different cryptocurrencies and market conditions, we base our study on a dataset covering several months, detailed in Table [1](https://arxiv.org/html/2407.09546v1#S3.T1 "Table 1 ‣ 3.1 Experimental Setup ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"). This dataset reflects the recent market performance of BTC, ETH, and SOL, presenting challenges in capturing market trends and volatility. We divide the dataset into validation and test sets, using the former to select model hyperparameters and the latter to evaluate model performance. We carefully select the test period after September 2021, the GPT-3.5’s knowledge cutoff date, to prevent data leakage. The dataset encompasses three market conditions: bull, sideways, and bear, allowing us to test the effectiveness of both the baselines and our model Baroiu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib2)); Cagan ([2024](https://arxiv.org/html/2407.09546v1#bib.bib3)), ensuring reliable and robust experimental results.

Evaluation Scheme. We initialize the trading agent with 1 million US dollars, split equally between cash and BTC/ETH/SOL, to enable potential profits from both buying and selling cryptocurrencies. At the end of the trading session, we use the following widely-accepted metrics: Return, Sharpe Ratio, Daily Return Mean, and Daily Return Std. This evaluation scheme ensures a rigorous and unbiased assessment of both baseline strategies and our CryptoTrade agent.

(1) Return measures the overall performance of the trading strategy, calculated using the formula $\frac{w^{end}-w^{start}}{w^{start}}$, where $w^{start}$ and $w^{end}$ represent the starting and ending net worth, respectively.

(2) Sharpe Ratio assesses the risk-adjusted return, using the formula $\frac{\bar{r}-{r_{f}}}{\sigma}$, where $\bar{r}$ is the mean of daily returns, $\sigma$ is the standard deviation of daily returns, and $r_{f}$ is the risk-free return, set to 0 following SocioDojo Cheng and Chin ([2024](https://arxiv.org/html/2407.09546v1#bib.bib5)).

(3) Daily Return Mean is the average of the daily returns over the trading period, providing insight into the typical daily performance of the trading strategy.

(4) Daily Return Std is the standard deviation of the daily returns, indicating the volatility and risk associated with the daily performance of the trading strategy.

Baseline Strategies. To benchmark the performance of our CryptoTrade agent, we compare it against widely recognized baseline strategies in the trading domain. We present these baselines and hyperparameters in [Appendix E](https://arxiv.org/html/2407.09546v1#A5 "Appendix E Baselines ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading").

### 3.2 Experimental Results

The performance comparison presented in [Table 2](https://arxiv.org/html/2407.09546v1#S3.T2 "Table 2 ‣ 3.2 Experimental Results ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"), [Table 3](https://arxiv.org/html/2407.09546v1#S3.T3 "Table 3 ‣ 3.2 Experimental Results ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"), [Table 4](https://arxiv.org/html/2407.09546v1#S3.T4 "Table 4 ‣ 3.2 Experimental Results ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading") between various trading strategies and our proposed CryptoTrade agent reveals significant insights into the efficacy of incorporating advanced data analysis techniques for cryptocurrency trading. The table highlights the returns and Sharpe Ratios for each method, where our CryptoTrade agent performs with outstanding percentage return and Sharpe Ratio. We outline the superiority of CryptoTrade in two key aspects:

Table 2: Performance of each strategy on BTC under Bull, Sideways, and Bear market conditions. For each market condition and each metric, the best result is highlighted in bold text and the runner-up result is underlined.

 | Strategy | Total Return | Daily Return | Sharpe Ratio |
|  | Bull | Sideways | Bear | Bull | Sideways | Bear | Bull | Sideways | Bear |
| Buy and hold | 39.66 | -0.83 | -15.61 | 0.56$\pm$2.23 | 0.00$\pm$1.74 | -0.24$\pm$2.07 | 0.25 | 0.00 | -0.11 |
| SMA | 22.58 | 3.65 | -21.74 | 0.35$\pm$1.89 | 0.06$\pm$1.21 | -0.36$\pm$1.25 | 0.18 | 0.05 | -0.29 |
| SLMA | 38.53 | -3.14 | -7.68 | 0.55$\pm$2.21 | -0.04$\pm$0.83 | -0.11$\pm$1.23 | 0.25 | -0.05 | -0.09 |
| MACD | 13.57 | -6.71 | -9.51 | 0.22$\pm$1.45 | -0.09$\pm$1.01 | -0.14$\pm$1.56 | 0.15 | -0.09 | -0.09 |
| Bollinger Bands | 2.97 | -3.19 | -1.17 | 0.05$\pm$0.32 | -0.04$\pm$0.87 | -0.02$\pm$0.51 | 0.15 | -0.05 | -0.03 |
| LSTM | 31.67 | -4.13 | -17.20 | 0.47$\pm$2.11 | -0.05$\pm$1.62 | -0.28$\pm$1.27 | 0.22 | -0.03 | -0.22 |
| Informer | 0.34 | -2.33 | -13.38 | 0.01$\pm$0.82 | -0.03$\pm$0.54 | -0.21$\pm$1.02 | 0.01 | -0.06 | -0.21 |
| AutoFormer | 14.73 | -4.90 | -12.72 | 0.24$\pm$1.65 | -0.07$\pm$1.15 | -0.20$\pm$1.13 | 0.14 | -0.06 | -0.18 |
| TimesNet | 2.84 | -5.12 | -13.64 | 0.05$\pm$1.06 | -0.07$\pm$1.10 | -0.22$\pm$1.04 | 0.05 | -0.06 | -0.21 |
| PatchTST | 1.79 | -5.02 | -21.94 | 0.03$\pm$0.71 | -0.07$\pm$0.57 | -0.37$\pm$1.05 | 0.04 | -0.13 | -0.35 |
| Ours(GPT-4) | 26.35 | -4.07 | -11.72 | 0.40$\pm$1.76 | -0.05$\pm$1.43 | -0.18$\pm$1.67 | 0.23 | -0.04 | -0.11 |
| Ours(GPT-4o) | 28.47 | -5.08 | -13.71 | 0.43$\pm$1.89 | -0.07$\pm$1.14 | -0.21$\pm$1.71 | 0.23 | -0.06 | -0.12 | 

Table 3: Performance of each strategy on ETH under Bull, Sideways, and Bear market conditions.

 | Strategy | Total Return (%) | Daily Return (%) | Sharpe Ratio |
|  | Bull | Sideways | Bear | Bull | Sideways | Bear | Bull | Sideways | Bear |
| Buy and Hold | 22.59 | -1.91 | -12.24 | 0.36$\pm$2.62 | -0.01$\pm$1.94 | -0.17$\pm$2.39 | 0.14 | -0.00 | -0.07 |
| SMA | 10.17 | -5.45 | -10.12 | 0.18$\pm$2.29 | -0.15$\pm$1.64 | -0.15$\pm$1.64 | 0.08 | -0.07 | -0.09 |
| SLMA | 5.20 | -2.62 | -15.90 | 0.11$\pm$2.37 | -0.03$\pm$1.08 | -0.24$\pm$1.86 | 0.05 | -0.03 | -0.13 |
| MACD | 7.72 | 0.77 | -12.15 | 0.13$\pm$1.22 | 0.02$\pm$1.43 | -0.18$\pm$1.56 | 0.10 | 0.01 | -0.12 |
| Bollinger Bands | 2.59 | 4.47 | -0.41 | 0.04$\pm$0.40 | 0.07$\pm$1.02 | 0.00$\pm$0.58 | 0.11 | 0.06 | -0.01 |
| LSTM | 22.12 | 1.27 | -13.22 | 0.36$\pm$2.59 | 0.02$\pm$1.11 | -0.19$\pm$2.36 | 0.14 | 0.15 | -0.08 |
| Informer | 14.55 | -4.74 | -11.49 | 0.23$\pm$1.54 | -0.06$\pm$1.45 | -0.17$\pm$1.65 | 0.15 | -0.04 | -0.10 |
| AutoFormer | 7.77 | -10.06 | -19.44 | 0.13$\pm$1.81 | -0.14$\pm$1.33 | -0.31$\pm$1.61 | 0.08 | -0.10 | -0.20 |
| TimesNet | 13.31 | -8.08 | -10.64 | 0.21$\pm$1.50 | -0.11$\pm$1.08 | -0.16$\pm$1.04 | 0.14 | -0.10 | -0.16 |
| PatchTST | 8.95 | -9.64 | -13.76 | 0.15$\pm$1.37 | -0.13$\pm$1.66 | -0.21$\pm$1.39 | 0.11 | -0.11 | -0.15 |
| Ours(GPT-4) | 25.72 | 0.72 | -13.72 | 0.41$\pm$2.45 | 0.03$\pm$1.67 | -0.21$\pm$2.02 | 0.17 | 0.02 | -0.10 |
| Ours(GPT-4o) | 25.47 | -6.59 | -15.35 | 0.40$\pm$2.25 | -0.07$\pm$1.81 | -0.23$\pm$2.16 | 0.18 | -0.04 | -0.11 | 

Table 4: Performance of each strategy on SOL under Bull, Sideways, and Bear market conditions.

 | Strategy | Total Return (%) | Daily Return (%) | Sharpe Ratio |
|  | Bull | Sideways | Bear | Bull | Sideways | Bear | Bull | Sideways | Bear |
| Buy and Hold | 176.72 | -3.23 | -36.08 | 1.83$\pm$6.00 | 0.01$\pm$3.92 | -0.61$\pm$3.45 | 0.30 | 0.00 | -0.18 |
| SMA | 119.37 | -0.62 | 1.04 | 1.43$\pm$5.67 | 0.03$\pm$3.06 | 0.02$\pm$0.10 | 0.25 | 0.01 | 0.16 |
| SLMA | 169.98 | 6.22 | -8.11 | 1.78$\pm$5.93 | 0.16$\pm$3.23 | -0.11$\pm$1.88 | 0.30 | 0.05 | -0.06 |
| MACD | 23.25 | -9.78 | -21.07 | 0.35$\pm$1.76 | -0.16$\pm$2.38 | -0.33$\pm$2.44 | 0.20 | -0.07 | -0.13 |
| Bollinger Bands | 2.92 | -0.46 | -21.69 | 0.05$\pm$0.35 | 0.00$\pm$1.23 | -0.35$\pm$1.75 | 0.13 | -0.00 | -0.20 |
| LSTM | 144.69 | -3.56 | -36.75 | 1.61$\pm$5.69 | 0.01$\pm$3.90 | -0.63$\pm$3.43 | 0.28 | 0.00 | -0.18 |
| Informer | 41.85 | -6.55 | -26.13 | 0.58$\pm$1.90 | -0.10$\pm$2.00 | -0.43$\pm$2.36 | 0.31 | -0.05 | -0.18 |
| AutoFormer | 35.86 | -6.17 | -23.56 | 0.51$\pm$1.97 | -0.10$\pm$1.90 | -0.38$\pm$2.35 | 0.26 | -0.05 | -0.16 |
| TimesNet | 45.28 | -10.63 | -21.60 | 0.64$\pm$2.66 | -0.18$\pm$2.01 | -0.35$\pm$1.75 | 0.24 | -0.09 | -0.20 |
| PatchTST | 18.45 | -7.10 | -27.86 | 0.29$\pm$1.57 | -0.11$\pm$1.98 | -0.46$\pm$2.49 | 0.18 | -0.06 | -0.19 |
| Ours(GPT-4) | 99.84 | -2.16 | -19.55 | 1.24$\pm$4.53 | 0.01$\pm$3.33 | -0.31$\pm$2.35 | 0.27 | 0.00 | -0.13 |
| Ours(GPT-4o) | 115.18 | 3.09 | -16.32 | 1.38$\pm$4.98 | 0.11$\pm$3.31 | -0.25$\pm$2.35 | 0.28 | 0.03 | -0.10 | 

Superior Performance under Different Market Conditions. Remarkably, even without fine-tuning, CryptoTrade outperforms Transformer-based time-series baselines in most bases, demonstrating the robust capabilities of LLMs. Additionally, its performance is comparable to traditional trading signals like Buy and Hold and MACD, further validating the potential of LLM-based approaches. For instance, CryptoTrade (GPT-4o) excels in all metrics under ETH’s bull market by 3% in total return and sharpe ratio. While CryptoTrade (GPT-4o) may not always be the top performer in every scenario, it consistently surpasses more than half of the trading signals across different market conditions, even without fine-tuning. This highlights the effectiveness and versatility of CryptoTrade in leveraging LLMs to navigate the complexities of the cryptocurrency market.

![Refer to caption](img/9a99c3aa30d7431da2d855c30d3a56ce.png)

Figure 2: Significant profitable periods exploited by the CryptoTrade agent. The yellow line shows the daily opening prices of Ethereum in US dollars. The blue line tracks the daily positions, indicating the amount of Ethereum possessed on each day. The blue dots denote trading decisions when the agent largely alters its position by trading Ethereum. The red dots represent the corresponding trading prices. The agent successfully forecasts price changes, securing substantial profits through low-price purchases and high-price sales.

Successful Trend Predictions. We draw [Figure 2](https://arxiv.org/html/2407.09546v1#S3.F2 "Figure 2 ‣ 3.2 Experimental Results ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading") to demonstrate the correlation between Ethereum’s opening prices and the positions held by the CryptoTrade agent, with the yellow and blue lines representing daily opening prices and Ethereum positions, respectively. The observed fluctuations highlight the market’s volatility, while the alignment between position adjustments and price movements showcases the agent’s proficiency in anticipating market trends. Unlike the static Buy and Hold strategy, CryptoTrade adopts a dynamic approach, optimizing trades based on market analysis—purchasing at lower prices and selling at peaks. This strategic adaptability, especially evident during shaded periods of preemptive position changes in anticipation of price shifts, underscores the agent’s capacity for risk management and its adeptness at leveraging market volatility for profit, marking a significant advancement over traditional trading strategies.

### 3.3 Ablation Study

The ablation study presented in [Table 5](https://arxiv.org/html/2407.09546v1#S3.T5 "Table 5 ‣ 3.3 Ablation Study ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading") critically examines the individual components of the prompt used by the CryptoTrade (GPT-4o) agent. By systematically removing key elements from the full prompt and observing the impact on percentage return and Sharpe ratio during a bull market for ETH, we can identify the contribution of each component to the overall performance of the trading strategy.We highlight the following two insights from the results:

Table 5: Ablation study on prompt components of the CryptoTrade agent. Base prompt encompasses necessary context including trading rules, valid action space, current cash and ETH holdings, and recent ETH prices.

| Prompt Components | Return (%) | Sharpe Ratio |
| --- | --- | --- |
| Full | 28.47 | 0.23 |
| w/o Reflection | 17.14 | 0.06 |
| w/o News | 19.69 | 0.06 |
| w/o TxnStats | 12.70 | 0.05 |
| w/o Technical | 17.27 | 0.05 |
| Base | 8.40 | 0.03 |

Superiority of the Full Prompt. The full prompt significantly outshines all other configurations with reduced components. The advantage of employing a full prompt over all deducted variants is rooted in the integration of diverse data sources. The full prompt encompasses the comprehensive price data, news analysis, technical indicators, on-chain transaction statistics, and reflective analysis to offer a holistic view of the market. This comprehensive approach allows the CryptoTrade agent to leverage a wide array of information, enabling it to navigate the complexities of the cryptocurrency market with more nuanced and informed trading decisions.

Advantage of Crypto Transaction Statistics. The omission of Ethereum transaction statistics results in a significant decrease of the outcome by around 16%, underscoring the indispensable role of on-chain statistics in enhancing trading strategies. This observation highlights the necessity of integrating on-chain transaction data, revealing its unique value in enriching the decision-making process in the cryptocurrency trading tasks.

### 3.4 Case Study

![Refer to caption](img/ab8bcb0b6271f73f2d3c2ae0a8f1f2bd.png)

Figure 3: Case study of CryptoTrade’s actions in response to news reports on early rumor and the actual event of Bitcoin ETF approval, which takes place on Jan 11, 2024\. The red circles denote the trading prices. The agent successfully benefits from a "buy the rumor, sell the news" strategy.

To assess the adaptability and responsiveness of the CryptoTrade agent, we conduct a case study focusing on its responsive actions in the context of the cryptocurrency market’s major events, illustrated in [Figure 3](https://arxiv.org/html/2407.09546v1#S3.F3 "Figure 3 ‣ 3.4 Case Study ‣ 3 Experiments ‣ A Reflective LLM-based Agent to Guide Zero-shot Cryptocurrency Trading"). It reveals that CryptoTrade’s strategy aligns with the "buy the rumor, sell the news" principle, effectively capitalizing on early signs of the Bitcoin ETF approval event, a scenario known to trigger market rallies due to speculative trading. By entering the market early, CryptoTrade secures positions at lower costs ahead of the rally.

As the approval of the Bitcoin ETF becomes a reality, the sentiment reaches a crescendo, resulting in inflated asset prices due to heightened demand. CryptoTrade, adhering to its strategic motivation, takes this peak as an optimal point to sell, which is validated in the subsequent decline in the Ethereum price. This strategic exit allows CryptoTrade to realize gains before the market adjusted to the new equilibrium, which results in a price pullback as early speculators take profits and the market sentiment normalizes.

To sum up, CryptoTrade’s provident actions underscore the delicate balance between foresight and timing in trading strategies. This case study demonstrates that an informed and timely response to market signals — both rumors and confirmed news — can yield advantageous outcomes. It also highlights the CryptoTrade agent’s understanding of market psychology and its ability to translate this into profitable trading decisions.

## 4 Related Work

LLMs for Economics and Financial Decisions Recent advancements in LLMs have significantly influenced economics and financial decision-making. Specialized LLMs like FinGPT, BloombergGPT, FinMA Liu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib17)); Wu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib34)); Xie et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib35)) are tailored for finance, handling tasks such as sentiment analysis, entity recognition, and question-answering. Another research direction uses LLMs for financial time-series forecasting. A notable contribution by Yu et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib39)) employed zero-shot or few-shot inference with GPT-4 and instruction-based fine-tuning with LlaMA to enhance cross-sequence reasoning and multi-modal signal integration. Additionally, the development of LLM-based agents for financial trading has gained attention. Sociodojo Cheng and Chin ([2024](https://arxiv.org/html/2407.09546v1#bib.bib5)) created analytical agents for stock portfolio management, showing the potential for generating "hyperportfolios." Despite these advancements, the focus has largely been on the stock market Koa et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib13)); Chen et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib4)), leaving a gap in the exploration of the cryptocurrency market where the on-chain data is approachable and with much information. Our work aims to address this gap by leveraging both on-chain and off-chain data to navigate the dynamic cryptocurrency market.

Time-Series Forecasting for Financial Markets Time-series forecasting has long been a cornerstone of research in economics and financial markets. Early studies focused on predicting stock market prices using methodologies such as machine learning Leung et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib15)); Patel and Yalamalle ([2014](https://arxiv.org/html/2407.09546v1#bib.bib22)), reinforcement learning Lee ([2001](https://arxiv.org/html/2407.09546v1#bib.bib14)), and traditional time-series models Herwartz ([2017](https://arxiv.org/html/2407.09546v1#bib.bib11)). The Long Short-Term Memory (LSTM) model has emerged as particularly influential Sunny et al. ([2020](https://arxiv.org/html/2407.09546v1#bib.bib28)) for its capability to process and analyze time-series data. With the rise of blockchain technology and cryptocurrencies, these techniques have been extended to crypto assets Khedr et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib12)). Recent research has evaluated the impact of various predictors on cryptocurrency pricing and returns, using both on-chain data—such as historical transactions and market volume Ferdiansyah et al. ([2019](https://arxiv.org/html/2407.09546v1#bib.bib9))—and off-chain factors like social media trends and news sentiment Abraham et al. ([2018](https://arxiv.org/html/2407.09546v1#bib.bib1)); Pang et al. ([2019](https://arxiv.org/html/2407.09546v1#bib.bib21)). These studies underscore the effectiveness of integrating diverse data sources for forecasting the volatile dynamics of the cryptocurrency market. Apart from above, Transformer-based models have shown particular promise in this area, with state-of-the-art models like Informer Zhou et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib42)), AutoFormer Wu et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib33)), PatchTST Nie et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib20)), and TimesNet Wu et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib32)) further advancing time-series forecasting.

Self-Reflective Language Agents The Self-Refine framework introduces an advanced approach for autonomous advancement through self-evaluation and iterative self-improvement Madaan et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib18)). This approach, along with efforts to automatically refine prompts Pryzant et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib24)); Ye et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib37)) and provide automated feedback to enhance reasoning capabilities Paul et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib23)), marks significant progress in the field. Notably, the "Reflexion" framework by Shinn et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib27)) revolutionizes the reinforcement of language agents by utilizing linguistic feedback and reflective text within an episodic memory buffer, diverging from traditional weight update methods. These advancements highlight the potential for LLMs to learn from their errors and evolve through self-reflection. Despite these developments, there is still untapped potential in applying self-reflective language agents to financial decision-making, particularly in cryptocurrency markets. This work aims to bridge that gap by investigating the application of self-reflective mechanisms to enhance financial decision-making processes in cryptocurrency trading.

## 5 Conclusion

We propose the CryptoTrade agent, an innovative approach to cryptocurrency trading that leverages advanced data analysis and LLMs. By integrating both on-chain and off-chain data, along with a self-reflective component, the CryptoTrade agent demonstrates a sophisticated understanding of market dynamics and achieves relatively high returns in cryptocurrency trading. Our comprehensive experiments comparing the CryptoTrade agent to traditional trading strategies and time-series models reveal its superior ability to navigate the volatile cryptocurrency market, consistently achieving relatively high returns on investment under different market conditions. This research underscores the significant potential of LLM-driven strategies in enhancing trading performance and sets a new benchmark for cryptocurrency trading with LLMs.

## 6 Limitations

One limitation of the current CryptoTrade framework is the reliance on a relatively limited dataset. To address this, we plan to enrich the dataset with additional off-chain data. Another limitation is the frequency of trading actions, which is currently set to day-to-day. We aim to refine this to hour-to-hour or minute-to-minute intervals to further optimize returns in the cryptocurrency market. Additionally, we have identified that the lack of fine-tuning for the LLMs using the validation set may be a significant factor behind the LLM-based agents’ underperformance compared to traditional trading signals. To improve the reliability of our forecasts, we intend to fine-tune the LLMs with the validation set.

## 7 Broader Impact

One potential broader impact of our research is the risk that individuals may follow the trading strategies we provide and subsequently incur financial losses. It is important to emphasize that these strategies are intended for academic research only. CryptoTrade is not for investment recommendations.

## References

*   Abraham et al. (2018) Jethin Abraham, Daniel Higdon, John Nelson, and Juan Ibarra. 2018. Cryptocurrency price prediction using tweet volumes and sentiment analysis. *SMU Data Science Review*, 1(3):1.
*   Baroiu et al. (2023) Alexandru Costin Baroiu, Vlad Diaconita, and Simona Vasilica Oprea. 2023. Bitcoin volatility in bull vs. bear market-insights from analyzing on-chain metrics and twitter posts. *PeerJ Computer Science*, 9:e1750.
*   Cagan (2024) Michele Cagan. 2024. *Stock Market 101: From Bull and Bear Markets to Dividends, Shares, and Margins—Your Essential Guide to the Stock Market*. Simon and Schuster.
*   Chen et al. (2023) Zihan Chen, Lei Nico Zheng, Cheng Lu, Jialu Yuan, and Di Zhu. 2023. Chatgpt informed graph neural network for stock movement prediction. *arXiv preprint arXiv:2306.03763*.
*   Cheng and Chin (2024) Junyan Cheng and Peter Chin. 2024. [Sociodojo: Building lifelong analytical agents with real-world text and time series](https://openreview.net/forum?id=s9z0HzWJJp). In *The Twelfth International Conference on Learning Representations*.
*   Day et al. (2023) Min-Yuh Day, Yirung Cheng, Paoyu Huang, and Yensen Ni. 2023. The profitability of bollinger bands trading bitcoin futures. *Applied Economics Letters*, 30(11):1437–1443.
*   Drożdż et al. (2023) Stanisław Drożdż, Jarosław Kwapień, and Marcin Wątorek. 2023. What is mature and what is still emerging in the cryptocurrency market? *Entropy*, 25(5):772.
*   Feichtinger et al. (2023) Rainer Feichtinger, Robin Fritsch, Yann Vonlanthen, and Roger Wattenhofer. 2023. The hidden shortcomings of (d) aos–an empirical study of on-chain governance. In *International Conference on Financial Cryptography and Data Security*, pages 165–185\. Springer.
*   Ferdiansyah et al. (2019) Ferdiansyah Ferdiansyah, Siti Hajar Othman, Raja Zahilah Raja Md Radzi, Deris Stiawan, Yoppy Sazaki, and Usman Ependi. 2019. A lstm-method for bitcoin price prediction: A case study yahoo finance stock market. In *2019 international conference on electrical engineering and computer science (ICECOS)*, pages 206–210\. IEEE.
*   Gencay (1996) Ramazan Gencay. 1996. Non-linear prediction of security returns with moving average rules. *Journal of Forecasting*, 15(3):165–174.
*   Herwartz (2017) Helmut Herwartz. 2017. Stock return prediction under garch—an empirical assessment. *International Journal of Forecasting*, 33(3):569–580.
*   Khedr et al. (2021) Ahmed M Khedr, Ifra Arif, Magdi El-Bannany, Saadat M Alhashmi, and Meenu Sreedharan. 2021. Cryptocurrency price prediction using traditional statistical and machine-learning techniques: A survey. *Intelligent Systems in Accounting, Finance and Management*, 28(1):3–34.
*   Koa et al. (2024) Kelvin JL Koa, Yunshan Ma, Ritchie Ng, and Tat-Seng Chua. 2024. Learning to generate explainable stock predictions using self-reflective large language models. In *Proceedings of the ACM on Web Conference 2024*, pages 4304–4315.
*   Lee (2001) Jae Won Lee. 2001. Stock price prediction using reinforcement learning. In *ISIE 2001\. 2001 IEEE International Symposium on Industrial Electronics Proceedings (Cat. No. 01TH8570)*, volume 1, pages 690–695\. IEEE.
*   Leung et al. (2021) Edward Leung, Harald Lohre, David Mischlich, Yifei Shea, and Maximilian Stroh. 2021. The promises and pitfalls of machine learning for predicting stock returns. *The Journal of Financial Data Science*.
*   Liang et al. (2022) Percy Liang, Rishi Bommasani, Tony Lee, Dimitris Tsipras, Dilara Soylu, Michihiro Yasunaga, Yian Zhang, Deepak Narayanan, Yuhuai Wu, Ananya Kumar, et al. 2022. Holistic evaluation of language models. *arXiv preprint arXiv:2211.09110*.
*   Liu et al. (2023) Xiao-Yang Liu, Guoxuan Wang, and Daochen Zha. 2023. Fingpt: Democratizing internet-scale data for financial large language models. *arXiv preprint arXiv:2307.10485*.
*   Madaan et al. (2024) Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, et al. 2024. Self-refine: Iterative refinement with self-feedback. *Advances in Neural Information Processing Systems*, 36.
*   Narayanan et al. (2016) Arvind Narayanan, Joseph Bonneau, Edward Felten, Andrew Miller, and Steven Goldfeder. 2016. *Bitcoin and cryptocurrency technologies: a comprehensive introduction*. Princeton University Press.
*   Nie et al. (2022) Yuqi Nie, Nam H Nguyen, Phanwadee Sinthong, and Jayant Kalagnanam. 2022. A time series is worth 64 words: Long-term forecasting with transformers. *arXiv preprint arXiv:2211.14730*.
*   Pang et al. (2019) Yan Pang, Ganeshkumar Sundararaj, and Jiewen Ren. 2019. Cryptocurrency price prediction using time series and social sentiment data. In *Proceedings of the 6th IEEE/ACM International Conference on Big Data Computing, Applications and Technologies*, pages 35–41.
*   Patel and Yalamalle (2014) Mayankkumar B Patel and Sunil R Yalamalle. 2014. Stock price prediction using artificial neural network. *International Journal of Innovative Research in Science, Engineering and Technology*, 3(6):13755–13762.
*   Paul et al. (2023) Debjit Paul, Mete Ismayilzada, Maxime Peyrard, Beatriz Borges, Antoine Bosselut, Robert West, and Boi Faltings. 2023. Refiner: Reasoning feedback on intermediate representations. *arXiv preprint arXiv:2304.01904*.
*   Pryzant et al. (2023) Reid Pryzant, Dan Iter, Jerry Li, Yin Tat Lee, Chenguang Zhu, and Michael Zeng. 2023. Automatic prompt optimization with" gradient descent" and beam search. *arXiv preprint arXiv:2305.03495*.
*   Pu et al. (2023) Xiao Pu, Mingqi Gao, and Xiaojun Wan. 2023. Summarization is (almost) dead. *arXiv preprint arXiv:2309.09558*.
*   Ren et al. (2023) Kunpeng Ren, Nhut-Minh Ho, Dumitrel Loghin, Thanh-Toan Nguyen, Beng Chin Ooi, Quang-Trung Ta, and Feida Zhu. 2023. Interoperability in blockchain: A survey. *IEEE Transactions on Knowledge and Data Engineering*.
*   Shinn et al. (2024) Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. 2024. Reflexion: Language agents with verbal reinforcement learning. *Advances in Neural Information Processing Systems*, 36.
*   Sunny et al. (2020) Md Arif Istiake Sunny, Mirza Mohd Shahriar Maswood, and Abdullah G Alharbi. 2020. Deep learning-based stock price prediction using lstm and bi-directional lstm model. In *2020 2nd novel intelligent and leading emerging sciences conference (NILES)*, pages 87–92\. IEEE.
*   Wang and Kim (2018) Jian Wang and Junseok Kim. 2018. Predicting stock price trend using macd optimized by historical volatility. *Mathematical Problems in Engineering*, 2018:1–12.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems*, 35:24824–24837.
*   Wei et al. (2023) Yu Wei, Yizhi Wang, Brian M Lucey, and Samuel A Vigne. 2023. Cryptocurrency uncertainty and volatility forecasting of precious metal futures markets. *Journal of Commodity Markets*, 29:100305.
*   Wu et al. (2022) Haixu Wu, Tengge Hu, Yong Liu, Hang Zhou, Jianmin Wang, and Mingsheng Long. 2022. Timesnet: Temporal 2d-variation modeling for general time series analysis. In *The eleventh international conference on learning representations*.
*   Wu et al. (2021) Haixu Wu, Jiehui Xu, Jianmin Wang, and Mingsheng Long. 2021. Autoformer: Decomposition transformers with auto-correlation for long-term series forecasting. *Advances in neural information processing systems*, 34:22419–22430.
*   Wu et al. (2023) Shijie Wu, Ozan Irsoy, Steven Lu, Vadim Dabravolski, Mark Dredze, Sebastian Gehrmann, Prabhanjan Kambadur, David Rosenberg, and Gideon Mann. 2023. Bloomberggpt: A large language model for finance. *arXiv preprint arXiv:2303.17564*.
*   Xie et al. (2023) Qianqian Xie, Weiguang Han, Xiao Zhang, Yanzhao Lai, Min Peng, Alejandro Lopez-Lira, and Jimin Huang. 2023. Pixiu: A large language model, instruction data and evaluation benchmark for finance. *arXiv preprint arXiv:2306.05443*.
*   Yang et al. (2024) Jingfeng Yang, Hongye Jin, Ruixiang Tang, Xiaotian Han, Qizhang Feng, Haoming Jiang, Shaochen Zhong, Bing Yin, and Xia Hu. 2024. Harnessing the power of llms in practice: A survey on chatgpt and beyond. *ACM Transactions on Knowledge Discovery from Data*, 18(6):1–32.
*   Ye et al. (2024) Seonghyeon Ye, Hyeonbin Hwang, Sohee Yang, Hyeongu Yun, Yireun Kim, and Minjoon Seo. 2024. Investigating the effectiveness of task-agnostic prefix prompt for instruction following. In *Proceedings of the AAAI Conference on Artificial Intelligence*, volume 38, pages 19386–19394.
*   Yi et al. (2024) Kun Yi, Qi Zhang, Wei Fan, Shoujin Wang, Pengyang Wang, Hui He, Ning An, Defu Lian, Longbing Cao, and Zhendong Niu. 2024. Frequency-domain mlps are more effective learners in time series forecasting. *Advances in Neural Information Processing Systems*, 36.
*   Yu et al. (2023) Xinli Yu, Zheng Chen, Yuan Ling, Shujing Dong, Zongyi Liu, and Yanbin Lu. 2023. Temporal data meets llm–explainable financial time series forecasting. *arXiv preprint arXiv:2306.11025*.
*   Zhang et al. (2023) Zhuosheng Zhang, Aston Zhang, Mu Li, Hai Zhao, George Karypis, and Alex Smola. 2023. Multimodal chain-of-thought reasoning in language models. *arXiv preprint arXiv:2302.00923*.
*   Zhao et al. (2023) Wayne Xin Zhao, Kun Zhou, Junyi Li, Tianyi Tang, Xiaolei Wang, Yupeng Hou, Yingqian Min, Beichen Zhang, Junjie Zhang, Zican Dong, et al. 2023. A survey of large language models. *arXiv preprint arXiv:2303.18223*.
*   Zhou et al. (2021) Haoyi Zhou, Shanghang Zhang, Jieqi Peng, Shuai Zhang, Jianxin Li, Hui Xiong, and Wancai Zhang. 2021. Informer: Beyond efficient transformer for long sequence time-series forecasting. In *Proceedings of the AAAI conference on artificial intelligence*, volume 35, pages 11106–11115.

## Appendix

## Appendix A License

The CryptoTrade’s dataset is released under the Creative Commons Attribution-NonCommercial-ShareAlike (CC BY-NC-SA) license. This means that anyone can use, distribute, and modify the data for non-commercial purposes as long as they give proper attribution and share the derivative works under the same license terms.

## Appendix B Data Ethics

### B.1 On-chain Data

We collect on-chain data from CoinMarketCap⁹⁹9[https://coinmarketcap.com](https://coinmarketcap.com) and Dune^(10)^(10)10[https://dune.com/home](https://dune.com/home). According to CoinMarketCap’s Terms of Service^(11)^(11)11[https://coinmarketcap.com/terms/](https://coinmarketcap.com/terms/), we are granted a limited, personal, non-exclusive, non-sub-licensable, and non-transferable license to use the content and service solely for personal use. We agree not to use the service or any of the content for any commercial purpose, and we adhere to these requirements. Regarding Dune’s Terms of Service^(12)^(12)12[https://dune.com/terms](https://dune.com/terms), we are permitted to access Dune’s application programming interfaces (the “API”) to perform SQL queries on blockchain data.

### B.2 Off-chain News

We employ the Gnews^(13)^(13)13[https://pypi.org/project/gnews/](https://pypi.org/project/gnews/) to systematically gather news articles related to each cryptocurrency. According to Gnews’ Terms of Service^(14)^(14)14[https://gnews.io/terms/](https://gnews.io/terms/), we can download the news for non-commercial transitory viewing only, and we cannot modify or copy the materials, use the materials for any commercial purpose or any public display, attempt to reverse engineer any software contained on Gnews API’s website, remove any copyright or other proprietary notations from the materials, or transfer the materials to another person or "mirror" the materials on any other server. We adhere to these conditions in our CryptoTrade dataset.

## Appendix C Experimental Environment

All models in our experiments were implemented using Pytorch 2.0.0 in Python 3.9.16, and run on a robust Linux workstation. This system is equipped with two Intel(R) Xeon(R) Gold 6226R CPUs, each operating at a base frequency of 2.90 GHz and a max turbo frequency of 3.90 GHz. With 16 cores each, capable of supporting 32 threads, these CPUs offer a total of 64 logical CPUs for efficient multitasking and parallel computing. The workstation is further complemented by a potent GPU setup, comprising eight NVIDIA GeForce RTX 3090 GPUs, each providing 24.576 GB of memory. The operation of these GPUs is managed by the NVIDIA-SMI 525.60.13 driver and CUDA 12.0, ensuring optimal computational performance for our tasks.

## Appendix D Analysts Examples

In this section, we provide some examples of News Analyst, Market Analyst, Reflection Analyst, and Trading Analyst.

![Refer to caption](img/66433e12f521e00fc53caf1ef214371e.png)

Figure 4: A sample of the Market Analyst.

![Refer to caption](img/61df430e7a9f0e2177e3d29a0177bcd2.png)

Figure 5: A sample of the News Analyst.

![Refer to caption](img/8cc1ecb8ee593529524f6c701566a7dd.png)

Figure 6: A sample of the Trading Analyst.

![Refer to caption](img/e349d55c774d24190e83014755c36f10.png)

Figure 7: A sample of the Reflection Analyst.

## Appendix E Baselines

1.  1.

    Buy and Hold: A straightforward strategy where an asset is purchased at the beginning of the period and held until its end.

2.  2.

    SMA Gencay ([1996](https://arxiv.org/html/2407.09546v1#bib.bib10)): SMA triggers buy or sell decisions based on the asset’s price relative to its moving average. We finetune the SMA period by testing different window sizes $[5,10,15,20,30]$. The optimal period is selected based on the best performance on the validation set.

3.  3.

    SLMA Wang and Kim ([2018](https://arxiv.org/html/2407.09546v1#bib.bib29)): SLMA involves two moving averages of different lengths, with trading signals generated at their crossover points. We use different combinations of short and long SMA periods, selecting the optimal ones based on validation set performance.

4.  4.

    MACD Wang and Kim ([2018](https://arxiv.org/html/2407.09546v1#bib.bib29)): A strategy that uses the MACD indicator to identify potential buy and sell opportunities based on the momentum of the asset. The MACD is calculated as the difference between the 12-day EMA and the 26-day EMA, with a 9-day EMA of the MACD line serving as the signal line. EMA stands for Exponential Moving Average. It is a type of moving average that places a greater weight and significance on the most recent data points.

5.  5.

    Bollinger Bands Day et al. ([2023](https://arxiv.org/html/2407.09546v1#bib.bib6)): This strategy generates trading signals based on price movements relative to the middle, lower, and upper Bollinger Bands. Bollinger Bands are constructed using a 20-day SMA and a multiplier (commonly set to 2) for the standard deviation. We use the recommended period and multiplier settings for this strategy.

6.  6.

    LSTM Ferdiansyah et al. ([2019](https://arxiv.org/html/2407.09546v1#bib.bib9))): This strategy involves comparing today’s price with the predicted price for tomorrow to identify potential buying and selling opportunities. We fine-tune the look-back window size using values in $[1,3,5,10,20,30]$ and select the parameters that perform best on the validation set.

7.  7.

    Informer Zhou et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib42)): Informer utilizes an efficient self-attention mechanism to capture dependencies among variables. We adopt the recommended configuration for our experimental settings: a dropout rate of 0.05, two encoder layers, one decoder layer, a learning rate of 0.0001, and the Adam optimizer Yi et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib38)). The look-back window size is selected using the same procedure as for the LSTM.

8.  8.

    AutoFormer Wu et al. ([2021](https://arxiv.org/html/2407.09546v1#bib.bib33)): AutoFormer introduces a decomposition architecture by embedding the series decomposition block as an inner operator, allowing for the progressive aggregation of the long-term trend from intermediate predictions. We use the recommended configuration for our experimental settings Yi et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib38)). The look-back window size is selected using the same procedure as for the LSTM.

9.  9.

    TimesNet Wu et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib32)): TimesNet provides a general framework for various time-series forecasting tasks. We adopt the recommended configurations for our experimental settings Wu et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib32)). The look-back window size is selected using the same procedure as for the LSTM.

10.  10.

    PatchTST Nie et al. ([2022](https://arxiv.org/html/2407.09546v1#bib.bib20)): PatchTST proposes an effective design for Transformer-based models in time series forecasting by introducing two key components: patching and a channel-independent structure Yi et al. ([2024](https://arxiv.org/html/2407.09546v1#bib.bib38)). The recommended configurations are used for our experimental settings. The look-back window size is selected using the same procedure as for the LSTM.

## Appendix F Author Statement

As authors of the CryptoTrade, we hereby declare that we assume full responsibility for any liability or infringement of third-party rights that may come up from the use of our data. We confirm that we have obtained all necessary permissions and/or licenses needed to share this data with others for their own use. In doing so, we agree to indemnify and hold harmless any person or entity that may suffer damages resulting from our actions.

Furthermore, we confirm that our CryptoTrade dataset is released under the Creative Commons Attribution-NonCommercial-ShareAlike (CC BY-NC-SA) license. This license allows anyone to use, distribute, and modify our data for non-commercial purposes as long as they give proper attribution and share the derivative works under the same license terms. We believe that this licensing model aligns with our goal of promoting open access to high-quality data while respecting the intellectual property rights of all parties involved.

## Appendix G Hosting Plan

After careful consideration, we have chosen to host our code and data on GitHub. Our decision is based on various factors, including the platform’s ease of use, cost-effectiveness, and scalability. We understand that accessibility is key when it comes to data management, which is why we will ensure that our data is easily accessible through a curated interface. We also recognize the importance of maintaining the platform’s stability and functionality, and as such, we will provide the necessary maintenance to ensure that it remains up-to-date, bug-free, and running smoothly.

At the heart of our project is the belief in open access to data, and we are committed to making our data available to those who need it. As part of this commitment, we will be updating our GitHub repository regularly, so that users can rely on timely access to the most current information. We hope that by using GitHub as our hosting platform, we can provide a user-friendly and reliable solution for sharing our data with others.