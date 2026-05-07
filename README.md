# Análise de Dados — Copa do Mundo FIFA 2022

## Visão Geral

Este projeto investiga, com base nos dados oficiais da Copa do Mundo FIFA 2022, quais fatores estatísticos efetivamente diferenciam times vencedores de não vencedores. A análise é estruturada em torno de três dimensões — posse de bola, eficiência ofensiva e construção de jogadas — e parte de uma pergunta clara: a posse de bola, métrica popularmente associada ao desempenho, é de fato um preditor de vitória?

A abordagem combina três níveis analíticos: estatística descritiva, testes de significância não paramétricos (Mann-Whitney U) e modelagem inferencial via regressão logística com features padronizadas. Essa estratificação permite distinguir associações observadas no agregado das relações causais condicionais e revelar efeitos que análises univariadas mascaram.

---

## Objetivo

Avaliar quantitativamente o impacto de três grupos de métricas sobre o resultado de uma partida:

1. **Posse de bola** — domínio territorial via tempo com a bola;
2. **Eficiência ofensiva** — capacidade de converter finalizações no alvo em gols;
3. **Construção de jogadas (build-up)** — progressão ofensiva e quebra das linhas defensivas adversárias.

A meta secundária é entregar um pipeline reprodutível, com pré-processamento documentado, validação estatística formal e modelagem interpretável.

---

## Tecnologias Utilizadas

- **Python 3.10+**
- **pandas** — manipulação e reestruturação dos dados
- **NumPy** — operações vetorizadas e tratamento de divisão por zero
- **Matplotlib** — visualizações analíticas
- **SciPy** — testes estatísticos não paramétricos (Mann-Whitney U)
- **statsmodels** — regressão logística com inferência estatística (p-valores, intervalos de confiança, odds ratios)
- **Jupyter Notebook** — ambiente de desenvolvimento e relatório executável

---

## Instruções de Uso

### 1. Clonar o repositório

```bash
git clone <url-do-repositorio>
cd PROJETO_COPA_DO_MUNDO
```

### 2. Criar ambiente virtual (recomendado)

```bash
python -m venv .venv
source .venv/bin/activate   # Linux / macOS
.venv\Scripts\activate      # Windows
```

### 3. Instalar dependências

```bash
pip install pandas numpy matplotlib scipy statsmodels jupyter
```

### 4. Executar o notebook

```bash
jupyter notebook notebooks/copa_do_mundo_2022.ipynb
```

O notebook normaliza automaticamente o diretório de trabalho para a raiz do projeto, então pode ser executado tanto a partir da raiz quanto da pasta `notebooks/`.

---

## Estrutura de Arquivos

```
PROJETO_COPA_DO_MUNDO/
├── data/
│   ├── raw/
│   │   └── Fifa_world_cup_matches.csv          # Dataset original (uma linha por partida)
│   └── processed/
│       └── world_cup_2022_cleaned.csv          # Dataset tratado (uma linha por time-jogo)
│
├── imagens_graficos/
│   ├── grafico_posse.png                       # Boxplot — posse por resultado
│   ├── grafico_eficiencia.png                  # Barras — eficiência por resultado
│   ├── grafico_construcao.png                  # Barras — build-up por resultado
│   └── grafico_relacao.png                     # Dispersão — eficiência vs gols
│
├── notebooks/
│   └── copa_do_mundo_2022.ipynb                # Pipeline completo, com docstrings
│
└── README.md
```

---

## Processamento dos Dados

O pipeline está encapsulado em funções com responsabilidade única, o que facilita teste, reuso e leitura. As principais etapas:

### 1. Filtragem de colunas relevantes

Apenas o subconjunto das variáveis ligadas às três dimensões de análise é mantido. Os nomes originais das colunas no CSV apresentam inconsistências de formatação (espaços duplos, espaço ausente entre palavras) que foram preservadas no código para garantir compatibilidade com o arquivo de origem e documentadas como peculiaridade conhecida.

### 2. Reestruturação para nível de time

