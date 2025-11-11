## Objetivo
Dado um conjunto de pontos de interesse (POIs) em uma cidade, estimar quantos quilômetros são suficientes para interligá-los por vias reais.  
Você deverá:
1) Modelar o grafo viário da(s) cidade(s) com OSMnx (ou equivalente);  
2) Calcular rotas mais curtas com A\* entre POIs;  
3) Construir e calcular a Árvore Geradora Mínima (MST) sobre o grafo completo entre POIs (arestas ponderadas pelo custo A\* nas vias) para obter o comprimento total mínimo necessário para conectar todos os POIs;  
4) Comparar o resultado com pelo menos 8 cidades.

## O que fazer (passo a passo)

### 1) Escolha dos POIs
- Escolha livre dos POIs (ex.: campus, museus, praças, arenas, hubs de transporte, escolas, ponto de onibus, locais turísticos, etc).
- Escolha um POI diferente do Notebook-base II.

### 2) Grafo viário da cidade
- Baixe o grafo viário com OSMnx (`graph_from_place` ou `graph_from_polygon`).  
- Projete o grafo para métrica (UTM) antes de medir distâncias (mesma estratégia usada no Notebook-base II).
- Para cada POI, pegue o nó mais próximo no grafo.

### 3) Rotas mais curtas com A\*
- Para cada par de POIs da cidade, compute o caminho mínimo usando A\* (heurística: distância em linha reta/“great-circle” ou Euclidiana no plano projetado).  
- Registre o custo (distância) de cada par e guarde a rota (lista de nós/arestas) para visualização posterior.  
  - Pode usar `networkx.astar_path`/`astar_path_length` ou implementar A\*.

### 4) MST sobre o grafo completo de POIs
- Com os pesos = distâncias A\* entre cada par de POIs, forme um grafo completo (POIs como vértices).  
- Calcule a MST (ex.: Kruskal) e some os pesos das arestas da MST → “km suficientes para ligar os POIs”.  
- Mapeie de volta cada aresta da MST para a rota na malha viária (união das rotas A\* correspondentes) e compute também o comprimento total real dessa rede resultante.

### 5) Comparação entre ≥ 8 cidades
- Repita os passos 1–4 em pelo menos 8 cidades (pode incluir Natal como uma delas).  
- Compare:  
  - Comprimento da MST (km);   
  - Média e Desvio Padrão por POI (km/POI) ou por par conectado da MST (km/aresta-MST);  

### 6) Visualização e análise
- Plote, para cada cidade, o subgrafo final (união das rotas A\* que compõem a MST entre POIs).  
- Produza tabela comparativa consolidando as métricas.  
- Escreva análise crítica (≈ 10–15 linhas): por que certas cidades exigem mais/menos km? Efeito da escolha de POIs? Limitações do método? O texto deverá estar no arquivo Markdown (README.md)