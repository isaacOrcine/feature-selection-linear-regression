# Seleção de variáveis em regressão linear: excesso × critério

Experimento reproduzível em Python que mostra, com números, por que escolher bem as variáveis faz diferença em uma regressão linear.

Dois modelos são treinados com os mesmos dados. Um usa **6 variáveis escolhidas com critério**. O outro usa **40 variáveis**: as mesmas 6 e mais 34 colunas redundantes ou sem sentido.

> **Resultado em uma frase:** o modelo com excesso de variáveis vence no treino, perde no teste e ainda conclui que cada m² a mais *reduz* o preço do imóvel.

## A ideia

Em dados reais nunca sabemos a "regra verdadeira" por trás do que queremos prever. Por isso este experimento usa **dados sintéticos**: o preço de 200 imóveis é gerado por uma fórmula conhecida. Assim dá para verificar duas coisas: qual modelo erra menos e qual deles **aprende a regra certa**.

O preço de cada imóvel é formado assim:

| Componente | Efeito real no preço |
|---|---|
| Valor base | R$ 80.000 |
| `area_m2` | + R$ 3.200 por m² |
| `quartos` | + R$ 10.000 por quarto |
| `banheiros` | + R$ 18.000 por banheiro |
| `vagas_garagem` | + R$ 22.000 por vaga |
| `idade_anos` | − R$ 2.500 por ano |
| `distancia_centro_km` | − R$ 6.000 por km |
| Ruído aleatório | desvio padrão de R$ 60.000 |

O ruído representa o que nenhum modelo consegue explicar: negociação, estado de conservação, sorte. Nenhum dado real de imóveis é usado.

## As duas bases

As duas bases têm **as mesmas 200 linhas e o mesmo preço**. Só muda quais colunas entram no modelo.

- **Base selecionada:** as 6 variáveis que de fato formam o preço.
- **Base completa:** as mesmas 6 e mais 34 colunas que ficaram de fora por um destes motivos:

| Motivo da exclusão | Variáveis |
|---|---|
| Redundante (mesma informação de outro jeito) | `area_pes2`, `area_total_m2`, `idade_meses`, `distancia_centro_milhas`, `total_comodos` |
| Identificador | `id_anuncio`, `numero_imovel` |
| Sem relação plausível com o preço | `fotos_no_anuncio`, `dias_anunciado`, `temperatura_dia_visita`, `letras_nome_rua`, `idade_corretor`, cor da fachada, signo do proprietário e dia da semana do anúncio (as três últimas em colunas 0/1) |

A seleção seguiu quatro perguntas:

1. **Faz sentido para o negócio?** Signo do proprietário não muda o preço de um imóvel.
2. **É redundante?** Área em m² e em pés² são a mesma informação.
3. **É só um identificador?** ID e número da porta não descrevem o imóvel.
4. **Os dados confirmam?** VIF, validação cruzada e desempenho fora da amostra.

## Principais resultados

Regressão linear por mínimos quadrados, com a mesma divisão 70% treino / 30% teste para os dois modelos. Em negrito, o melhor resultado de cada linha.

| Métrica | 6 selecionadas | 40 (excesso) |
|---|---|---|
| R² no treino | 0,895 | **0,920** |
| R² no teste | **0,833** | 0,755 |
| RMSE no teste | **R$ 63.668** | R$ 77.059 |
| MAE no teste | **R$ 49.274** | R$ 63.852 |
| RMSE na validação cruzada (5 partes × 20 repetições) | **R$ 63.895** | R$ 73.601 |
| VIF máximo (multicolinearidade) | **3,9** | 16.967 |
| Valor de 1 m² estimado (real: R$ 3.200) | **R$ 2.992** | −R$ 31.476 |
| Intervalo de 95% do valor do m² (bootstrap) | **R$ 2.587 a R$ 3.424** | −R$ 69.163 a R$ 8.671 |

O modelo com excesso só ganha no treino, justamente onde já viu as respostas. No teste, erra 21% mais.

O valor negativo do m² vem das colunas redundantes. Com área em m², em pés² e área total no mesmo modelo, ele não consegue separar o efeito de cada uma, e o coeficiente perde o sentido.

### Foi sorte?

O experimento foi repetido em **500 bases diferentes**. O modelo com 6 variáveis teve menor erro de teste em **98,8%** delas, com R² médio no teste de 0,903 contra 0,871.