O dataset original armazena cada partida em uma única linha, com colunas espelhadas para `team1` e `team2`. A transformação aplicada gera duas observações por partida — uma para cada time — com nomes padronizados, viabilizando análises por grupo.

### 3. Limpeza de tipos

A coluna `possession` está originalmente em string com sufixo percentual (ex.: `"58%"`). A conversão para float remove o caractere `%` e aplica coerção segura via `errors='coerce'`.

### 4. Engenharia de atributos

| Variável | Definição | Observação |
|---|---|---|
| `efficiency` | `goals / on_target_attempts` | Tratamento explícito de divisão por zero via `np.where` |
| `build_up_score` | `final_third_receptions + attempted_line_breaks + completed_line_breaks` | Métrica composta de progressão ofensiva |
| `win` | `1` se o time tem o maior número de gols na partida, `0` caso contrário | Empates resultam em `win=0` para ambos |
| `match_result` | Categórica `'W'` / `'D'` / `'L'` | Preserva a informação de empate, complementar à `win` |

### 5. Identificação de partidas

A coluna `match_id` é criada a partir do índice original do dataset, garantindo identificador único por partida e agrupamento confiável após o reshape.

---

## Principais Resultados

A análise é apresentada em três níveis crescentes de rigor.

### Nível 1 — Análise descritiva

Médias condicionadas à variável `win`:

| Métrica | Não venceu (média) | Venceu (média) |
|---|---|---|
| Posse de bola (%) | 45,4 | 43,3 |
| Eficiência (gols/chutes no alvo) | 0,19 | 0,43 |
| Build-up score | 281,8 | 297,3 |

A diferença visualmente expressiva está em **eficiência**. Posse e build-up apresentam diferenças pequenas entre os grupos.

### Nível 2 — Validação estatística (Mann-Whitney U)

Adoção do teste não paramétrico justifica-se por dois motivos: distribuições não gaussianas das métricas e robustez a outliers em amostra pequena.

| Métrica | p-valor | Tamanho de efeito | Significante (α=5%) |
|---|---|---|---|
| Posse | 0,252 | 0,12 (desprezível) | Não |
| Eficiência | < 0,001 | 0,45 (moderado) | **Sim** |
| Build-up | 0,269 | 0,12 (desprezível) | Não |

Apenas eficiência sobrevive ao teste de significância no recorte univariado.

### Nível 3 — Modelagem inferencial (regressão logística)

Com features padronizadas, os coeficientes são diretamente comparáveis em magnitude, e a exponencial fornece o **odds ratio** — razão multiplicativa de chance de vitória por desvio-padrão de aumento na variável.

| Variável | Coeficiente | p-valor | Odds ratio |
|---|---|---|---|
| Posse | **−0,76** | 0,018 | 0,47 |
| Eficiência | +0,88 | < 0,001 | 2,42 |
| Build-up | +0,94 | 0,007 | 2,56 |

Pseudo R² de McFadden ≈ 0,16.

A análise multivariada revela dois achados que a descritiva mascara:

1. **Posse de bola tem efeito negativo significante** quando controlada pelas demais variáveis. Mais posse está associada a *menor* chance de vencer — coerente com a hipótese de que posse, em torneios eliminatórios, é frequentemente um indicador de equipes em desvantagem que precisam atacar.
2. **Build-up tem o coeficiente de maior magnitude no modelo**, ainda que não seja significante isoladamente no Mann-Whitney. Isso ilustra como variáveis colineares podem mutuamente mascarar seus efeitos individuais em análises univariadas.

---

## Conclusão

A combinação dos três níveis de análise inverte a narrativa popular: dominar a posse de bola não apenas não garante vitória — em contexto multivariado, está associada a desvantagem. O que diferencia vencedores na Copa do Mundo 2022 é a combinação entre **eficiência na finalização** e **eficácia na construção**. Posse, isoladamente, é um indicador pobre; controlada pelas outras dimensões, é um indicador de fragilidade.

A discrepância entre os achados univariados e multivariados sobre `build_up_score` é, em si, uma lição metodológica: testes de média entre grupos não capturam relações condicionais, e análises descritivas precisam ser complementadas por modelos para evitar conclusões frágeis.

