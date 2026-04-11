# Documentação das Features

## Visão Geral
O modelo utiliza **52 features** divididas em 4 categorias e 3 timeframes (1h, 15m, 4h).

---

## 1. Price Features

Features derivadas diretamente dos preços de fechamento (close).

### 1.1 Log Return (Retorno Logarítmico)
**Features:** `log_return_1h`, `log_return_15m`, `log_return_4h`

#### Fórmula
$$r_t = \ln(P_t) - \ln(P_{t-1}) = \ln\left(\frac{P_t}{P_{t-1}}\right)$$

#### Explicação
* Calcula o retorno percentual entre dois períodos usando logaritmos naturais
* Propriedade aditiva: retornos de diferentes períodos podem ser somados
* Aproximadamente igual ao retorno simples para valores pequenos
* Mais adequado para séries temporais financeiras (normalidade, estacionariedade)

#### Interpretação
* $r_t > 0$: Preço subiu em relação ao período anterior
* $r_t < 0$: Preço caiu em relação ao período anterior
* $|r_t|$ grande: Movimento forte (alta volatilidade)

---

### 1.2 Volatilidade
**Features:** `volatility_1h`, `volatility_15m`, `volatility_4h`

#### Fórmula
$$\sigma_t = \sqrt{\frac{1}{n-1} \sum_{i=0}^{n-1} (r_{t-i} - \bar{r})^2}$$

Onde:
* $r_{t-i}$ = log return no período $t-i$
* $\bar{r}$ = média dos retornos na janela
* $n$ = tamanho da janela (tipicamente 20 períodos)

#### Explicação
* Desvio padrão dos retornos logarítmicos em uma janela móvel
* Mede a dispersão dos retornos (risco/incerteza)
* Valores altos indicam movimentos grandes e imprevisíveis

#### Interpretação
* $\sigma$ alto: Alta volatilidade, mercado agitado, maior risco
* $\sigma$ baixo: Baixa volatilidade, mercado calmo, menor risco
* Usado para dimensionar stop loss e take profit

---

### 1.3 Z-Score do Preço
**Features:** `z_score_close_1h`, `z_score_close_15m`, `z_score_close_4h`

#### Fórmula
$$z_t = \frac{P_t - \mu}{\sigma}$$

Onde:
* $P_t$ = preço atual
* $\mu$ = média móvel do preço (janela de 20 períodos)
* $\sigma$ = desvio padrão do preço (janela de 20 períodos)

#### Explicação
* Normaliza o preço em relação à sua distribuição histórica
* Mede quantos desvios padrão o preço está da média
* Remove tendência e escala, tornando ativos comparáveis

#### Interpretação
* $z > 2$: Preço muito acima da média (possível sobrecompra)
* $z < -2$: Preço muito abaixo da média (possível sobrevenda)
* $|z| > 3$: Evento extremo (reversão provável)

---

### 1.4 Média Móvel
**Features:** `ma_close_1h`, `ma_close_15m`, `ma_close_4h`

#### Fórmula (SMA - Simple Moving Average)
$$MA_t = \frac{1}{n} \sum_{i=0}^{n-1} P_{t-i}$$

Onde:
* $n$ = período da média (tipicamente 20)

#### Explicação
* Média aritmética dos últimos $n$ preços
* Suaviza flutuações de curto prazo
* Identifica direção da tendência

#### Interpretação
* $P_t > MA_t$: Preço acima da média (tendência de alta)
* $P_t < MA_t$: Preço abaixo da média (tendência de baixa)
* Cruzamentos: Sinais de compra/venda

---

### 1.5 Bollinger Bands
**Features:** `bb_upper_1h`, `bb_lower_1h`, `bb_position_1h` (e versões 15m, 4h)

#### Fórmulas
$$BB_{upper} = MA_t + k \cdot \sigma_t$$
$$BB_{lower} = MA_t - k \cdot \sigma_t$$
$$BB_{position} = \frac{P_t - BB_{lower}}{BB_{upper} - BB_{lower}}$$

Onde:
* $MA_t$ = média móvel (20 períodos)
* $\sigma_t$ = desvio padrão (20 períodos)
* $k$ = multiplicador (tipicamente 2)

