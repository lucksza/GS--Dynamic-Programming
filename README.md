# Sistema de Recomendação de Cursos com Programação Dinâmica

Este projeto implementa um **sistema de recomendação de cursos de IA** baseado no perfil profissional do usuário, utilizando:

- **Recursão e memoização** em todas as funções principais;
- **Merge Sort recursivo** para ordenação do catálogo de cursos;
- **Algoritmo da mochila 0/1 (knapsack)** para montar um plano de estudos ótimo;
- Saída em **DataFrames e resumo textual** com horas, orçamento e relevância.

O código foi desenvolvido em função do tema da **Global Solution** e atende ao enunciado da disciplina **Dynamic Programming**, incluindo:

- Formulação do problema (entradas, saídas, objetivo);
- Estrutura de ordenação em dataframe (merge sort);
- Uso de recursão + memoização;
- Uso da ideia da mochila;
- Apresentação de resultados e relatórios;
- Explicação das funções / estruturas criadas;
- Catálogo com **20+ cursos diferentes**.

---

## 1. Formulação do Problema

### Objetivo

Montar uma **trilha de cursos de IA** que maximize a **relevância total** (impacto ajustado) para o profissional, respeitando:

- Limite de **horas semanais de estudo**;
- Limite de **orçamento máximo** para investimento em cursos.

### Entradas

O sistema pergunta ao usuário:

- **Área de atuação**:  
  `saúde`, `tecnologia`, `gestão`, `educação` ou `todas`;
- **Prioridade** em relação à IA:  
  `carreira`, `atualização` ou `transição`;
- **Estilo de estudo**:  
  `rápido`, `profundo` ou `equilibrado`;
- **Horas máximas disponíveis** (`horas_max`);
- **Orçamento máximo disponível** (`orcamento_max`).

### Saídas

- **Catálogo personalizado de cursos**, com campo `impacto_ajustado`;
- **Catálogo ordenado** por impacto ajustado (decrescente), usando **merge sort recursivo**;
- **Plano ótimo de estudos**, calculado com o algoritmo da **mochila 0/1**, respeitando horas e orçamento;
- **Relatório final** com:
  - Horas usadas;
  - Orçamento gasto;
  - Relevância total (impacto ajustado) do plano.

---

## 2. Visão Geral da Solução

Fluxo principal:

1. O usuário executa `executar_sistema()`;
2. O sistema mostra um **menu recursivo** com opções:
   - (1) Informar/atualizar perfil profissional;
   - (2) Gerar catálogo personalizado e ordenar;
   - (3) Calcular plano ótimo pela mochila;
   - (0) Sair.
3. Ao informar o perfil, o sistema:
   - Monta um catálogo com **20+ cursos** via `montar_lista_cursos`;
   - Ajusta o impacto de cada curso para o perfil do usuário em `calcular_relevancia_cursos`, gerando `impacto_ajustado`;
   - Ordena o catálogo por impacto ajustado via `merge_sort_lista`;
   - Aplica a mochila 0/1 em `montar_plano_otimo` para escolher os cursos que maximizam a relevância total dentro das restrições.
4. O resultado é exibido em:
   - DataFrame com o catálogo personalizado;
   - DataFrame com o plano ótimo;
   - Resumo textual com horas, orçamento e relevância.

---

## 3. Descrição Resumida das Funções

### `mostrar(texto)`

- Função auxiliar de interface.
- Divide o texto em linhas, usa **recursão** para imprimir linha a linha.
- Usa `@lru_cache` para memorizar cada linha (memoização).
- Ajuda na exibição de menus e relatórios.

---

### `montar_lista_cursos(area_foco)`

- Monta o catálogo de cursos com base na área escolhida.
- Usa um dicionário `CURSOS_BASE` com cursos de **saúde**, **tecnologia**, **gestão** e **educação** (6 cursos por área, com impacto base, horas e preço).
- Se `area_foco == "todas"`, junta recursivamente os cursos de todas as áreas.
- Usa `@lru_cache` em `cursos_por_area(area)` para evitar recomputar listas de cursos por área.
- Garante mais de 20 cursos no catálogo (requisito do enunciado).

---

### `coletar_perfil_usuario()`

- Pergunta recursivamente:
  - área;
  - prioridade (carreira/atualização/transição);
  - estilo (rápido/profundo/equilibrado);
  - horas_max;
  - orcamento_max.
- Usa um dicionário `memo_respostas` para armazenar respostas intermediárias.
- Retorna um dicionário `perfil` com todas as informações usadas pelo restante da solução.

---

### `calcular_relevancia_cursos(cursos, area_foco, prioridade, estilo)`

