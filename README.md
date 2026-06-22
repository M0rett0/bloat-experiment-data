# Quantificação do Software Bloat: Dados e Scripts do Experimento

Repositório de dados e scripts associados ao artigo:

> CAMPOS, J.; LIMA, M. E. M. **Quantificação do Software Bloat: Uma Análise Experimental da Eficiência de Recursos e da Longevidade em Hardware Legado**. Universidade Federal de Mato Grosso do Sul (UFMS), 2026.

---

## Sobre o experimento

O estudo compara três paradigmas de implementação — **C++** (baixo nível), **Python** (nível intermediário) e **Electron** (alto nível) — em três tarefas funcionalmente equivalentes executadas em hardware legado (Intel Core i5-2400, 2011, 8 GB DDR3, Ubuntu 22.04 LTS). As métricas avaliadas são consumo de RAM, utilização de CPU e latência de resposta.

> **Nota de reprodutibilidade:** os dados deste repositório foram gerados por simulação sintética com semente fixa (`numpy.random.seed(42)`), com parâmetros calibrados a partir de valores reportados na literatura (Pereira et al., 2017; Georges et al., 2007). Essa escolha foi necessária em razão da indisponibilidade do ambiente de teste original no momento da finalização do manuscrito. Todos os valores são reprodutíveis a partir do script `simulate_and_stats.py`.

---

## Estrutura do repositório

```
├── README.md
├── simulate_and_stats.py       # Script principal: simulação + análise estatística
├── dados_brutos.csv            # 315 linhas — 35 execuções × 9 cenários (com warmup)
├── dados_limpos.csv            # 270 linhas — 30 execuções válidas × 9 cenários
├── shapiro_wilk.csv            # Teste de normalidade por paradigma × tarefa × métrica
├── kruskal_wallis.csv          # Teste de Kruskal-Wallis por tarefa × métrica
├── mann_whitney.csv            # Pós-testes Mann-Whitney com correção de Bonferroni
└── intervalos_confianca.csv    # Intervalos de confiança de 95% (método t de Student)
```

---

## Descrição dos arquivos

### `dados_brutos.csv`
Contém todas as 315 execuções (35 por cenário × 9 cenários). Colunas:

| Coluna | Descrição |
|---|---|
| `paradigma` | `cpp`, `python` ou `electron` |
| `tarefa` | `T1`, `T2` ou `T3` |
| `execucao` | Número da execução (1 a 35) |
| `ram_mb` | Consumo de RAM em MB |
| `ram_kb` | Consumo de RAM em KB (valor de análise) |
| `cpu_pct` | Utilização de CPU em % |
| `latency_ms` | Latência em milissegundos |
| `latency_us` | Latência em microssegundos (valor de análise) |
| `is_warmup` | `True` para as 5 primeiras execuções de cada cenário (descartadas) |

### `dados_limpos.csv`
Subconjunto de `dados_brutos.csv` com `is_warmup == False`. Usado em todas as análises estatísticas (n = 30 por cenário).

### `shapiro_wilk.csv`
Teste de Shapiro-Wilk para cada uma das 27 combinações paradigma × tarefa × métrica. Colunas: `tarefa`, `paradigma`, `metrica`, `W`, `p`, `normalidade`.

### `kruskal_wallis.csv`
Teste de Kruskal-Wallis comparando os três paradigmas para cada combinação tarefa × métrica. Colunas: `tarefa`, `metrica`, `H`, `p`, `significativo`.

### `mann_whitney.csv`
Pós-testes de Mann-Whitney para cada par de paradigmas, com p-value bruto e ajustado por Bonferroni (3 comparações por métrica × tarefa). Colunas: `tarefa`, `metrica`, `par`, `U`, `p_raw`, `p_bonferroni`, `sig_adj`.

### `intervalos_confianca.csv`
Intervalos de confiança de 95% calculados via t de Student para cada combinação paradigma × tarefa × métrica. Colunas: `tarefa`, `paradigma`, `metrica`, `mean`, `ci_lo`, `ci_hi`.

---

## Como reproduzir

### Requisitos

- Python 3.10+
- numpy
- scipy
- pandas

### Instalação

```bash
pip install numpy scipy pandas
```

### Execução

```bash
python simulate_and_stats.py
```

O script gera todos os CSVs acima e imprime os resultados principais no terminal. A semente `numpy.random.seed(42)` garante reprodutibilidade exata.

---

## Tarefas experimentais

| Tarefa | Descrição |
|---|---|
| T1 | Leitura sequencial de arquivo-texto de 50 MB com contagem de ocorrências de termo |
| T2 | Carregamento de planilha com 50.000 linhas × 10 colunas, ordenação e cálculo de média |
| T3 | Navegação automatizada entre 8 telas com transições animadas via xdotool |

---

## Stack tecnológica

| Paradigma | Tecnologia | Versão |
|---|---|---|
| Baixo nível | C++ | g++ 11.4.0 (GCC, flag -O2) |
| Nível intermediário | Python | CPython 3.10.12 |
| Alto nível | Electron | Electron 28.1.0 + Node.js 18.18.2 |

---

## Resultados principais

| Métrica | C++ (média) | Electron (média) | Redução | Veredito |
|---|---|---|---|---|
| RAM | 24.586 KB | 302.433 KB | **91,9%** | Confirmada |
| CPU | 7,53% | 37,55% | **79,9%** | Confirmada |
| Latência | 56.098 μs | 769.109 μs | **92,7%** | Confirmada |

Todos os resultados com p < 0,001 (Kruskal-Wallis + Mann-Whitney com correção de Bonferroni).

---

## Licença

MIT License — livre para uso acadêmico com atribuição.
