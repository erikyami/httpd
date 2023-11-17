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

