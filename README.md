# 🌍 Previsão de Emissões de CO₂ per capita com Indicadores Globais

## 📌 Visão Geral

Este repositório implementa um **pipeline completo de previsão de emissões de CO₂ per capita** (`EN.ATM.CO2E.PC`) utilizando um painel anual por país construído a partir de indicadores socioeconômicos e energéticos globais.

O projeto investiga como dados históricos podem ser utilizados para **prever emissões futuras**, simulando um cenário real de forecasting e evitando vazamento temporal (*temporal leakage*).

Trata-se de um projeto com finalidade **didática e de portfólio**, com foco em boas práticas de ciência de dados, validação temporal correta e comparação metodológica entre diferentes estratégias de modelagem.

---

## 🎯 Objetivo

Dado um conjunto de indicadores anuais por país, o objetivo é:

> **Prever o CO₂ per capita do próximo ano** utilizando apenas informações disponíveis até o presente.

A avaliação segue um esquema **walk-forward**, reproduzindo o uso real do modelo:

| Rodada | Treino | Teste |
|------|--------|-------|
| 1 | 2000–2018 | 2019 |
| 2 | 2000–2019 | 2020 |

---

## ⚙️ Fases do Projeto

### **Fase 1 — ETL e Preparação dos Dados (`df_long → df_wide`)**

Nesta etapa é construído o painel analítico a partir dos indicadores brutos.

**Processos principais:**

- Extração e consolidação dos indicadores em formato longo (`df_long`)
  - 1 linha por **país–indicador–ano**
- Limpeza de entidades não analíticas:
  - remoção de `XY / Not classified`
  - remoção de agregados e regiões
  - manutenção apenas de países/territórios
- Pré-análise de valores ausentes:
  - missing global
  - missing por país, indicador e ano

**Transformação estrutural:**

- Pivot para formato *wide* (`df_wide`)
  - 1 linha por `(Country, Indicator)`
  - colunas representando anos
  - valores correspondentes ao indicador

**Testes de sanidade:**

- verificação de anos disponíveis  
- checagem de duplicatas na chave  
- análise de missing por ano  

---

### **Fase 2 — Treinamento e Teste (Walk-Forward 2019–2020)**

O `df_wide` é convertido para um painel **país–ano**, contendo:

- 1 linha por `(país, ano)`
- colunas representando indicadores
- variável alvo: CO₂ per capita no ano seguinte

Para isolar o efeito da modelagem, é utilizada uma **única estratégia conservadora de imputação**, aplicada separadamente em cada rodada:

- interpolação temporal por país  
- propagação limitada  
- fallback pela mediana do conjunto de treino  
- sem acesso a dados futuros  

---

## 🤖 Abordagens de Modelagem

### ✅ Abordagem 1 — Machine Learning Tabular com Engenharia Temporal

O problema é tratado como regressão tabular utilizando atributos temporais por país:

- lags (`t-1`, `t-2`) do CO₂ e indicadores
- variações entre períodos (*delta*)
- relações não lineares entre variáveis

Modelo típico:
- Gradient Boosting ou equivalente.

**Motivação:**  
capturar padrões temporais e interações complexas sem modelagem explícita de séries temporais.

---

### ✅ Abordagem 2 — Painel Explícito com Efeito Fixo de País

Treinamento de um modelo global compartilhado entre países contendo:

- indicadores no ano `t`
- efeito estrutural de país (codificação do país)

Modelo típico:
- regressão regularizada (ex.: Ridge).

**Motivação:**  
equilibrar interpretabilidade, robustez e generalização diante da heterogeneidade entre países.

---

### ✅ Abordagem 3 — Modelagem em Duas Etapas (Drivers → CO₂)

Pipeline estruturado em dois passos:

1. Previsão dos **drivers macroeconômicos/energéticos** no ano de teste  
   (baseline conservador, ex.: persistência `t = t-1`)

2. Previsão do CO₂ utilizando os drivers previstos como entrada.

**Motivação:**  
impor coerência estrutural, fazendo com que emissões acompanhem variáveis econômicas e energéticas relevantes.

---

## 📊 Fase 3 — Avaliação e Comparação

As abordagens são comparadas nos anos de teste (2019 e 2020) utilizando:

- **MAE — Mean Absolute Error**
- **RMSE — Root Mean Squared Error**

A análise considera:

- desempenho por rodada  
- estabilidade entre anos  
- sensibilidade a missing  
- robustez em países com séries curtas  
- plausibilidade das previsões  

O resultado final desta etapa é a identificação da abordagem mais consistente para evolução futura do projeto.

---

## 🧠 Principais Aprendizados

Este projeto enfatiza:

- validação temporal realista
- modelagem de dados em painel
- engenharia temporal de atributos
- comparação entre ML e abordagens estruturadas
- construção de pipelines reproduzíveis

---

## 📁 Organização do Repositório

```text
├── data/
├── notebooks/
├── src/
│   ├── etl/
│   ├── features/
│   ├── models/
│   └── evaluation/
├── results/
└── README.md
```

---

## ⚠️ Aviso

Este projeto possui caráter **educacional e analítico**.  
Os resultados não representam previsões oficiais nem recomendações de políticas públicas.

---

## 🔮 Possíveis Extensões

- horizontes multi-ano
- previsões probabilísticas
- interpretabilidade (ex.: SHAP)
- simulações de cenários econômicos e energéticos

---

## 👤 Autor

Projeto desenvolvido como parte de um portfólio pessoal em Ciência de Dados, com foco em previsão temporal e modelagem de dados em painel.


