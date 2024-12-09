# Integrando Flask com Apache e HTTPS

Este guia fornece instruções para rodar uma aplicação Flask no Apache com suporte a HTTPS, utilizando um ambiente virtual.

## Passo 1: Instalar Dependências

1. **Instalar o Apache e mod_wsgi**:
   ```bash
   sudo apt update
   sudo apt install apache2 libapache2-mod-wsgi-py3
2. **Criar e ativar o ambiente virtual**:
    ```bash 
    virtualenv env
    source env/bin/activate
3. **Instalar o flask**:
    ```bash
    pip install flask
## Passo 2:Criar a Aplicação Flask
Crie um arquivo Python para sua aplicação Flask, por exemplo, app.py:
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == "__main__":
    app.run()
```
## Passo 3:Configurar o mod_wsgi
**Criar um arquivo WSGI**: Crie um arquivo chamado flask_project.wsgi no mesmo diretório que seu app.py com o seguinte conteúdo:
```python
import sys
import logging

# Caminho para o diretório da aplicação
sys.path.insert(0, '/home/eved/flask_project')

# Ativar logging
logging.basicConfig(stream=sys.stderr)
sys.stderr = open('/var/log/myapp.log', 'a')

# Ativar o ambiente virtual
activate_this = '/caminho/para/seu/projeto/venv/bin/activate_this.py'
exec(open(activate_this).read(), dict(__file__=activate_this))

from app import app as application
```
## Passo 4: Configurar o Apache para HTTPS
1. **Obter um Certificado SSL**: Você pode usar um certificado SSL gratuito do Let's Encrypt ou um certificado autoassinado para testes. Para um certificado autoassinado, você pode gerar um com o seguinte comando:
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout </etc/apache2/ssl/arquivo.key> -out </etc/apache2/ssl/arquivo.crt>
```
2. **Criar um arquivo de configuração para o Apache**: Por exemplo, /etc/apache2/sites-available/flask_project.conf:
```apache
<VirtualHost *:80>
    ServerName flask_project.com
    Redirect permanent / https://flask_project.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName flask_project.com
    WSGIScriptAlias / /home/eved/flask_project/flask_project.wsgi

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/arquivo.crt
    SSLCertificateKeyFile /etc/apache2/ssl/arquivo.key

    <Directory /home/eved/flask_project/>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/myapp_error.log
    CustomLog ${APACHE_LOG_DIR}/myapp_access.log combined
</VirtualHost>
```
3. **Habilitar os módulos necessários e o site**:
```bash
sudo a2enmod ssl
sudo a2ensite flask_project.conf
sudo systemctl restart apache2
```
## Passo 5: Testar a Aplicação

Abra seu navegador e acesse https://flask_project.com. Você deve ver a mensagem "Hello, World!" e a conexão deve ser segura.