### E com mais dados?

| Imóveis na base | Erro a mais com excesso | Valor do m²: 90% das estimativas (6 variáveis) | Valor do m²: 90% das estimativas (40 variáveis) |
|---|---|---|---|
| 100 | +46% | R$ 2.730 a R$ 3.693 | −R$ 52.909 a R$ 45.328 |
| 500 | +5% | R$ 3.010 a R$ 3.432 | −R$ 13.382 a R$ 17.831 |
| 5.000 | +0,5% | R$ 3.121 a R$ 3.265 | −R$ 661 a R$ 7.325 |

Mais dados aproximam o erro dos dois modelos, mas o modelo com excesso continua sem saber quanto vale o m².

## Conclusões

- **A métrica de treino engana.** Mais colunas quase sempre aumentam o R² de treino, mesmo quando são ruído puro.
- **Variáveis sem sentido causam overfitting.** O modelo usa colunas aleatórias para decorar o ruído do treino.
- **Variáveis redundantes destroem a interpretação.** Quando duas colunas dizem a mesma coisa, os coeficientes perdem o sentido.
- **Mais dados ajudam a prever, mas não a explicar.**
- **Na vida real ninguém conhece a regra verdadeira.** Por isso a seleção combina conhecimento do negócio com validação estatística.

## Como rodar

**Requisitos:** Python 3.9 ou mais recente, com `numpy`, `pandas`, `scikit-learn` e `matplotlib`.

1. Baixe o repositório (botão **Code → Download ZIP**) ou clone com `git clone`.
2. Instale as bibliotecas:

   ```bash
   pip install numpy pandas scikit-learn matplotlib notebook
   ```

3. Abra o notebook:

   ```bash
   jupyter notebook feature_selection_regressao.ipynb
   ```

4. Rode tudo com *Kernel → Restart & Run All*. Leva cerca de 1 minuto.

Também funciona no JupyterLab, no VS Code e no Google Colab. No Colab, basta fazer upload do arquivo `.ipynb` pelo menu *Arquivo*.

Ao final, o notebook salva estes arquivos na mesma pasta:

| Arquivo | Conteúdo |
|---|---|
| `base_completa.csv` | Base com as 40 variáveis e o preço |
| `base_selecionada.csv` | Base com as 6 variáveis e o preço |
| `metricas.csv` | Todas as métricas dos dois modelos |
| `simulacoes.csv` | R² e RMSE de teste nas 500 simulações |
| `curva_tamanho_amostra.csv` | Erro e valor do m² para bases de 100 a 5.000 imóveis |

Para organizar esses arquivos em subpastas (`dados/` e `resultados/`), mude `USAR_SUBPASTAS = True` na primeira célula de código.

## O que tem no notebook

1. Configuração
2. A regra verdadeira do preço
3. Geração da base
4. Por que cada variável ficou de fora, com o VIF por variável
5. Treino e comparação dos dois modelos
6. Coeficientes estimados × valores reais, com intervalo de 95% por bootstrap
7. Repetição em 500 bases diferentes
8. Efeito do tamanho da base (100 a 5.000 imóveis)
9. Gráficos dos resultados
10. Resumo dos resultados
11. Conclusões
12. Sugestões para experimentar

## Reprodutibilidade

Todos os sorteios usam sementes fixas, então rodar de novo gera exatamente os mesmos números. O notebook foi executado com Python 3.13 e 3.14 (numpy 2.4 e 2.5, pandas 3.0, scikit-learn 1.9, matplotlib 3.11), e os resultados foram idênticos.

## Experimente

Mude os parâmetros e rode tudo de novo:

- `N_LINHAS`: com 1.000 ou 5.000 imóveis, o modelo com excesso melhora na previsão. E os coeficientes?
- `RUIDO`: com menos ruído, o overfitting fica menor ou maior?
- `SEMENTE`: veja se a conclusão se mantém com outros sorteios.
- `VARIAVEIS_SELECIONADAS`: acrescente `"area_pes2"` à lista e observe o VIF e o coeficiente da área.
- Troque `LinearRegression` por `Ridge` ou `Lasso` e compare: a regularização compensa parte do excesso de variáveis?

## Estrutura do repositório

```
.
├── feature_selection_regressao.ipynb   # experimento completo
└── README.md
```
