# Sistema de Recomendação de Cursos com IA

Este projeto implementa um sistema de recomendação de cursos baseado no perfil profissional do usuário,
utilizando **recursão**, **memoização** e o **algoritmo da mochila 0/1** para montar um plano de estudos ótimo.

---

## 1. Formulação do Problema

**Objetivo:**

Montar uma trilha de cursos de IA que maximiza a **relevância total** para o profissional,
respeitando duas restrições principais:

- Horas máximas de estudo disponíveis (`horas_max`);
- Orçamento máximo para investir em cursos (`orcamento_max`).

**Entradas:**

- Área de atuação (saúde, direito, educação, engenharia, administração ou genérico);
- Prioridade dada ao tema IA (1 a 10);
- Estilo de estudo (curto e barato, equilibrado ou aprofundado).

Essas escolhas definem os parâmetros do problema de mochila.

**Saídas:**

- Catálogo de cursos personalizados com relevância calculada;
- Conjunto de cursos selecionados (trilha ótima);
- Relatório com horas usadas, orçamento gasto e relevância total.

---

## 2. Fluxo Geral da Aplicação

1. O usuário executa `executar_sistema()`;
2. O menu principal pergunta se ele deseja montar um plano de estudos;
3. Se sim, `coletar_perfil_usuario()` pergunta área, prioridade e estilo de estudo;
4. Com esses dados, `montar_plano_otimo(...)`:
   - Gera o catálogo (`montar_lista_cursos`);
   - Calcula relevância (`calcular_relevancia_cursos`);
   - Ordena os cursos por relevância/hora (`merge_sort_lista` com recursão + memo);
   - Aplica o algoritmo da mochila 0/1 (`knapsack` com recursão + lru_cache);
   - Reconstrói a solução (lista de cursos escolhidos).
5. O sistema exibe:
   - Catálogo personalizado;
   - Trilha ótima;
   - Resumo (horas usadas, orçamento gasto, relevância total).

---

## 3. Explicação das Funções

### 3.1 `mostrar(texto)`

Função auxiliar para exibir mensagens formatadas no Jupyter Notebook.

- Usa `IPython.display.HTML` para mostrar o texto dentro de `<pre>`, mantendo quebras de linha;
- É usada em menus, títulos de seção e resumo final;
- Não tem lógica algorítmica (apenas saída), por isso não usa recursão.

---

### 3.2 `montar_lista_cursos(area_foco)`

Esta função monta o catálogo dinâmico de cursos utilizado pelo sistema.

Ela inicia com três cursos gerais e, em seguida, utiliza um dicionário contendo cinco cursos específicos para cada área (saúde, direito, educação, engenharia e administração).

O comportamento é o seguinte:

- **Se a área escolhida estiver no dicionário:**
  - Todos os cursos dessa área são adicionados ao catálogo;
  - Além disso, três cursos de cada uma das demais áreas também são adicionados, garantindo diversidade;
  - O catálogo final sempre contém mais de 20 cursos.

- **Se a área for genérica (“todas”) ou inválida:**
  - Cursos de todas as áreas são adicionados.

Cada curso contém os campos `curso`, `area`, `horas`, `preco` e `impacto`, servindo de base para o cálculo de relevância e para a etapa da mochila.


---

### 3.3 `coletar_perfil_usuario()`

Lida com toda a coleta de parâmetros do problema via menu textual.

Pergunta ao usuário:

1. **Área de atuação** (1 a 6) → mapeada para strings como `'saúde'`, `'direito'` etc.
2. **Prioridade em IA** (1 a 10) → trata entradas inválidas e força o valor para ficar entre 1 e 10.
3. **Estilo de estudo** (1 a 3):
   - 1: curto e barato → poucas horas e baixo orçamento;
   - 2: equilibrado → horas e orçamento intermediários;
   - 3: aprofundado → mais horas e maior orçamento.

Retorna:

```python
area_foco, prioridade, horas_max, orcamento_max
```

Esses valores alimentam diretamente `montar_plano_otimo()`.

---

### 3.4 `calcular_relevancia_cursos(cursos, area_foco, prioridade)`

Enriquece o catálogo de cursos com um campo adicional: `relevancia`.

Para cada curso:

1. Começa em `base = curso["impacto"]`;
2. Se a área do curso for igual à área do profissional → `bonus_area = 2`, senão `1`;
3. Calcula `extra = prioridade // 5`, que pode ser 0, 1 ou 2;
4. Soma tudo e limita em 10:

```python
relevancia = min(10, base + bonus_area + extra)
```

Retorna uma nova lista de cursos, agora cada um com o campo `"relevancia"`.
Essa relevância será usada como função de valor no problema da mochila.

---

### 3.5 `merge_sort_lista(lista, chave, memo)`

Implementa o **Merge Sort recursivo com memoização** para ordenação dos cursos.

**Parâmetros:**

- `lista`: lista de elementos (no caso, dicionários de cursos);
- `chave`: função que recebe um curso e retorna o valor usado na comparação;
- `memo`: dicionário de cache, onde a chave é uma tupla com os IDs dos elementos da lista.

**Passos:**

1. Constrói `tupla = tuple(id(x) for x in lista)` e verifica se já existe em `memo`.
   - Se existir, retorna a versão já ordenada → **memoização manual**.
2. Se o tamanho da lista for 0 ou 1, é o **caso base da recursão**:
   - Armazena no memo e retorna a própria lista.
3. Para listas maiores:
   - Calcula o meio: `meio = len(lista) // 2`;
   - Chama recursivamente para as duas metades:
     - `e = merge_sort_lista(lista[:meio], chave, memo)`
     - `d = merge_sort_lista(lista[meio:], chave, memo)`
