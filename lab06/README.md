# Equipe Data Miners

# Subgrupo A
- Lucio Bueno Vieira Junior - 221029
- Guilherme Sampaio Cintra - 248313
- Guilherme de Oliveira Zaleski - 235914

## Tarefa de Cypher sobre Patologias, Medicamentos e Efeitos Colaterais

## Exercício

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (d)-[t1:Treats]->(p1), (d)-[t2:Treats]->(p2)
WHERE t1.weight > 20 AND t2.weight > 20
MERGE (p1)<-[r:P_Relates]->(p2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~

# Trabalhando com Efeitos Colaterais

## Exercício

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
* Rodar em células diferentes

* Criação de uma relação de pessoas que tomam determinado remédio
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MERGE (d:Drug {code: line.codedrug})
MERGE (p:Person {id: line.idperson})
MERGE (p)-[:TAKES]->(d)
~~~

* Criação de uma relação de pessoas que tiveram efeito colateral
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MERGE (s:Pathology {code: line.codePathology})
MERGE (p:Person {id: line.idPerson})
MERGE (p)-[:HAS_SIDE_EFFECT]->(s)
~~~

* Relacionamento entre os dados anteriores
~~~cypher
MATCH (p:Person)-[:TAKES]->(d:Drug), (p)-[:HAS_SIDE_EFFECT]->(pa:Pathology)
MERGE (d)-[c:Causes]->(pa)
ON CREATE SET c.weight = 1
ON MATCH SET c.weight = c.weight + 1
~~~

## Exercício

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
Avaliando um peso minimo na relação de causa de efeito colateral, podemos analisar as drogas que mais manifestam um tipo de sintoma, resultando em uma informação teoricamente mais embasada.

Por exemplo: pode ser observado que o medicamento "Risperdal" e a "Risperidona" que é seu princípio ativo estão constantemente ligados ao sintomas de ginecomastia.

~~~cypher
MATCH p=()-[c:Causes]->() 
WHERE c.weight > 25
RETURN p
~~~