#### Explicação
* Bandas que se expandem/contraem com a volatilidade
* Delimitam região "normal" de variação do preço
* `bb_position` normaliza a posição do preço entre as bandas

#### Interpretação
* $BB_{position} > 0.8$: Próximo à banda superior (sobrecompra)
* $BB_{position} < 0.2$: Próximo à banda inferior (sobrevenda)
* Bandas estreitas: Baixa volatilidade (possível breakout)
* Bandas largas: Alta volatilidade (movimento forte)

---

## 2. Volume Features

Features derivadas do volume de negociação.

### 2.1 Z-Score do Volume
**Features:** `z_score_volume_1h`, `z_score_volume_15m`, `z_score_volume_4h`

#### Fórmula
$$z_{vol,t} = \frac{V_t - \mu_{vol}}{\sigma_{vol}}$$

Onde:
* $V_t$ = volume atual
* $\mu_{vol}$ = média do volume (20 períodos)
* $\sigma_{vol}$ = desvio padrão do volume (20 períodos)

#### Explicação
* Normaliza o volume em relação ao histórico
* Identifica picos anormais de atividade

#### Interpretação
* $z_{vol} > 2$: Volume muito acima da média (forte interesse)
* $z_{vol} < -1$: Volume muito abaixo da média (pouco interesse)
* Volume alto + movimento de preço: Confirmação de tendência

---

### 2.2 Média Móvel do Volume
**Features:** `ma_volume_1h`, `ma_volume_15m`, `ma_volume_4h`

#### Fórmula
$$MA_{vol,t} = \frac{1}{n} \sum_{i=0}^{n-1} V_{t-i}$$

#### Explicação
* Média móvel simples do volume
* Baseline para comparar volume atual

#### Interpretação
* $V_t > MA_{vol,t}$: Volume acima da média (interesse crescente)
* Breakouts com volume alto: Mais confiáveis

---

### 2.3 Volume Change (Mudança Percentual)
**Features:** `vol_change_1h`, `vol_change_15m`, `vol_change_4h`

#### Fórmula
$$\Delta V_t = \frac{V_t - V_{t-1}}{V_{t-1}}$$

#### Explicação
* Variação percentual do volume em relação ao período anterior
* Detecta acelerações/desacelerações na atividade

#### Interpretação
* $\Delta V_t > 1$: Volume dobrou (forte aumento de interesse)
* $\Delta V_t < -0.5$: Volume caiu pela metade (diminuição de interesse)

---

## 3. Technical Indicators

Indicadores técnicos clássicos.

### 3.1 RSI (Relative Strength Index)
**Features:** `rsi_1h`, `rsi_15m`, `rsi_4h`

#### Fórmula
$$RSI = 100 - \frac{100}{1 + RS}$$

Onde:
$$RS = \frac{\text{Média de ganhos (14 períodos)}}{\text{Média de perdas (14 períodos)}}$$

$$\text{Ganho}_t = \max(P_t - P_{t-1}, 0)$$
$$\text{Perda}_t = \max(P_{t-1} - P_t, 0)$$

#### Explicação
* Oscilador de momentum (0-100)
* Compara magnitude de ganhos recentes vs perdas recentes
* Identifica condições de sobrecompra/sobrevenda

#### Interpretação
* $RSI > 70$: Sobrecompra (possível reversão de baixa)
* $RSI < 30$: Sobrevenda (possível reversão de alta)
* $RSI \approx 50$: Equilíbrio entre compra e venda
* Divergências: RSI vs preço indicam possíveis reversões

---

### 3.2 ATR (Average True Range)
**Features:** `atr_1h`, `atr_15m`, `atr_4h`

#### Fórmula
$$TR_t = \max(H_t - L_t, |H_t - C_{t-1}|, |L_t - C_{t-1}|)$$
$$ATR_t = \frac{1}{n} \sum_{i=0}^{n-1} TR_{t-i}$$

Onde:
* $H_t$ = máxima do período
* $L_t$ = mínima do período
* $C_{t-1}$ = fechamento anterior
* $n$ = período (tipicamente 14)

#### Explicação
* Mede a volatilidade real do ativo (range médio)
* Considera gaps entre períodos
* Base para stop loss e take profit adaptativos

