# Servidor Nginx

Documentação: https://nginx.org/en/docs/

# Instalação:


## Distribuições baseadas em Red Hat

```bash
yum install epel-release
yum install nginx
systemctl enable nginx --now
```

## Distribuições baseadas em Debian

```bash
apt install nginx
```

## Comandos

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


## Arquivo de configuração

```
/etc/nginx/nginx.conf
```