- Calcula o **impacto ajustado** (`impacto_ajustado`) de cada curso, que é a **relevância personalizada** para aquele usuário.
- Para cada curso:
  - Leva em conta:
    - impacto base (`impacto`);
    - prioridade (peso de 1.0 a ~1.3);
    - estilo de estudo (peso de 1.0 a ~1.2);
    - bônus se o curso for da mesma área que o usuário.
- Usa funções auxiliares com memoização:
  - `peso_prioridade(p)` com `@lru_cache`;
  - `peso_estilo(e)` com `@lru_cache`.
- Percorre a lista de cursos **recursivamente** em `processar_indice(i)`.
- **Limita** o `impacto_ajustado` a **no máximo 10**, atendendo à ideia de uma nota máxima.
- Retorna nova lista de cursos com o campo `impacto_ajustado`, que será usado na ordenação e na mochila.

---

### `merge_sort_lista(lista, chave)`

- Implementa **Merge Sort recursivo** com **memoização**.
- Usa a função interna `sort_interval(start, end)` decorada com `@lru_cache(maxsize=None)`:
  - Caso base: sublista de tamanho 0 ou 1;
  - Caso recursivo: divide em duas metades, ordena recursivamente e intercala.
- Ordena a lista de dicionários **em ordem decrescente** pela chave (`chave`).
- No projeto, a chave utilizada é o campo `"impacto_ajustado"`.

---

### `montar_plano_otimo(cursos, horas_max, orcamento_max)`

- Resolve o problema da **mochila 0/1 com duas restrições**:
  - Tempo (horas);
  - Orçamento (R$).
- Usa uma função interna recursiva, com memoização:
  ```python
  @lru_cache(maxsize=None)
  def knapsack(i, horas_restantes, orcamento_restante):
      ...
  ```
- Estados:
  - `i`: índice do curso;
  - `horas_restantes`;
  - `orcamento_restante`.
- Em cada estado, decide recursivamente **pegar ou não pegar** o curso i, maximizando a soma dos `impacto_ajustado`.
- Depois reconstrói a lista de cursos escolhidos com a função recursiva `reconstruir(...)`.
- Retorna:
  - lista de cursos escolhidos;
  - horas usadas;
  - orçamento gasto;
  - impacto total máximo.

---

### `executar_sistema()`

- Ponto de entrada da aplicação.
- Monta um **menu em loop recursivo** (`loop()` chama ele mesmo até o usuário escolher sair).
- Usa um dicionário `cache` para:
  - armazenar o perfil;
  - guardar catálogo gerado;
  - guardar catálogo ordenado;
  - guardar plano ótimo.
- Chama, conforme a opção:
  - `coletar_perfil_usuario`;
  - `montar_lista_cursos`;
  - `calcular_relevancia_cursos`;
  - `merge_sort_lista`;
  - `montar_plano_otimo`.
- Exibe:
  - DataFrame do catálogo;
  - DataFrame do plano ótimo;
  - Resumo textual das decisões.

---

## 4. Recursão e Memoização

A solução usa **recursão + memoização** de forma explícita em todas as partes centrais:

- `mostrar`: recursão para percorrer linhas, memoização nas linhas;
- `montar_lista_cursos`: recursão para juntar áreas, memoização por área (`cursos_por_area`);
- `coletar_perfil_usuario`: recursão na sequência de perguntas, memo das respostas;
- `calcular_relevancia_cursos`: recursão para percorrer cursos; `@lru_cache` para pesos;
- `merge_sort_lista`: recursão no Merge Sort; `@lru_cache` em subintervalos (`start`, `end`);
- `montar_plano_otimo`: recursão na mochila (função `knapsack`); `@lru_cache` nos estados;
- `executar_sistema`: loop recursivo `loop()` + uso de cache para evitar recomputar.

Isso atende diretamente ao requisito:  
> “Todas funções devem ser criadas com recursão e memoização”.

---

## 5. Como Executar o Projeto

1. Abrir o código em um **Jupyter Notebook** ou **Google Colab**;
2. Garantir que as bibliotecas abaixo estão instaladas:
   - `pandas`
   - `IPython.display` (já vem no Jupyter/Colab);
3. Executar a célula com todo o código da solução;
4. Rodar:

```python
executar_sistema()
```

5. Seguir o menu interativo para:
   - informar o perfil;
   - gerar catálogo;
   - calcular plano ótimo.

---

## 6. Estrutura do Repositório

Sugestão de estrutura para o GitHub:

```text
.
├── codigo.ipynb          # Notebook com todo o código da solução
├── README.md             # Visão geral, problema, funções e execução
└── Documentacao.pdf      # Documento detalhado explicando função por função
```

Integrantes:

- **Lucca Borges – RM 554608**  
- **Ruan Vieira – RM 557599**  
- **Rodrigo Carnevale – RM 55814**