#### Interpretação
* $ATR$ alto: Alta volatilidade, maior risco, stops mais largos
* $ATR$ baixo: Baixa volatilidade, menor risco, stops mais apertados
* **Usado no projeto:** TP = 2×ATR, SL = 1×ATR

---

### 3.3 Momentum
**Features:** `momentum_1h`, `momentum_15m`, `momentum_4h`

#### Fórmula
$$M_t = \frac{P_t - P_{t-n}}{P_{t-n}}$$

Onde:
* $n$ = lookback period (tipicamente 3-5 períodos)

#### Explicação
* Velocidade da mudança de preço
* Retorno percentual ao longo de $n$ períodos

#### Interpretação
* $M_t > 0$: Momentum positivo (alta)
* $M_t < 0$: Momentum negativo (baixa)
* $|M_t|$ grande: Movimento forte e rápido

---

## 4. Correlation Features

Features de correlação com ativos benchmark (BTC, ETH).

### 4.1 Beta com BTC
**Feature:** `beta_btc`

#### Fórmula
$$\beta = \frac{\text{Cov}(R_{asset}, R_{BTC})}{\text{Var}(R_{BTC})}$$

Onde:
* $R_{asset}$ = retornos do ativo
* $R_{BTC}$ = retornos do Bitcoin
* Janela de cálculo: 30 períodos

#### Explicação
* Mede sensibilidade do ativo em relação ao Bitcoin
* Beta do modelo CAPM (Capital Asset Pricing Model)
* Indica quanto o ativo move quando o BTC move 1%

#### Interpretação
* $\beta = 1$: Move igual ao BTC (correlação perfeita)
* $\beta > 1$: Move mais que o BTC (mais volátil)
* $\beta < 1$: Move menos que o BTC (mais estável)
* $\beta < 0$: Move na direção oposta (correlação negativa)

---

### 4.2 Relative Strength (Força Relativa)
**Features:** `relative_strength_btc`, `relative_strength_eth`

#### Fórmula
$$RS_t = \frac{P_{asset,t} / P_{asset,t-n}}{P_{benchmark,t} / P_{benchmark,t-n}}$$

Onde:
* $n$ = período de lookback (tipicamente 20)

#### Explicação
* Compara performance relativa do ativo vs benchmark
* Ratio dos retornos percentuais
* Identifica ativos que estão superando (outperforming) ou ficando para trás (underperforming)

#### Interpretação
* $RS > 1$: Ativo mais forte que benchmark (outperforming)
* $RS < 1$: Ativo mais fraco que benchmark (underperforming)
* $RS$ crescente: Força relativa aumentando
* $RS$ decrescente: Força relativa diminuindo

---

### 4.3 Volatility Ratio
**Feature:** `volatility_ratio_btc`

#### Fórmula
$$VR_t = \frac{\sigma_{asset,t}}{\sigma_{BTC,t}}$$

#### Explicação
* Compara a volatilidade do ativo com a do Bitcoin
* Identifica ativos mais ou menos voláteis que o benchmark

#### Interpretação
* $VR > 1$: Ativo mais volátil que BTC (maior risco)
* $VR < 1$: Ativo menos volátil que BTC (menor risco)

---

### 4.4 Volume Ratio
**Feature:** `volume_ratio_btc`

#### Fórmula
$$VR_{vol,t} = \frac{V_{asset,t}}{V_{BTC,t}}$$

#### Explicação
* Compara o volume do ativo com o do Bitcoin
* Identifica ativos com maior ou menor liquidez relativa

#### Interpretação
* $VR_{vol}$ alto: Alto volume relativo (boa liquidez)
* $VR_{vol}$ baixo: Baixo volume relativo (possível iliquidez)

---

### 4.5 BTC/ETH Raw Features
**Features:** `btc_close`, `btc_log_return`, `btc_volume`, `btc_volatility`, `eth_close`, `eth_log_return`, `eth_volume`, `eth_volatility`

#### Explicação
* Features brutas do Bitcoin e Ethereum
* Permitem ao modelo capturar dinâmicas diretas dos benchmarks
* Complementam as features de correlação

---

## Feature Importance (Top 10)

Com base no último treinamento (04/04/2026):

