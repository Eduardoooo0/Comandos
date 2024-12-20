# Conceito de Proxy Reverso
Um proxy reverso é um servidor que atua como intermediário para requisições de clientes que buscam acessar outros servidores. Em vez de o cliente se conectar diretamente ao servidor de origem, ele se conecta ao proxy reverso, que então redireciona a requisição para o servidor apropriado. O proxy reverso pode oferecer várias funcionalidades, como:
* **Balanceamento de carga**: Distribui requisições entre múltiplos servidores para otimizar a performance.

* **Segurança**: Oculta a identidade e a estrutura da rede interna, protegendo os servidores de ataques diretos.

* **Cache**: Armazena respostas de requisições comuns para acelerar o tempo de resposta.

* **SSL Termination**: Gerencia o tráfego HTTPS, descarregando a criptografia dos servidores de aplicação.

![Sistema do proxy reverso](https://kinsta.com/pt/wp-content/uploads/sites/3/2020/08/funciona-servidor-proxy-reverso.png)
https://kinsta.com/pt/blog/proxy-reverso/

# Aplicação e Configuração do Proxy Reverso
Para implementar um proxy reverso em um servidor web, pode-se usar o Nginx.
1. **Instalação do Nginx**:
```bash
sudo apt update
sudo apt install nginx
```
2. **Criar um arquivo de configuração**:
```bash
sudo nano /etc/nginx/sites-available/nginx.conf
```
3. **Configuração do Nginx como Proxy Reverso**:

código de configuração do nginx.conf:
```bash
server {
    listen      80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;  # Redireciona para o apache
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
    }
    # para redirecionar para outra rota
    location /sua_rota {
        proxy_pass http://127.0.0.1:5000/sua_rota;
    }
    # a rota deve tá criada no arquivo .py
}
```
exemplo de um arquivo apache usando flask:
```bash
<VirtualHost *:5000>  # Mude para a porta que o Apache usará
    ServerName example.com
    ServerAlias www.example.com

    WSGIDaemonProcess flaskapp threads=5
    WSGIScriptAlias / /var/www/example.com/flaskapp.wsgi

    <Directory /var/www/example.com>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/flaskapp_error.log
    CustomLog ${APACHE_LOG_DIR}/flaskapp_access.log combined
</VirtualHost>
```
**A APLICAÇÃO APACHE COM FLASK DEVE ESTÁ TOTALMENTE CORRETA E COMPLETA!**

4. **Salve o arquivo de hospedagem virtual criado**:
Em seguida, ative o novo host virtual criando um link simbólico para os arquivos chamados **nginx.conf** tanto no diretório **/etc/nginx/sites-available** quanto nos diretórios **/etc/nginx/sites-enabled**.
```bash
sudo ln -s /etc/nginx/sites-available/nginx.conf /etc/nginx/sites-enabled/nginx.conf
```
5. **Testar o Nginx**:
```bash
sudo nginx -t
```
6. **Reiniciar o Nginx e o apache**:
```bash
# Reinicie o Apache
sudo systemctl restart apache2

# Reinicie o Nginx
sudo systemctl restart nginx
```
6. **Testar a Configuração**:
Acesse **http://seu_dominio.com** no navegador para verificar se as requisições estão sendo redirecionadas corretamente.

# Referências
**https://kinsta.com/pt/blog/proxy-reverso/**
