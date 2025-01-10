---
title: "Write Up do Desafio Try me Out"
date: 2024-01-09
published: 2024-01-09
description: "Solução de um dos desafios incluidos no CTF da H2HC de 2023."
tags:
  - "Algebra Linear"
  - "C"
  - "Engenharia Reversa"
  - "CTF"
categories:
  - "Hard"
images: []
author: martinamarques
---
Este foi um dos desafios inclusos no CTF da H2HC 2023. Para resolver esse desafio de forma mais elegante, é necessário observar os procedimentos matemáticos realizados e perceber suas relações com um algoritmo de decomposição de matrizes bem conhecido.  

### Entendendo o código

A principal função deste programa é a função verify, que recebe um array de 24 caracteres. Isso dá o tamanho da flag. As variáveis mat1 e mat2 são arrays de arrays, o que é uma notação conhecida para matrizes. Além disso, mat2 é a transposta de mat1. A partir dessas variáveis, são encontradas outras duas, AAT e ATA, também matrizes, e surgem ma multiplicação de mat1 por mat2 e de mat2 por mat1, respectivamente. A função MatMul faz excluisvamente a multiplicação entre matrizes.   

![Figura 1](/images/verify.jpg)

![Figura 2](/images/matmul.jpg)

A parte principal da função verify e o laço de repetição while. É ele que retira as informações do input para realizar a verificação. Ele depende da variável contador2, que é incialmente definida como 8.   

Logo no início do laço, são selecionados dois números da entrada e que são colocados no array temp. Esse número é convertido para float e armazenado na variável singval (o nome dessa variável é sugestivo). A variável eigval é obtida elevando o valor armazenado em singval divido por 10 ao quadrado. Em seguida, são encontradas condicionais, e em cada uma são definidos dois vetores de valores inteiros. No geral, afirma-se que o vetor eigvet2 é definido por elementos em posições predefinidas e que o vetor eigvet1 depende do valor da variável contador2. Em alguns casos, os números são multiplicados por -1. Isso é bastante relevante e é uma dica importante.

![Figura 3](/images/while.jpg)

Após sair da condicional, os valores nos arrays eigvet1 e eigvet2 são divididos por 10 e a partir disso são definidos outros dois vetores, result11 e result12, o quais possuem os valores armazenados em eigvet1 e em eigvet2 multiplicados pelo valor float armazenado em eigval. Além disso, são definidos os arrays result21 e result22, que são definidos como o resultado da multiplicação da matriz AAT pelo array eigvet1 e ATA pelo array eigvet2, respectivamente. Após isso, os valores em result11 são comparados com aqueles em result21, bem como os valores em result12 são comparados com aqueles em result22. Isso é feito em um laço de repetição for, que compara os elementos da mesma posição de cada array. Se a diferença entre os valores absolutos de cada elemento for menor ou igual a 1, é acrescido um ponto.   

Após sair deste laço for, a variável contador2 é acresida em 8. Ao fim do laço while, verifica-se a quantidade de pontos. Esta deve ser obrigatóriamente igual a 18 para a resposta correta.  

![Figura 3](/images/poscond.jpg)

Vendo que a comparação é feita entre vetores que surgem a partir de uma multiplicação de vetor por escalar e de multiplicação matriz por vetor, fica evidente que trata-se, essencialmente, de um problema de autovalores e autovetores. Os autovalores surgem ao elevar um outro valor númerico ao quadrado, e as matrizes são resultados de uma matriz multiplicada por sua transposta e da transposta de uma matriz multiplicada pela matriz original... Tudo isso lembra a Decomposição SVD.   

### A teoria por trás

A Decompoisção em Valores Singulares (Singular Value Decomposition) é um algoritmo de decomposição existente para qualquer matriz. Em um espaço complexo, a Decomposição SVD diz que qualquer matriz M pode ser decomposta em: ![Figura 4](/images/dec.png) 

em que U e V* são matrizes unitárias. A matriz V* é a conjugada transposta da matriz V. Uma matriz unitária é aquela em que sua conjugada complexa coincide com sua inversa única. Como a matriz em questão é real, a matriz V* é simplesmente a matriz V transposta (V^T).

Existem diversas formas de interpretar a decomposição SVD, mas ela pode ser interpretada como uma generalização da diagonalização de matrizes, procedimento que só pode ser feito para matrizes quadradas. Nesse sentido, a matriz Sigma deve ser uma matriz diagonal.  

