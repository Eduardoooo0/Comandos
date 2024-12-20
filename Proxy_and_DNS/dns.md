# Conceito de DNS
  O DNS é um sistema que converte nomes de domínio legíveis por humanos em endereços IP legíveis por máquinas. Este processo permite que os usuários acessem websites utilizando nomes fáceis de lembrar, em vez de precisar memorizar longos números. O DNS funciona de maneira semelhante a uma agenda telefônica, mapeando os nomes de domínio para endereços IP, e permite que os usuários façam consultas que resultam na localização de servidores e recursos na internet.
 1. **Estrutura e Tipos de Servidores DNS**:
  A estrutura do DNS é composta por diferentes tipos de servidores, cada um com funções específicas, sendo os principais:
    * **Servidores DNS Autoritativos**:
  Os servidores DNS autoritativos têm a autoridade final sobre um domínio específico e são responsáveis por fornecer as respostas para as consultas DNS relacionadas a esse domínio.
   Eles armazenam os registros DNS, que mapeiam os nomes de domínio para os endereços IP. O **Amazon Route 53** é um exemplo de serviço DNS autoritativo, fornecendo esse tipo de funcionalidade.

    * **DNS local**:
Local DNS é um sistema de tradução de nomes de domínios em endereços IP. Ele permite que os usuários acessem sites e serviços sem precisar memorizar números complexos.
2. **Funcionamento do DNS Local**:
Quando um usuário digita um nome de domínio no navegador, o computador envia uma solicitação ao servidor DNS local. Esse servidor pode verificar seu cache para ver se já possui o endereço IP associado ao domínio. Caso o endereço IP esteja armazenado no cache, ele é imediatamente retornado ao usuário, o que acelera o processo de navegação. Se o endereço IP não estiver no cache, o servidor DNS local encaminha a solicitação para um servidor DNS autoritativo, que é responsável por fornecer o endereço IP correto para o nome de domínio solicitado. Depois de obter a informação, o servidor DNS local a armazena em seu cache para futuras consultas e retorna o endereço IP ao usuário.
# Aplicação e Configuração do servidor DNS
O dnsmasq é um servidor DNS e DHCP leve, ideal para redes locais.
1. **Instalar o dnsmasq no Servidor**:
 
    ```bash
    sudo apt update
    sudo apt install dnsmasq
    ```
 2. **Configurar o dnsmasq**:
    *  Após a instalação, é necessário configurar o dnsmasq para resolver nomes locais para os IPs dos servidores Flask.**Edite o arquivo de configuração do dnsmasq**:
    ```bash
    sudo nano /etc/dnsmasq.conf
    ```
    * **Adicione as seguintes configurações**:
    ```bash
    domain=laboratorio.local

    address=/flaskapp.laboratorio.local/192.168.1.50  # IP do servidor Flask

    cache-size=1000

    server=8.8.8.8  # Usando o Google DNS como exemplo

    address=/flaskapp.laboratorio.local/192.168.1.50: Resolve o nome flaskapp.laboratorio.local para o endereço IP 192.168.1.50 (substitua pelo IP do seu servidor Flask).
    server=8.8.8.8: Configura o servidor DNS upstream (Google DNS, por exemplo) para resolver domínios externos.
    ```
3. **Reiniciar o dnsmasq**:
    ```bash
    sudo systemctl restart dnsmasq
    ```
4. **Configurar as Estações de Trabalho para Usar o DNS Local**:
    * As estações de trabalho ou dispositivos na rede precisam ser configurados para usar o servidor DNS local. Isso pode ser feito manualmente ou por DHCP.
        * **Manualmente**:
            Em cada dispositivo, configure o servidor DNS para apontar para o IP do servidor onde o dnsmasq está rodando (por exemplo, 192.168.1.10).
            Via DHCP (se o dnsmasq também for o servidor DHCP):
            Se você configurar o dnsmasq para também fornecer endereços IP via DHCP, ele pode automaticamente fornecer o servidor DNS para os dispositivos na rede. No arquivo de configuração, adicione:
            ```bash
            dhcp-range=192.168.1.20,192.168.1.50,12h  # Definindo o intervalo de endereços IP para DHCP
            dhcp-option=6,192.168.1.10  # IP do servidor DNS local
            ```




        * **Reinicie o dnsmasq novamente para aplicar as configurações do DHCP**:
            ```bash
            sudo systemctl restart dnsmasq
            ```
5. **Verificar a Resolução de Nomes**:
    * **Agora, em um dispositivo cliente na rede, tente resolver o nome de domínio**:
        ```bash
        nslookup flaskapp.laboratorio.local
        ```
        * O comando deve retornar o IP 192.168.1.50 (ou o IP do servidor Flask).
6. **Configuração do Flask**:
    * No servidor Flask, você deve garantir que o aplicativo Flask esteja acessível na rede local. Modifique o código de execução do Flask para permitir acessos externos, não apenas do localhost:
        ```bash
        from flask import Flask

        app = Flask(_name_)

        @app.route('/')
        def hello_world():
            return 'Hello, World!'

        if _name_ == "_main_":
            # Tornar o Flask acessível para todos os dispositivos da rede local
            app.run(host='0.0.0.0', port=5000)

        ```
        * Isso fará com que o Flask aceite conexões de qualquer IP na rede local, e o servidor Flask ficará acessível em http://flaskapp.laboratorio.local:5000.
# Referências
* https://aws.amazon.com/pt/route53/what-is-dns/
* https://napoleon.com.br/glossario/o-que-e-local-dns/
* https://wiki.debian.org/dnsmasq
* https://thekelleys.org.uk/dnsmasq/doc.html




