# -*- mode: ruby -*-
# vi: set ft=ruby :

# Define a versão da API do Vagrantfile. Essencial para compatibilidade.
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Define a imagem base do Ubuntu que será usada para todas as VMs.
  # O Vagrant irá baixá-la automaticamente na primeira vez.
  config.vm.box = "ubuntu/jammy64" # Ubuntu 22.04 LTS

  # --- 1. CONFIGURAÇÃO DA VM-PROXY ---
  config.vm.define "proxy" do |proxy|
    
    # Define o hostname da máquina
    proxy.vm.hostname = "proxy"

    # Placa de Rede 1: Host-Only (para acesso do seu PC)
    # Daremos um IP estático para facilitar o acesso.
    proxy.vm.network "private_network", ip: "192.168.56.10"

    # Placa de Rede 2: Rede Interna (para falar com app e bd)
    # O `virtualbox__intnet` cria uma rede interna chamada "rede-app"
    proxy.vm.network "private_network", ip: "192.168.10.101", virtualbox__intnet: "redeapp"

    # Provisionamento: Comandos que rodam automaticamente para instalar o Nginx.
    proxy.vm.provision "shell", inline: <<-SHELL
      echo "--> Provisionando a VM-PROXY"
      apt-get update
      apt-get install -y nginx
    SHELL
  end

  # --- 2. CONFIGURAÇÃO DA VM-APP ---
  config.vm.define "app" do |app|

    app.vm.hostname = "app"

    # Placa de Rede 1: Apenas Rede Interna, garantindo o isolamento.
    app.vm.network "private_network", ip: "192.168.10.102", virtualbox__intnet: "redeapp"

    # Provisionamento: Instala o Node.js (usando o repositório oficial para uma versão recente)
    app.vm.provision "shell", inline: <<-SHELL
      echo "--> Provisionando a VM-APP"
      apt-get update
      apt-get install -y ca-certificates curl gnupg
      mkdir -p /etc/apt/keyrings
      curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
      NODE_MAJOR=20
      echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
      apt-get update
      apt-get install -y nodejs
      # Instala o PM2 globalmente
      npm install -g pm2
    SHELL
  end

  # --- 3. CONFIGURAÇÃO DA VM-BD ---
  config.vm.define "db" do |db|
    
    db.vm.hostname = "bd"

    # Placa de Rede 1: Apenas Rede Interna, também isolada.
    db.vm.network "private_network", ip: "192.168.10.103", virtualbox__intnet: "redeapp"

    # Provisionamento: Instala e configura o PostgreSQL para aceitar conexões da VM-APP
    db.vm.provision "shell", inline: <<-SHELL
      echo "--> Provisionando a VM-BD"
      apt-get update
      apt-get install -y postgresql postgresql-contrib

      # Configura o PostgreSQL para aceitar conexões de qualquer IP na rede interna
      echo "--> Configurando PostgreSQL"
      sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /etc/postgresql/14/main/postgresql.conf
      
      # Libera o acesso para o usuário postgres na rede interna (para testes iniciais)
      # Em um ambiente real, criaríamos um usuário específico para a aplicação.
      echo "host    all             all             192.168.10.0/24         md5" >> /etc/postgresql/14/main/pg_hba.conf

      # Reinicia o PostgreSQL para aplicar as configurações
      systemctl restart postgresql
    SHELL
  end

  # Configurações do Provedor VirtualBox (opcional, mas recomendado)
  # Define a memória e a quantidade de CPUs para cada VM.
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024 # 1GB de RAM
    v.cpus = 1
  end
end