1. **log_return_4h** (13.8%) - Retorno logarítmico 4h é o fator mais crítico
2. **momentum_1h** (9.8%) - Momentum de curto prazo (velocidade do movimento)
3. **log_return_1h** (5.4%) - Retorno logarítmico operacional
4. **z_score_close_1h** (4.2%) - Normalização do preço 1h (desvio da média)
5. **momentum_4h** (3.9%) - Momentum estratégico (tendência de médio prazo)
6. **bb_position_4h** (3.6%) - Posição nas Bollinger Bands 4h
7. **volatility_1h** (2.5%) - Volatilidade operacional (dispersão de retornos)
8. **bb_position_1h** (2.5%) - Posição nas Bollinger Bands 1h
9. **bb_position_15m** (2.5%) - Posição nas Bollinger Bands 15m
10. **z_score_close_15m** (2.3%) - Normalização do preço 15m

---

## Insights

### Timeframe Analysis
* **4h é extremamente importante:** Feature mais relevante (log_return_4h)
  * Timeframe estratégico captura tendências de médio prazo
  * Reduz ruído e falsos sinais
* **1h domina em quantidade:** Múltiplas features 1h no top 10
  * Timeframe operacional principal
  * Melhor balanço entre sinal e ruído para horizonte de 24h
* **15m contribui tacticamente:** Features 15m no top 10
  * Captura microestrutura de curto prazo
  * Útil para timing de entrada

### Feature Categories (por importância)
* **Price features dominam:** log_return, z_score, volatility, bb_position
* **Momentum é crítico:** momentum_1h e momentum_4h no top 5
* **Bollinger Bands consistente:** bb_position em todos os timeframes
* **Volume é secundário:** Menor importância relativa
* **Correlação com BTC/ETH:** beta_btc aparece no ranking mas não no top 10

### Key Findings
* **Retornos logarítmicos são cruciais:** log_return_4h é a feature #1
* **Multi-timeframe efetivo:** Modelo captura bem 1h, 15m e 4h
* **Posição relativa importante:** bb_position consistente em todos os timeframes
* **Momentum é preditivo:** Velocidade de mudança é um forte indicador

---

## Próximas Features (Roadmap)

### 1. Sentiment & Market Microstructure
* **Funding rate:** Taxa de financiamento de contratos perpétuos (sentimento bull/bear)
* **Open interest:** Volume total de contratos abertos (interesse no mercado)
* **Long/short ratio:** Proporção de posições compradas vs vendidas

### 2. Order Book Features
* **Bid-ask spread:** Diferença entre melhor oferta de compra e venda (liquidez)
* **Order book imbalance:** Desequilíbrio entre ordens de compra/venda
* **Market depth:** Profundidade do livro de ofertas (resistência a movimentos)

### 3. On-Chain Metrics (para criptomoedas)
* **Transaction volume:** Volume de transações on-chain
* **Active addresses:** Número de endereços ativos (adoção)
* **Exchange inflow/outflow:** Fluxo para/de exchanges (pressão de venda/compra)
* **MVRV ratio:** Market Value to Realized Value (sobrecompra/sobrevenda on-chain)

### 4. Cross-Asset Correlations
* **S&P 500:** Correlação com mercado acionário tradicional (risk-on/risk-off)
* **DXY (US Dollar Index):** Correlação com dólar (risk sentiment)
* **Gold:** Correlação com safe-haven assets
* **VIX:** Correlação com volatilidade implícita de ações (fear index)

### 5. Time-Based Features
* **Day of week:** Segunda-feira vs sexta-feira (padrões semanais)
* **Hour of day:** Horários de maior volume/volatilidade
* **US market hours:** Overlap com horário de NY (maior liquidez)
* **Holiday indicators:** Vésperas de feriados (menor liquidez)

---

## Referências

* **RSI:** Wilder, J. W. (1978). New Concepts in Technical Trading Systems
* **Bollinger Bands:** Bollinger, J. (2001). Bollinger on Bollinger Bands
* **ATR:** Wilder, J. W. (1978). New Concepts in Technical Trading Systems
* **Beta/CAPM:** Sharpe, W. F. (1964). Capital Asset Prices

---

**Última atualização:** 04/04/2026
