# Documentação do Projeto: Dashboard ONS (Sistema Elétrico Brasileiro)
**Destinado a:** Agentes de Inteligência Artificial / Desenvolvedores Futuros.

## 📌 Contexto
Este projeto é um dashboard interativo desenvolvido em **Streamlit** para visualização e análise de dados do Sistema Elétrico Brasileiro, com base em informações fornecidas pelo ONS (Operador Nacional do Sistema Elétrico). O escopo temporal cobre o período de **2021 a 2026**.
O objetivo final é apresentar análises profundas sobre Segurança Energética, Transição Energética e Custo da Energia para fins acadêmicos em uma universidade federal.

## 📂 Estrutura de Diretórios
A pasta raiz do projeto está em `c:/Users/enzog/Desktop/grafico enregia/`.
- `Dados/`: Contém arquivos `.csv` anuais (separados por `;`).
  - `BALANCO_ENERGIA_SUBSISTEMA_*.csv` (Geração Hidráulica, Térmica, Eólica, Solar)
  - `CARGA_ENERGIA_*.csv` (Carga e demanda)
  - `CMO_SEMANAL_*.csv` (Custo Marginal de Operação)
  - `EAR_DIARIO_SUBSISTEMA_*.csv` (Energia Armazenada)
  - `ENA_DIARIO_SUBSISTEMA_*.csv` (Energia Natural Afluente - Chuvas)
- `app.py`: Arquivo principal da aplicação Streamlit.

## 🛠️ Tecnologias e Dependências
Para que o dashboard funcione plenamente, as seguintes bibliotecas foram instaladas no ambiente local do usuário (`C:/Users/enzog/AppData/Local/Python/pythoncore-3.14-64/bin/python.exe`):
- `streamlit`: Framework principal.
- `pandas`: Processamento de dados (`concat`, agregação, conversões de `datetime`).
- `plotly`: Criação de gráficos dinâmicos (`plotly.express` e `plotly.graph_objects`).
- `statsmodels`: Necessário estritamente para o cálculo de linhas de tendência OLS (Ordinary Least Squares) nos gráficos de dispersão do Plotly (`trendline="ols"`).

*(Comando base para recriar o ambiente: `python -m pip install streamlit pandas plotly statsmodels`)*

## 🧠 Lógica de Implementação no `app.py`

### 1. Carregamento de Dados (`@st.cache_data`)
Criamos uma função genérica `carregar_dados` utilizando a biblioteca `glob` para iterar de forma dinâmica por todos os arquivos de dados de todos os anos (2021-2026) que correspondam a um padrão (ex: `BALANCO_*.csv`). Isso elimina a necessidade de codificar anos manualmente. O `st.cache_data` é usado intensivamente para manter os dataframes massivos na RAM sem reprocessamento a cada interação.

### 2. Filtros Dinâmicos (Sidebar)
Os dados brutos passam por uma função unificada `aplicar_filtros()` baseada na data (`data_inicio`, `data_fim`) e na seleção múltipla dos `subsistemas`.
- *Ponto de atenção:* Diferentes planilhas possuem diferentes colunas de tempo (`din_instante`, `ear_data`, `ena_data`). A função mapeia e aplica corretamente os filtros para os 5 dataframes principais (`df_balanco`, `df_carga`, `df_cmo`, `df_ear`, `df_ena`).

### 3. Normalização de Subsistemas (⚠️ importante)
Diferentes CSVs usavam grafias distintas para o mesmo subsistema (`"NORTE"` vs `"Norte"`, `"SUDESTE/CENTRO-OESTE"` vs `"SUDESTE"`). Isso fazia o filtro de subsistemas falhar silenciosamente em todas as abas exceto a primeira. O `carregar_dados()` agora aplica o dicionário `MAPA_SUBSISTEMA` para unificar tudo em `NORTE / NORDESTE / SUDESTE / SUL / SIN`.

Adicionalmente, `"SISTEMA INTERLIGADO NACIONAL"` (SIN) no BALANCO é uma linha de **soma agregada** — precisa ser separada dos regionais, caso contrário qualquer agregação duplica os valores. Os DataFrames `df_balanco_reg` (regiões) e `df_balanco_sin` (total nacional) isolam esses escopos.

### 4. Modularização das Abas
O dashboard foi estruturado através do `st.tabs` abordando 8 perspectivas:
1. **Visão Geral**: Resumo executivo com KPIs principais (com delta vs período anterior), matriz de geração em pizza e carga × geração total.
2. **Segurança Energética**: EAR × despacho térmico com eixo duplo, correlação de Pearson exibida, dispersão com trendline OLS.
3. **Transição Energética**: *Stacked Area* + linhas de *Market Share* + cálculo de crescimento renovável (p.p.) entre início e fim do período.
4. **Subsistemas & Intercâmbio**: Carga regional + **gráfico de barras divergente** mostrando importação/exportação líquida via `val_intercambio`.
5. **Sazonalidade**: *Boxplot* mensal + *Heatmap* Ano × Mês + KPI do mês de maior consumo.
6. **Custo (CMO)**: Evolução por subsistema + dispersão **CMO × EAR** com correlação (merge_asof para alinhar série semanal com diária).
7. **Impacto das Chuvas (ENA)**: ENA e EAR sobrepostos com linha de referência da MLT.
8. **Comparação Anual**: Curvas sobrepostas por ano para qualquer indicador (Carga/EAR/ENA/CMO/Térmica) + tabela estatística.

### 5. Sidebar — Presets e Alertas
- **Presets de período**: botões rápidos para "Último ano", "Últimos 6 meses", "Ano atual", "Crise Hídrica 2021".
- **Alertas dinâmicos**: classifica o EAR mais recente por subsistema em 🔴 crítico (<30%), 🟡 baixo (<50%) ou 🟢 OK.

### 6. UI / Estilização (Custom CSS)
Foi injetado CSS via `st.markdown(unsafe_allow_html=True)` para a criação de Cards KPI (fundo `#1E1E1E`, tipografia clara, sombras discretas). Os cards agora suportam `delta` (▲/▼ vs período anterior) e subtextos. As caixas de alerta seguem o mesmo padrão (borda lateral colorida).

## 🚀 Como Executar
1. Navegue até o diretório via terminal.
2. Execute o comando: `streamlit run app.py`
3. A aplicação estará disponível em `http://localhost:8501`.

## 🤝 Instruções para Futuras IAs
Se você, agente de inteligência artificial, for solicitado a adicionar novas features, por favor considere:
- **Respeitar os esquemas de cores do Plotly**: Cores já definidas para energias (Azul para Hídrica, Vermelho para Térmica, Ciano para Eólica e Amarelo para Solar).
- **Evitar quebras de Memória**: Ao fazer novas integrações (`pd.merge`), tome cuidado com cruzamentos cartesianos, visto que a base contém dados *diários* por subsistema ao longo de 6 anos.
- **Inserir Insights Dinâmicos**: A interface faz uso da caixa `st.info()` para descrever aprendizados acadêmicos. Tente seguir esse padrão de narrativa de dados.