---

## Destaques Técnicos

### Análise estratificada em três níveis (descritiva → testes → modelo)
O projeto não para na média entre grupos. Aplica Mann-Whitney U para validação não paramétrica e regressão logística com features padronizadas para inferência multivariada. Cada nível responde a uma pergunta distinta: "há diferença?" → "a diferença é compatível com o acaso?" → "qual o efeito condicional de cada variável?".

### Escolha justificada de teste não paramétrico
Mann-Whitney U foi escolhido sobre o t-test porque (a) as métricas não são gaussianas, (b) o tamanho amostral é pequeno (n=128 time-jogos) e (c) o teste é robusto a outliers, frequentes em métricas de futebol como eficiência. Tamanho de efeito reportado via rank-biserial correlation, que oferece interpretação independente do tamanho amostral.

### Regressão logística com coeficientes padronizados
As features são z-score-normalizadas antes do ajuste, permitindo comparação direta dos coeficientes. Os odds ratios são reportados com intervalo de confiança 95%, abordagem que privilegia interpretabilidade sobre métricas de classificação como acurácia, mais adequada quando o objetivo é compreensão e não predição.

### Encapsulamento em funções com responsabilidade única
Cada etapa do pipeline é uma função independente, com docstring no padrão Google em português. Funções como `extrair_dados_time`, `calcular_eficiencia`, `criar_variavel_resultado`, `teste_mann_whitney` são reutilizáveis e testáveis isoladamente.

### Tratamento explícito de divisão por zero
O cálculo de `efficiency` usa `np.where(on_target_attempts > 0, goals / on_target_attempts, 0)` em vez de divisão direta. Evita `NaN`/`inf` e mantém o DataFrame consistente para etapas subsequentes.

### Dupla codificação do resultado (W/D/L e binária)
Além da variável binária `win`, o pipeline gera `match_result` com três classes preservando a informação de empate. A premissa de equiparar empate a derrota na binária é declarada e auditável; a categórica está disponível para análises onde essa simplificação não é aceitável.

### Definição operacional de vitória via groupby + transform
A construção de `win` usa `groupby('match_id')['goals'].transform('max')`, vetorizada e idiomática em pandas, robusta a empates e sem necessidade de `apply` ou loops.

### Robustez do caminho de execução
O notebook normaliza automaticamente o diretório de trabalho (`os.chdir('..')` se executado a partir de `notebooks/`), eliminando erros de caminho relativo que tipicamente quebram notebooks compartilhados.

### Visualizações com função analítica clara
Cada gráfico tem título que comunica a leitura esperada (ex.: *"Posse de bola não garante vitória"*). Paleta fixa e padronizada: vermelho para não vencedores, verde para vencedores. A matriz de correlação usa Spearman (não Pearson) por consistência com a escolha do Mann-Whitney — ambos não exigem normalidade.

### Separação raw/processed
Estrutura `data/raw/` e `data/processed/` segue prática consolidada em ciência de dados, isolando a fonte imutável do artefato derivado e garantindo auditabilidade do pipeline.

---

## Limitações

Os resultados devem ser interpretados considerando que: (1) a amostra é pequena (64 partidas, 128 observações time-jogo) e específica a um torneio eliminatório de seleções, com pouca generalização para futebol de clubes ou ligas regulares; (2) métricas agregadas não capturam contexto situacional (posse defensiva vs dominante, chutes em pressão vs contra-ataque); (3) o modelo identifica associações condicionais, não causalidade, fatores não observados como qualidade do plantel podem explicar parte das relações encontradas.

---

## Possíveis Extensões

- Modelagem multinomial considerando vitória, empate e derrota (`match_result`) como variável-alvo de três classes.
- Inclusão de métricas avançadas como expected goals (xG) e expected threat (xT).
- Análise segmentada por fase da competição (grupos vs eliminatórias) ou por perfil tático.
- Validação cruzada e métricas preditivas (AUC, Brier score) caso o objetivo migre de inferência para predição.
- Replicação em ligas longas (Premier League, Champions League) para testar generalização dos achados.
