# Quant Predictor
### Previsão de movimentos de curto prazo em criptomoedas
Este projeto utiliza PySpark e Scikit-Learn para processar dados históricos de criptomoedas (OHLCV), realizar engenharia de atributos (feature engineering) avançada e treinar modelos de classificação para identificar oportunidades de compra baseadas em volatilidade e exaustão de preço.

#### Funcionalidades
- **Ingestão de Dados:** Conexão com API da Bybit/Binance via CCXT com paginação automática para coletar grandes históricos (>10k candles).
- **Target Dinâmico (ATR):** O alvo de lucro (Take Profit) não é fixo; ele se ajusta automaticamente à volatilidade do mercado usando o Average True Range.
- **Feature Engineering:** - Indicadores Técnicos: RSI, Bandas de Bollinger, VWAP Diário.
- **Estatística de Janela Móvel:** Z-Score, Assimetria (Skewness) e Curtose.
- **Momentum e Sazonalidade:** Log-returns, Momentum de 3 períodos e extração de hora do dia.
- **Pipeline de ML:** - Separação Temporal (Train/Test Split) para evitar Data Leakage.
- Balanceamento de classes via Undersampling
- Ajuste de Threshold de decisão para otimização de Precision/Recall

#### Ferramentas
- Databricks (Ambiente de desenvolvimento)
- PySpark (Processamento de grandes volumes de dados)
- Pandas & Scikit-Learn (Modelagem e avaliação)
- CCXT (Conexão com Exchanges de Cripto)
- Delta Lake (Armazenamento de features)

#### Comentário
- Desempenho ainda não está muito bom
- Falta explorar mais variáveis
