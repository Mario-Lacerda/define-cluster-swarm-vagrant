
Desafio Dio - Definição de um Cluster Swarm Local com o Vagrant


Aqui está um exemplo de como podemos configurar o nosso Vagrantfile para criar um cluster Docker Swarm com uma máquina master e três nodes workers:

# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# Definição dos IPs fixos para cada VM
IPs = {
  "master" => "192.168.56.100",
  "node01" => "192.168.56.101",
  "node02" => "192.168.56.102",
  "node03" => "192.168.56.103"
}

# Configuração das VMs
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Itera sobre cada par chave-valor no hash IPs
  IPs.each do |name, ip|
    config.vm.define name do |node|
      node.vm.box = "ubuntu/bionic64"
      node.vm.hostname = name
      node.vm.network "private_network", ip: ip

      # Instalação do Docker
      node.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        sudo apt-get update
        sudo apt-get install -y docker-ce
      SHELL

      # Configuração específica para o master
      if name == "master"
        node.vm.provision "shell", inline: <<-SHELL
          sudo docker swarm init --advertise-addr #{ip}
        SHELL
      else
        # Configuração específica para os workers
        node.vm.provision "shell", privileged: false, run: "always", inline: <<-SHELL
          TOKEN=$(docker -H 192.168.56.100:2377 swarm join-token worker -q)
          docker swarm join --token $TOKEN 192.168.56.100:2377
        SHELL
      end
    end
  end
end
Este script Vagrantfile define uma máquina master e três nodes workers, todos com IPs fixos e Docker pré-instalado. A máquina master é configurada como o nó manager do cluster, e as outras máquinas são adicionadas ao cluster como workers.

Lembre-se de substituir os IPs e configurações conforme necessário para o seu ambiente específico. Depois de configurar o Vagrantfile, você pode executar vagrant up para iniciar as máquinas virtuais e configurar o cluster.

Uma aplicação clássica para rodar em um cluster Docker Swarm é o WordPress com um serviço de banco de dados MySQL. Aqui está um exemplo de como podemos configurar um docker-compose.yml para implantar o WordPress e o MySQL no seu cluster Swarm:

version: '3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: senha_root
      MYSQL_DATABASE: wordpress
      MYSQL_USER: usuario
      MYSQL_PASSWORD: senha_usuario
    networks:
      - webnet

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: usuario
      WORDPRESS_DB_PASSWORD: senha_usuario
      WORDPRESS_DB_NAME: wordpress
    networks:
      - webnet

volumes:
  db_data:

networks:
  webnet:
Este arquivo define dois serviços: db para o MySQL e wordpress para o aplicativo WordPress. Eles são conectados através de uma rede chamada webnet, e o serviço WordPress será acessível na porta 8000 do host.

Para implantar essa stack no seu cluster Swarm, salve o conteúdo acima em um arquivo chamado docker-compose.yml e execute o seguinte comando no nó manager do seu cluster:

docker stack deploy --compose-file=docker-compose.yml wp_stack
Isso irá criar uma stack chamada wp_stack com os serviços WordPress e MySQL. Lembre-se de alterar as senhas e configurações conforme necessário para garantir a segurança.

Além do WordPress, existem muitos outros exemplos de aplicações que podemos rodar em um cluster Docker Swarm, como sistemas de monitoramento, aplicações de análise de dados, e muito mais. A escolha depende das necessidades específicas do seu projeto ou aprendizado.



https://academiapme-my.sharepoint.com/:p:/g/personal/kawan_dio_me/EZ0N-dJP3WpOrPJv3FZHISkBtgBXM9fA4kVwI-kyJshk0Q?e=E5GBqv

https://github.com/denilsonbonatti/docker-projeto2-cluster