4. Faz o **merge** (intercalação) das duas metades ordenadas, comparando `chave(e[i])` e `chave(d[j])` até consumir todos os elementos.
5. Salva o resultado em `memo[tupla]` e retorna a lista ordenada.

Na aplicação, a chave usada é:

```python
lambda c: c["relevancia"] / c["horas"]
```

Ou seja, cursos com maior relevância por hora tendem a ficar mais bem posicionados.

---

### 3.6 `montar_plano_otimo(horas_max, orcamento_max, area_foco, prioridade)`

É a função que resolve o problema principal usando o algoritmo da **mochila 0/1** com recursão e memoização.

#### Etapas internas

1. **Gera o catálogo de cursos** da área:
   ```python
   cursos = montar_lista_cursos(area_foco)
   ```

2. **Calcula a relevância de cada curso**:
   ```python
   cursos = calcular_relevancia_cursos(cursos, area_foco, prioridade)
   ```

3. **Ordena os cursos** por relevância/hora usando Merge Sort recursivo com memo:
   ```python
   memo_sort = {}
   cursos_ordenados = merge_sort_lista(
       cursos,
       lambda c: c["relevancia"] / c["horas"],
       memo_sort
   )
   ```

4. **Define a função recursiva de mochila 0/1 com memoização automática**:

   ```python
   @lru_cache(maxsize=None)
   def knapsack(i, h, o):
       if i == len(cursos_ordenados) or h == 0 or o == 0:
           return 0

       curso = cursos_ordenados[i]
       melhor = knapsack(i+1, h, o)  # não pega o curso

       if curso["horas"] <= h and curso["preco"] <= o:
           melhor = max(
               melhor,
               curso["relevancia"] + knapsack(i+1, h-curso["horas"], o-curso["preco"])
           )
       return melhor
   ```

   - `i`: índice do curso atual;
   - `h`: horas restantes;
   - `o`: orçamento restante.

   A função tenta decidir entre **pegar** ou **não pegar** o curso i.  
   O decorador `@lru_cache` faz a memoização: estados `(i, h, o)` já calculados não são recomputados.

5. **Reconstrução da solução ótima**

   Após obter o valor máximo com `knapsack(0, horas_max, orcamento_max)`, o código percorre os índices
   e verifica se cada curso faz parte da solução:

   ```python
   escolhidos = []
   h = horas_max
   o = orcamento_max
   for i in range(len(cursos_ordenados)):
       if knapsack(i, h, o) != knapsack(i+1, h, o):
           escolhidos.append(cursos_ordenados[i])
           h -= cursos_ordenados[i]["horas"]
           o -= cursos_ordenados[i]["preco"]
   ```

   Se o valor do estado muda ao pular o curso (`i` → `i+1`), significa que aquele curso contribuiu para a solução ótima,
   então ele é adicionado à lista `escolhidos` e as restrições `h` e `o` são atualizadas.

6. **Retorno**

   A função retorna:

   - `df_catalogo`: DataFrame com todos os cursos ordenados;
   - `df_escolhidos`: DataFrame com os cursos da trilha ótima;
   - `valor_max`: relevância total máxima encontrada pela mochila;
   - `horas_usadas`: horas realmente usadas;
   - `orcamento_gasto`: valor realmente gasto.

---

### 3.7 `executar_sistema()`

Função responsável pelo **menu principal** e pela interação geral com o usuário.

- Exibe o título da plataforma e as opções (montar plano ou sair);
- Lê a opção digitada pelo usuário;
- Se for `1`, chama toda a cadeia de funções:
  - `coletar_perfil_usuario()`
  - `montar_plano_otimo(...)`
- Exibe:
  - O catálogo personalizado de cursos;
  - A trilha recomendada (se houver);
  - O resumo com horas usadas, orçamento gasto e relevância total.

Se o usuário escolher `0`, o sistema mostra uma mensagem de saída e encerra o laço.

---

## 4. Recursão e Memoização na Prática

O código usa recursão + memoização de forma explícita em dois pontos essenciais:

1. **Ordenação (Merge Sort)** — `merge_sort_lista()`
   - Recursão: divide a lista em sublistas até o caso base (0 ou 1 elemento);
   - Memoização: reutiliza resultados para sublistas já ordenadas, usando um dicionário `memo`.

2. **Mochila 0/1 (Knapsack)** — função interna `knapsack(i, h, o)` em `montar_plano_otimo()`
   - Recursão: modelo clássico de programação dinâmica top-down;
   - Memoização: feita via `@lru_cache`, armazenando resultados por estado `(i, h, o)`.

Esses dois blocos cumprem diretamente o que o enunciado exige sobre o uso de recursão e memoização
nos algoritmos principais da solução (ordenação + mochila).

---

## 5. Estrutura de Saída e Relatórios

A aplicação exibe os resultados em três blocos principais:

1. **Catálogo de cursos personalizado** (DataFrame):
   - Mostra todos os cursos considerados, com área, horas, preço e relevância.

2. **Melhor trilha de cursos** (DataFrame):
   - Lista apenas os cursos escolhidos pelo algoritmo da mochila 0/1.

3. **Resumo textual**:
   - Horas usadas no plano;
   - Orçamento efetivamente gasto;
   - Relevância total alcançada.

Essa saída permite que o avaliador veja claramente:
- Como o problema foi modelado;
- Quais decisões o algoritmo tomou;
- Se os recursos (tempo e dinheiro) foram bem aproveitados.


Integrantes:

-Lucca Borges RM554608

-Ruan Vieira RM557599

-Rodrigo Carnevale RM55814
