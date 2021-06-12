The document root /var/www/html of all web apps is on NFS share /data on storage server in Stratos Datacenter. We have a requirement where we want to password protect a directory in the Apache web server document root. We want to password protect http://<website-url>:<apache_port>/protected URL as per the following requirements (you can use any website-url for it like localhost since there are no such specific requirements as of now):

a. We want to use basic authentication.

b. We do not want to use htpasswd file base authentication. Instead, we want to use PAM authentication, i.e Basic Auth + PAM so that we can authenticate with a Linux user.

c. We already have a user john with password dCV3szSGNA which you need to provide access to.

d. You can access the website on LBR link. To do so click on the + button on top of your terminal, select Select port to view on Host 1, and after adding port 80 click on Display Port.



```
yum install -y mod_authnz_external pwauth

vi /etc/httpd/conf.d/authnz_external.conf

# Adicionar estas linhas no arquivo
<Directory /var/www/html/protected>
  AuthType Basic
  AuthName "Pam Authentication"
  AuthBasicProvider external
  AuthExternal pwauth
  require valid-user
</Directory>


# Check config httpd
httpd -t


# Restart httpd
systemctl restart httpd
```

