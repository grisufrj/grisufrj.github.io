---
title: "JSON Web Tokens"
date: 2024-03-14
published: 2024-03-14
tags: [web]
author: jhaysonj
categories:
  - "medium"
tags:
  - "web"
---

# Definindo JSON Web Tokens

JSON Web Token (JWT) é um formato  para transmitir informações entre partes como um objeto JSON. Do ponto de vista de segurança, JWT é uma ferramenta valiosa, pois oferece autenticação e autorização em um formato que pode ser facilmente verificado e confiado.

Para identificar um JWT, pode-se observar a estrutura básica composta por três partes separadas por pontos `.`: o cabeçalho (header), o payload (carga útil), e a assinatura (signature). Essas partes são codificadas em Base64 e são visualmente separadas por pontos.

Exemplo de um JWT:

![Figura 1](/images/2024-03-14-1.png)


- O cabeçalho contém o tipo de token (JWT) e o algoritmo de assinatura usado.
- O payload contém as declarações (claims), que são as informações sobre o usuário ou entidade e metadados adicionais.
- A assinatura é criada a partir do cabeçalho, payload e uma chave secreta, usando um algoritmo especificado no cabeçalho. Essa assinatura é usada para verificar a integridade dos dados e garantir que o token não tenha sido alterado.



## Brute Force da chave secreta
A ideia do brute force para saber como assinar um JWT envolve tentar adivinhar a chave secreta usada para assinar o token. 

Isso é particularmente relevante quando o algoritmo de assinatura usado é **baseado em HMAC (como HMAC-SHA256), que requer uma chave secreta compartilhada entre as partes que emitem e verificam o token**. 

Um atacante pode tentar gerar tokens com diferentes chaves e comparar as assinaturas resultantes até encontrar uma correspondência. Isso é extremamente difícil se a chave for longa e aleatória o suficiente, mas pode ser viável se a chave for fraca ou previsível. Portanto, a escolha de uma chave forte e segura é fundamental para mitigar esse tipo de ataque. Além disso, algoritmos de assinatura assimétrica, como RSA, podem ser preferíveis em alguns casos, pois não exigem uma chave secreta compartilhada e são mais resistentes a ataques de força bruta.

De forma geral:
```
hashcat -a 0 -m 16500 <jwt> <wordlist>
```
Em posse da chave secreta usada para assinar os JWTs, um atacante pode realizar várias ações maliciosas, dependendo do contexto e das permissões concedidas pelos tokens. Algumas possíveis consequências incluem:

- Falsificação de tokens: O atacante pode gerar tokens falsos com informações alteradas ou falsas no payload. Isso pode permitir acesso não autorizado a recursos ou informações privilegiadas.

- Escalada de privilégios: Se os tokens incluírem informações sobre permissões ou níveis de acesso, o atacante pode modificar esses dados para aumentar seus próprios privilégios ou níveis de autorização.

- Ataques de sessão: Com acesso aos JWTs, um atacante pode assumir sessões de usuários legítimos, permitindo acesso contínuo a sistemas ou aplicativos mesmo após o término da sessão legítima.

## Laboratório Portswigger
Agora vamos resolver um dos laboratórios do portswigger para compreendermos melhor como o JWT funciona

Acesse o **[laboratório de JWT](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key)** do portswigger.

Para isso, será necessário utilizar a extensão `JWT editor`, no BurpSuite
![Figura 2](/images/2024-03-14-2.png)

Dentro do laboratório o nosso objetivo é aplicar força bruta à chave secreta do site. Depois de a obter, utilizaremos a chave para assinar um token de sessão modificado que nos dá acesso ao painel de administração em /admin e, em seguida, eliminaremos o usuário carlos.

Vamos capturar o um JWT válido ao relizar login com as seguintes credenciais: wiener:peter

Ao realizarmos o login, podemos ver no burp que capturamos um JWT

![Figura 3](/images/2024-03-14-3.png)

Enviamos a primeira requisição em azul para o repeater. 
No menu `JSON web token` podemos validar que o algoritmo é `"alg": "HS256"` que requer uma chave secreta compartilhada entre as partes que emitem e verificam o token
![Figura 4](/images/2024-03-14-4.png)

```
hashcat -a 0 -m 16500 eyJraWQiOiIyMmE0MWIzNC05MmEzLTRhYzMtYmVhZC1lYWVkMTVjMTM1MmEiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTcxMDQzNDQ4MSwic3ViIjoid2llbmVyIn0.zYGjEa63cPmqIvlKeWqbnKN_old9IDyfzW4VxtX4PhQ ~/wordlists/jwt.secrets.list 
```
Com isso conseguimos a chave secreta `secret1`.

### Falsificação de tokens
Em posse da chave secreta, vamos alterar o nosso JWT das credenciais `wiener:peter` para admin e assinarmos com uma assinatura válida.

Para isso, vamos utilizar o **[JWT io](https://jwt.io/)**

Copiamos o JWT da wiener e alteramos o usuário `wiener` para `administrator`, além disso adicionamos a assinatura com a chave encontrada
![Figura 5](/images/2024-03-14-5.png)

Baixe a extensão `cookie-editor` de navegador para facilitar a criação de uma sessão
![Figura 6](/images/2024-03-14-6.png)

Basta adicionar o JWT no campo `value` do cookie-editor, com isso conseguimos entrar no painel de administrador localizado em `/admin` e, consequentemente, excluirmos o usuário carlos.


## Referências

- **[portswigger JWT](https://portswigger.net/web-security/jwt)**
- **[JWT io](https://portswigger.net/web-security/jwt)**
