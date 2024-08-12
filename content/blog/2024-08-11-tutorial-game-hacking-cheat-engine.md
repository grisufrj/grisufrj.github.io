---
title: Tutorial de Game Hacking utilizando Cheat Engine 
date: 2024-08-12
description: "Aprender como utilizar o Cheat Engine para encontrar e manipular endereços de memória de interesse, injeção/remoção de código para mudar o funcionamento do jogo"
tags: [Game Hacking, Engenharia Reversa]
categories: [intro]
author: asw4ng
published: false
---

Abordaremos neste artigo sobre os princípios fundamentais sobre Game Hacking, com a utilização do Cheat Engine. (Explicar de forma sucinta o que é Cheat Engine). (Falar sobre as práticas e exemplos deste POST).

## Solucionando o Tutorial Padrão (Tutorial-x86_64.exe)

	(CONTEÚDO!!!)

## Explorando o Tutorial GUI (gtutorial-x86_64.exe)

	(CONTEÚDO!!!)

## Prática com Jogo real

	(CONTEÚDO!!!)

## Jogos de Browser

	(CONTEÚDO!!!)

## Fechamento

	(CONTEÚDO!!!)

## Referências
	
	(Esquece n, pls)

##### Indice

1. Apresentar os conceitos de game Hacking e o Cheat Engine, além do que será feito neste POST
2. Tutorial do Cheat Engine (Tutorial-x86_64.exe)
	1. Anexando um processo ao Cheat Engine
	2. Escaneamento por Valor Exato
	3. Valor Inicial desconhecido
	4. Floats/Doubles
	5. Code finder (Substituir uma instrução Assembly por NOP)
	6. Ponteiros
	7. Injeção de Código
	8. Ponteiros para ponteiros para Ponteiros para...
	9. Código compartilhado
3. GUI tutorial do Cheat Engine (gtutorial-x86_64.exe)
	1. Destruir Alvo que se cura
	2. Abater naves inimigas mais fortes
	3. Plataformer 
4. Prática em um Jogo real (Assault Cube ou Holocure)
5. Jogos single player de web (Achar o núm do processo)
6. Conclusão
7. Referências


> DELETAR ABAIXO AO PUBLICAR!!!

Below is just about everything you'll need to style in the theme. Check the source code to see the many embedded elements within paragraphs.

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

##### Heading 5

###### Heading 6

### Body text

Lorem ipsum dolor sit amet, test link adipiscing elit. **This is strong**. Nullam dignissim convallis est. Quisque aliquam.

![Smithsonian Image](/images/3953273590_704e3899d5_m.jpg)


*This is emphasized*. Donec faucibus. Nunc iaculis suscipit dui. 53 = 125. Water is H<sub>2</sub>O. Nam sit amet sem. Aliquam libero nisi, imperdiet at, tincidunt nec, gravida vehicula, nisl. The New York Times <cite>(That’s a citation)</cite>. <u>Underline</u>. Maecenas ornare tortor. Donec sed tellus eget sapien fringilla nonummy. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus.

HTML and <abbr title="cascading stylesheets">CSS<abbr> are our tools. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus. Praesent mattis, massa quis luctus fermentum, turpis mi volutpat justo, eu volutpat enim diam eget metus.

### Blockquotes

> Lorem ipsum dolor sit amet, test link adipiscing elit. Nullam dignissim convallis est. Quisque aliquam.

## List Types

### Ordered Lists

1. Item one
   1. sub item one
   2. sub item two
   3. sub item three
2. Item two

### Unordered Lists

* Item one
* Item two
* Item three

## Tables

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|----
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=====
| Foot1   | Foot2   | Foot3
{: rules="groups"}

## Code Snippets

Syntax highlighting via Pygments

{% highlight css %}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
{% endhighlight %}

Non Pygments code example

    <div id="awesome">
        <p>This is great isn't it?</p>
    </div>

## Buttons

Make any link standout more when applying the `.btn` class.

{% highlight html %}
<a href="#" class="btn btn-success">Success Button</a>
{% endhighlight %}

<div markdown="0"><a href="#" class="btn">Primary Button</a></div>
<div markdown="0"><a href="#" class="btn btn-success">Success Button</a></div>
<div markdown="0"><a href="#" class="btn btn-warning">Warning Button</a></div>
<div markdown="0"><a href="#" class="btn btn-danger">Danger Button</a></div>
<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div>
