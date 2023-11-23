# Servidor Nginx


Documentação: https://nginx.org/en/docs/

## Instalação:


### Distribuições baseadas em Red Hat

```bash
yum install epel-release
yum install nginx
systemctl enable nginx --now
```

### Distribuições baseadas em Debian

```bash
apt install nginx
```

### Verificando Instalação

```bash
nginx -v
nginx version: nginx/1.22.1
```

### Comandos

Iniciar o nginx

```bash
systemctl start nginx
```

Verificar o status do processo:

```bash
systemctl status nginx
```

Parar o processo do nginx

```bash
systemctl stop nginx
```

Reiniciar o nginx:

```bash
systemctl restart nginx
```

Configurar para subir juntamente com o sistema:

```bash
systemctl enable nginx
```

Desabilitar a subida do nginx juntamente com o sistema:

```bash
systemctl disable nginx
```


## Arquivos, Diretórios e Comandos

É necessário entender os diretórios e comandos do NGINX

### Arquivos e Diretórios

`/etc/nginx`

É o diretório padrão raiz do servidor NGINX. Dentro deste diretório estão os arquivos de configuração que ditam como o servidor deve se comportar.


`/etc/nginx/nginx.conf`

É o arquivo padrão de configuração, usado pelo service do NGINX. É responsável pela configuração global de vários parâmetros, como worker processes, tuning, logging, carregar módulos dinamicamente, referenciar outros arquivos de configuração do NGINX.


`/etc/nginx/conf.d`

Contém o arquivo de configuração do servidor padrão HTTP. Arquivos neste diretório terminam com a extensão `.conf` e são incluídos no bloco `http` do arquivo `/etc/nginx/nginx.conf` através do `include`. É uma boa prática utilizar includes e organizar os arquivos de configuração desta forma para manter uma padronização destes arquivos. Em alguns pacotes de repositórios, este diretório é chamado de `sites-enabled` e arquivos de configuração são um link simbólico chamado de `sites-available`, esta convenção está em desuso (deprecated).

`/var/log/nginx`

É o diretório padrão de logs para o NGINX. Dentro deste diretório é encontrado o `access.log` e o `error.log`. O `access.log` contém as entradas para cada requisição dos servidores NGINX. O `error.log` contém informação de eventos e debug se o módulo `debug` estiber habilitado.


### Comandos

```bash 
nginx -h
```

Mostra o menu de ajuda do NGINX.

```bash 
nginx -v
```

Mostra a versão do NGINX

```bash 
nginx -t
```

Valida a configuração do NGINX


```bash
nginx -s signal
```

A flag `-s` envia um sinal para o processo master do NGINX. É possível enviar sinais como `stop`, `quit`, `reload`, e `reopen`.


Uma boa prática após realizar alteração nas configurações é testar as alterações com o comando: `nginx -t` e em caso de sucesso, fazer o reload do NGINX com o comando: `nginx -s reload`

## Servindo Arquivos Estáticos

Criar um servidor NGINX, criar o arquivo `/etc/nginx/conf.d/default.conf`

```conf
server {
    listen 8080 default_server;
    server_name www.example.com;

    location / {
        root /usr/share/nginx/html;
        # alias /usr/share/nginx/html;
        index index.html index.htm;
   }
}
```

### Reiniciar o NGINX

Verificar se há erros de sintaxe:

```bash
nginx  -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Reload do NGINX:

```bash
nginx -s reload
```

```bash
curl -I localhost:8080

HTTP/1.1 200 OK
Server: nginx/1.22.1
Date: Fri, 17 Nov 2023 22:43:50 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 19 Oct 2022 08:02:20 GMT
Connection: keep-alive
ETag: "634faf0c-267"
Accept-Ranges: bytes
```

## Load Balancer | Balanceador de Carga

### Conceitos

A experiência do usuário de internet nos dias atuais demanda performance e disponibilidade. Para alcançar este objetivo, múltiplas cópias do mesmo sistema estão em execução e distribuem a carga entre si. Esta arquitetura é chamada de *"horizontal scaling"*.

O NGINX é capaz de fazer o balanceamento de carga em várias formas como HTTP, TCP e UDP.

Quando balancear a carga é importante que o impacto na experiência dos usuários seja positiva. Mitas aplicações web modernas possuem camadas *stateless* (sem estado), armazenando o estado em memória compartilhada ou banco de dados. Porém nem todas as aplicações trabalham desta forma.

O estado da Sessão é possui muito valor e muito utilizado em aplicações interativas. Este estado pode ser armazenado localmente no servidor de aplicações por inúmeras razões; por exemplo, em aplicações em que cada dado/workload esteja sendo trabalhado é muito grande e a complexidade (overhead) da Rede impacte na performance.

Quando o estado é armazenado localmente num servidor de aplicação, é extremamente importante que a experiência do usuário e suas próximas requisições continuem sendo entregues para o mesmo servidor.

Trabalhar com aplicações *stateful* em escala requer um balanceamento de carga inteligente.


### HTTP Load Balancing

**Problema**:
Você precisa distribuir a carga entre dois ou mais servidores HTTP.

**Solução**:
Usar o módulo HTTP do NGINX para fazer o balanceamento de carga entre servidores HTTP usando o bloco *upstream*:

`default.conf`

```conf
server {
    listen 8081 default_server;
    server_name server1;

    location / {
        root /usr/share/nginx/server1;
        # alias /usr/share/nginx/html;
        index index.html index.htm;
   }
}

server {
    listen 8082 default_server;
    server_name server2;

    location / {
        root /usr/share/nginx/server2;
        # alias /usr/share/nginx/html;
        index index.html index.htm;
   }
}

server {
    listen 8083 default_server;
    server_name backup;

    location / {
        root /usr/share/nginx/backup;
        # alias /usr/share/nginx/html;
        index index.html index.htm;
   }
}

```

`lb-nginx.conf`

```conf
upstream backend {
   server localhost:8081 weight=1;
   server localhost:8082 weight=1;
   server localhost:8083 backup;
}

server {
   listen 8084;
   location / {
       proxy_pass http://backend;
   }
}

```

#### Logs - real_ip

Quando utilizamos o NGINX como balanceador de carga, o IP enviado para o log dos endpoints é do próprio balanceador e não do cliente que está acessando. É possível enviar o IP do cliente realizando as seguintes configurações:

1 - Configuração do `log_format` na seção `http` do arquivo `/etc/nginx/nginx.conf`
Adicionar a configuração:

```conf
log_format combined_real_ip '$http_x_real_ip - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent"';
```

2 - Configurar Load Balancer no arquivo que faz o balanceamento para adicionar o header `X-Real-IP`:

```conf
location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
}
```

3 - Configurar o Log no endpoint adicionando `combined_real_ip` que foi configurado no arquivo `nginx.conf`

```conf

access_log /var/log/nginx/server1-access.log combined_real_ip;
error_log /var/log/nginx/server1-error.log;
```

4 - Verificar a sintaxe e recarregar o NGINX

```bash
nginx -t
nginx -s reload
```
