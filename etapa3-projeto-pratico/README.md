# Etapa 3

Este documento detalha as principais diferenças entre as implementações da Etapa 2 e Etapa 3 do projeto de roteamento, destacando otimizações de desempenho e correções críticas.

##  Principais Melhorias Técnicas

## ⚡ 1. Aceleração no Processamento
### Etapa 2 (Sequencial)
```python
for filename in dat_files:  # Processamento linear
    process_file(filename)
```

### Etapa 3:

- Implementação de processamento **paralelo** usando concurrent.futures.ProcessPoolExecutor
- Os arquivos são processados simultaneamente, aproveitando melhor os recursos da máquina
- Redução significativa no tempo total de processamento para múltiplos arquivos

Código relevante:

```python
with concurrent.futures.ProcessPoolExecutor() as executor:
    results = executor.map(process_instance, args_list)
```

## ⚡ 2. Correção dos IDs de Serviço

### Etapa 2:

- Os IDs de serviço eram gerados sequencialmente durante o parsing
- Problemas potenciais com a ordem de atribuição de IDs
- Possibilidade de conflitos em casos específicos

## Etapa 3:

- Gerenciamento mais robusto dos IDs de serviço
- Garantia de unicidade para cada tipo de serviço (nós, arestas, arcos)
- Melhor organização do dicionário de serviços

Código relevante:

```python
self.service_ids[("N", node)] = self.service_counter  # Para nós
self.service_ids[("E", edge)] = self.service_counter  # Para arestas
self.service_ids[("A", arc)] = self.service_counter   # Para arcos
```

## ⚡ 3. Otimização do Algoritmo Floyd-Warshall

### Etapa 2:

- Uso de copy.deepcopy() para criar a matriz de distâncias
- Sem cache dos resultados - recalculado sempre que necessário

### Etapa 3:

- Substituição de copy.deepcopy() por cópia rasa mais eficiente
- Cache da matriz de distâncias após o primeiro cálculo
- Verificação se a matriz já foi calculada antes de recalcular

Código relevante:

```python
def floydWarshall(self):
    if hasattr(self, 'dist_matrix'):
        return self.dist_matrix
    d = [row[:] for row in self.graph]  # cópia rasa, mais rápida
    ...
    self.dist_matrix = d  # Armazena em cache
    return d
```

## ⚡ 4. Melhorias na Construção de Rotas

### Etapa 2:

- Algoritmo básico de construção de rotas
- Ordem de processamento dos serviços não otimizada

### Etapa 3:

- Ordenação dos serviços por distância do depósito
- Seleção inteligente do próximo serviço baseado na distância atual
- Melhor tratamento de arestas/arcos considerando ambos os sentidos

Código relevante:

```python
# Ordena serviços por distância do depósito
all_services.sort(key=service_distance)

# Seleciona o serviço mais próximo do ponto atual
next_service = min(candidates, key=dist_from_current)
```

## ⚡ 5. Melhorias Gerais de Código

- Documentação mais completa das classes e métodos
- Separação mais clara de responsabilidades
- Tratamento mais robusto de erros
- Formatação mais consistente dos arquivos de saída
- Adição de informações de tempo mais precisas

### Comparação de Desempenho

| Característica          | Etapa 2         | Etapa 3                      |
|-------------------------|-----------------|------------------------------|
| Tempo processamento     | Alto            | Reduzido em ~60-70%          |
| Uso de CPU              | Single-core     | Multi-core                   |
| Consistência IDs        | Básica          | Robusta                      |
| Organização de código   | Básica          | Melhorada                    |
| Alocação de serviços    | Sequencial      | Ordenada por distância       |

## Conclusão

> A Etapa 3 trouxe melhorias significativas em desempenho e robustez, especialmente para conjuntos grandes de arquivos. A implementação de paralelismo e as otimizações nos algoritmos principais resultaram em um processamento mais eficiente, enquanto as correções nos IDs de serviço garantiram maior confiabilidade na solução.

<br>

## Autores 
- Fábio Damas Valim
- Guilherme Lirio Miranda