Se multiplicarmos a equação da imagem acima pela conjugada transposta pela esquerda, encontramos: 
![Figura 5](/images/pelaesquerda.jpg)
Como U é unitária, ao multiplicá-la pela sua transposta conjugada, encontra-se a matriz identidade, e pela matriz Sigma ser diagonal, Sigma multiplicada pela sua transposta resulta em uma matriz diagonal cujos elementos são os elementos da matriz Sigma ao quadrado.  

Analogamente, se multiplicarmos a equação da imagem pela conjugada transposta pela direita, encontramos: ![Figura 6](/images/peladireita.jpg). 
Como V é unitária, ao multiplicá-la pela sua transposta conjugada, encontra-se a matriz identidade, e pela matriz Sigma ser diagonal, Sigma multiplicada pela sua transposta resulta em uma matriz diagonal cujos elementos são os elementos da matriz Sigma ao quadrado.

Como a matriz em questão é real, essa demonstração é feita simplesmente com a matriz transposta ao invés da conjugada transposta. 

Sabe-se que, uma matriz multiplicada, independente da ordem, por sua transposta sempre resultará em uma matriz simétrica. Pelo teorema espectral para uma matriz em um espaço real com produto interno de dimensão finita, se uma matriz for simétrica, existe uma base ortonormal de autovetores de tal matriz, sendo que essa obrigatoriamente possui autovalores reais.  

Assim, aplicando esse teorema nas equações encontradas, percebe-se que as matriz U e V são compostas pelos autovetores das matrizes M^TM e MM^T, respectivamente. Além disso, nota-se que os autovalores dessas matrizes são iguais, e são equivalem aos elementos da matriz Sigma^2, o que garante que sejam reais. Assim, conclui-se que a matriz Sigma é uma matriz diagonal composta pelas raízes quadradas dos autovalores das matrizes M^TM e MM^T e a esses valores é dado o nome de valor singular. 

A decomposição SVD da matriz em questão é aproximadamente:
![Figura 7](/images/decmatriz.jpg)

As demais casas decimais foram excluídas por um motivo escrito mais à frente. 

Após computar a decomposição SVD da matriz em questão, deve-se observar novamente o algoritmo para saber qual deve ser a ordem dos elementos da flag.  

Nota-se que a cada ciclo do laço de repeitção while a variável contador2 é acrescida em 8. A função desta variável é selecionar a coordenada do elemento da entrada que é o primeiro algarismo do número de dois dígitos que será usado para encontrar o autovalor. Este número de dois algarismos é dividido por 10 para formar o autovalor. A variável contador2 só pode assumir os valores 0, 8 e 16. Nesse sentido, sabe-se que essas serão as posições no array para o primeiro algarismo de cada valor singular multiplicado por 10.   

Em seguida, observa-se as condicionais: o array eigvet1 é sempre formado pelos números seguintes ao valor singular multiplicado por 10. Este array representa os autovetores da matriz AAT. logo, compõem a coluna da matriz U. Antes de ocorrer a multiplicação pela matriz ou pelo autovalor, lembre-se de que este vetor também tem seus elementos divididos por 10. Logo, percebe-se que os 3 números seguidos a cada valor singular multiplicado por 10 são os elementos do respectivo autovetor da matriz AAT, também multiplicados por 10.  

Com relação ao vetor eigvet2, se observarmos com cuidado, é possível ver que, nas condicionais, os primeiros elementos de cada autovetor estão em sequência. Isso significa que os 3 elementos após o último elemento do autovetor de AAT representam a coluna de V^T de mesma posição da coluna do valor singular em análise. Seus valores também são multiplicados por 10.

![Figura 8](/images/ultimacond.jpg)

Todos os valores são multiplicados por 10 pois, como o teorema espectral exige, os autovetores são ortonormais, ou seja, tem elementos de valor absoluto menor que 1. Escrever um valor decimal em uma flag é inviável. Como a função verify informa, a flag deve ter 24 caracteres, o que leva a entender que após a multiplicação por 10, deve ser ignorada a parte decimal restante. 

Por fim, a condição do primeiro if da função verify informa a ordem dos números com base nos valores singulares:
![Figura 9](/images/if.jpg)


Assim, controi-se a flag: 608522291758029140119913

### Referências
https://towardsdatascience.com/simple-svd-algorithms-13291ad2eef2

https://en.wikipedia.org/wiki/Singular_value_decomposition

LIMA, Elon Lages. Álgebra Linear. 8. ed. [S. l.]: Instituto Nacional de Matemática Pura e Aplicada, 1995. ISBN 978-85-244-0089-0.
