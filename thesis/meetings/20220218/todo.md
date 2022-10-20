# General
ler o artigo do Professor e perceber a estrutura de um artigo tipo

Montar a estrutura do nosso artigo, e indicar que topicos vao ser discutidos em cada secção.


# Abstract
Problema que estamos a resolver

Como se resolveu

# Introduction
Relevancia dos message brokers nos processos de sistemas distribuidos.

Cloud por forma a associar custos a instancias reservadas

Necessidade de otimizar com base na quantidade de recursos na cloud (carga)

enquadra-se no BPP, modelou-se com a variação do tamanho dos bins com o tempo

a abordagem que foi feita + contribuições

algoritmo que resolve

validado em ambiente real

# Problem Formulation
Explaining the context 
Definição do Problema BPP

Definição das variantes deste problem

# Related Work
Apresentação dos algoritmos relacionados do Offline BPP

Apresentação dos algoritmos também relativo ao TBPP

Dizer quais os algoritmos que vamos usar para comparar os resultados

Apontar as diferenças entre os modelos que ja foram implementados, e o modelo que vamos precisar para o nosso problema.

# Modelo 
como é que formulamos o nosso problema apresentar 
figura

formulação matemática

# Experimentação
Infrastrutura

Componentes 

Metricas

Geraç

Testes do algoritmo em simulação 
	- Explicar como é que geramos os dados
	- Estudar a distribuição de poisson para simular os dados
		- Ver como podemos aplicar

Testes do sistema em caso real
Subsecção para indicarmos quais as metricas de avaliação
	- Indicar como é que estamos a avaliar o sistema para fazermos a ponte com a simulação

# Conclusões
Fala-se do problema

Metodologia

Resultados principais

Trabalho futuro

Criar uma secção para termos o algoritmo e o Rscore.




Relativo ao problema podemos ter filas de mensagens que têm uma ordem e tem de ser processdado naquela ordem logo nao podemos distribuir as mensagens entre varias filas.

Queremos equilibrar o processamento (load balancing) das mensagens entre os varios consumidores.

Queremos minimizar a quantidade de consumidores que temos ao mesmo tempo que temos em conta o custo de redistribuição.

O problema enquadra-se no Bin Packing para depois conseguirmos distribuir o numero de partições para os consumidores bins.

O bin packing ja foi muito estudado... Este problema apresenta uma nova dimensão que é o espetro temporal, o que força que possa haver ajustes.

A nossa semelhança ao related work é o facto de que as nossas tarefas podem ir chegando ao longo do tempo (temporal BPP) e depois os nossos items podem aumentar de tamanho e serem redistribuidas entre servidores.

à semelhança da nossa aplicação, o caso do Kubernetes seria uma outra aplicação do trabalho desenvolvido.

Porque é que o RabbitMQ nao resolve este problema?
Porque é que o Kafka não resolve este problema? 
Que outras soluções tentaram resolver este problema.

2. Related Work

A nossa semelhança ao related work é o facto de que as nossas tarefas podem ir chegando ao longo do tempo (temporal BPP) e depois os nossos items podem aumentar de tamanho e serem redistribuidas entre servidores.



Dar enfase ao problema do Bin Packing.

Usar o caso do Kafka como um exemplo de aplicação.

Porque nao podemos usar outro sistema para resolver o nosso problema.

RabbitMQ garante order das mensagens para uma mesma queue.

Como isto podia ser aplicado tambem em outro sistemas: Kubernetes

Dynamic Bin Packing For Publish/Subscribe

Converter imagens de svg para pdf: 
https://stackoverflow.com/questions/1890215/getting-r-plots-into-latex
![[convert_to_pdf.png]]

# Introdução

# Problem Formulation

# Related Work
The Bin Packing problem has been extensively reviewed in the literature. 

# Algoritmo
## Rscore
## Modified Any Fit

# Aplicação no Kafka
## Monitor
## Consumidor
## Controlador

# Experimentação
## Metricas
## Geração de dados
## Resultados Algoritmos
## Resultados Aplicação Kafka


# Conclusão
