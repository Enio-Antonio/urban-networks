## Análise de Rede Mínima para Pontos de Interesse (POIs) Urbanos
Este projeto utiliza Python, OSMnx e NetworkX para estimar o comprimento mínimo da rede de vias necessárias para interconectar um 
conjunto de Pontos de Interesse (POIs) em diversas cidades.

O script foca em __shoppings__ como POIs de exemplo, mas pode ser adaptado para qualquer tag do OpenStreetMap. O objetivo é encontrar 
a Árvore Geradora Mínima (MST) com base nas distâncias *reais* de rua (calculadas com A*), fornecendo uma estimativa de 
*infraestrutura* de rede.

## Índice
* [Pre-requisitos](#-pré-requisitos)
* [Como usar](#-como-usar)
* [Metodologia](#️-metodologia-detalhada)
* [Saída esperada](#-saída-esperada)
* [Resultados](#-resultados)
    * [Tabela de resultados](#tabela-de-resultados)

## 📋 Pré-requisitos
Certifique-se de ter o Python 3.8+ instalado. As bibliotecas necessárias podem ser instaladas via ```pip```:
```bash
pip install osmnx networkx pandas numpy matplotlib geopandas
```

## 🚀 Como Usar
1. Clone este repositório.
1. Certifique-se de que todas as bibliotecas acima estão instaladas.
1. Execute o script Python
    ```bash
    python main.py
    ```
1. O script irá processar cada cidade da lista ```cities```. Você pode editar esta lista para adicionar ou remover localidades.

## ⚙️ Metodologia Detalhada
O script executa uma análise em 6 passos principais para cada cidade:
1. __Escolha dos POIs (Shoppings)__

    O script baixa os dados de "features" do OpenStreetMap usando a tag ```{'shop': 'mall'}``` para identificar os shoppings na área 
    da cidade.

2. __Grafo Viário e Mapeamento__ 
    * Baixa o grafo viário (ruas e estradas) da cidade usando ```osmnx.graph_from_place```.
    * Projeta o grafo para um sistema de coordenadas métrico (UTM) para permitir cálculos de distância precisos em metros.
    * Para cada POI encontrado, identifica o nó (intersecção) mais próximo na malha viária.

3. __Rotas Mais Curtas (Algoritmo A*)__
    * Para cada par de POIs únicos, o script calcula o caminho de rua mais curto usando o algoritmo __A*__ (implementado em 
    ```networkx.astar_path_length```).
    * A heurística utilizada é a distância Euclidiana no plano projetado (UTM), o que garante a otimalidade do A*.
    * As distâncias reais de rua (o "custo" do A*) são armazenadas.

4. __MST sobre o Grafo Completo de POIs__
    * Um novo grafo "virtual" completo é criado, onde os POIs são os vértices.
    * O peso de cada aresta neste grafo é a distância de rua (custo A*) calculada no passo anterior.
    * O script calcula a __Árvore Geradora Mínima (MST)__ (usando o algoritmo de Kruskal) sobre este grafo completo.
    * A soma dos pesos das arestas da MST representa a __distância teórica mínima__ (em km) para interligar todos os shoppings.

5. __Cálculo de Métricas (MST vs. Rede Real)__

    O script calcula duas métricas principais de comprimento:

    1. ```Compr. Total MST (km)```: A soma das distâncias A* que compõem a MST. Esta é a "estimativa" teórica.
    1. ```Compr.Rede Real (km)```: A soma do comprimento das ruas reais usadas. Isso é feito pegando todas as rotas A* da MST, unindo 
    todos os segmentos de rua e somando seus comprimentos. Este valor é menor ou igual ao da MST, pois segmentos de rua compartilhados 
    por duas rotas diferentes são contados apenas uma vez.

6. __Comparação e Visualização__
    * As métricas de todas as cidades são compiladas em uma tabela comparativa.
    * Métricas normalizadas (como ```km/POI```) são adicionadas para permitir uma comparação mais justa entre cidades de tamanhos 
    diferentes.

## 📊 Saída Esperada

O script produz duas saídas principais:

1. __Tabela Comparativa (no Console)__

    Uma tabela em Markdown é impressa no final da execução, consolidando os resultados.

2. __Visualizações (Gráficos)__

    Para cada cidade processada com sucesso, uma janela ```matplotlib``` será aberta, mostrando o mapa de ruas (em cinza) com a rede 
    mínima otimizada (as rotas da MST) destacada em vermelho.

## 📈 Resultados
A análise dos fatores de custo e limitações do método MST + A* mostra que o comprimento da rede viária mínima entre pontos de 
interesse (POIs) varia conforme três aspectos principais: morfologia urbana, topografia e eficiência da malha viária.

A escolha dos POIs define a escala da análise. Usar shoppings (shop=mall) representa uma rede regional, conectando grandes polos 
comerciais — poucas conexões longas, avaliando a infraestrutura principal. Já escolas (amenity=school) criariam uma rede densa e 
local, medindo a eficiência da malha viária de bairro. Hospitais (amenity=hospital) ficariam entre esses dois extremos. Assim, mudar o 
tipo de POI muda completamente a pergunta investigada.

Quanto às limitações do método, a MST representa apenas a rede de menor quilometragem, sem redundância — diferente das redes reais, 
que precisam de rotas alternativas para garantir resiliência. O algoritmo A* usa distância como custo, ignorando o tempo de 
deslocamento, o que pode levar a rotas curtas, porém lentas. Além disso, a qualidade dos resultados depende fortemente dos dados do 
OpenStreetMap, que podem conter lacunas ou classificações incorretas. Há também simplificações de modelagem, como o uso de nós 
próximos ao POI (nem sempre o ponto de acesso real) e a suposição de custo uniforme por km, o que não reflete diferenças locais (ex.: 
túneis x avenidas planas).

Em síntese, o método MST + A* é valioso para análises comparativas de alto nível, permitindo identificar padrões de espalhamento 
urbano e o impacto de barreiras geográficas. Contudo, para fins de planejamento urbano real, ele deve ser complementado por 
informações sobre tráfego, redundância e custos reais de infraestrutura.

### Tabela de Resultados
| Cidade         |   POIs (Shoppings) |   Compr. Total MST (km) |   Compr. Rede Real (km) |   MST (km/POI) |   Rede Real (km/POI) |   Compr. Médio Aresta MST (km) |   Desv. Padrão Aresta MST (km) |
|---------------|-------------------|------------------------|------------------------|---------------|---------------------|-------------------------------|-------------------------------|
| São Luís       |                124 |                   94.57 |                   85.61 |           0.76 |                 0.69 |                           0.77 |                           0.91 |
| João Pessoa    |                 37 |                   30.24 |                   28.83 |           0.82 |                 0.78 |                           0.84 |                           0.98 |
| Natal          |                 74 |                   61.8  |                   58.07 |           0.84 |                 0.78 |                           0.85 |                           1.04 |
| Aracaju        |                 31 |                   35.18 |                   33.97 |           1.13 |                 1.1  |                           1.17 |                           1.09 |
| Maceió         |                 21 |                   34.95 |                   34.03 |           1.66 |                 1.62 |                           1.75 |                           1.21 |
| Teresina       |                 10 |                   21.06 |                   19.98 |           2.11 |                 2    |                           2.34 |                           1.37 |
| Palmas         |                  4 |                   14.6  |                   14.32 |           3.65 |                 3.58 |                           4.87 |                           3.52 |
| Fortaleza |                122 |                  112.68 |                  104.79 |           0.92 |                 0.86 |                           0.93 |                           0.81 |
| Recife    |                 32 |                   58.64 |                   54.96 |           1.83 |                 1.72 |                           1.89 |                           1.43 |