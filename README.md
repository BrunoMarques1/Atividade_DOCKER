### Criar VPC
- Acesse o serviço VPC da AWS;
- No canto superior esquerdo, clique em "Criar VPC";
- Selecione "Somente VPC";
- Escolha um nome para a VPC (por exemplo, "VPC-AT");
- Na seção "Bloco CIDR IPv4", selecione "Entrada manual de CIDR IPv4" e, em seguida, insira o CIDR IPv4 desejado (por exemplo, 172.28.0.0/16);
- Na seção "Bloco CIDR IPv6", mantenha selecionada a opção "Nenhum bloco IPv6";
- Mantenha a opção "Padrão" selecionada em "Locação";
- Clique em "Criar VPC".
<br>

### Criar Sub-Redes
- Ainda no serviço VPC, no menu do lado esquerdo, clique em "Sub-redes";
- Clique em "Criar sub-rede" no canto superior direito;
- Em "ID da VPC", selecione a VPC criada anteriormente;
- Configure quatro sub-redes com as seguintes configurações:

  Nome  | Zona de disponibilidade  | Bloco CIDR IPv4
  ------------- | ------------- | ------------- 
  SUB-privada01 | us-east-1a | 172.28.0.0/24
  SUB-privada02 | us-east-1b | 172.28.3.0/24
  SUB-publica01 | us-east-1a | 172.28.1.0/24
  SUB-publica02 | us-east-1b | 172.28.2.0/24

- Após criar todas as sub-redes, selecione uma sub-rede pública, clique em "ações" e depois em "Editar configurações de sub-rede". Marque a opção "Habilitar endereço IPv4 público de atribuição automática" e salve as alterações;
- Repita os mesmos passos anteriores para a outra sub-rede pública.
<br>

### Criar Gateway da internet
- No menu na parte esquerda, clique em "Gateways da Internet";
- No canto superior direito, clique em "Criar gateway da internet";
- Escolha um nome (por exemplo, "IGW-AT") e crie o gateway.;
- Selecione o gateway da internet criado e associe-o à VPC criada anteriormente.
<br>

### Criar Gateway NAT
- No menu na parte esquerda, clicar em "Gateways NAT";
- No canto superior direito, clicar em "Criar gateway NAT";
- Escolha um nome (por exemplo, "NGW-AT") e crie o gateway;
- Selecione uma sub-rede publica;
- Em "Tipo de conectividade", mantenha selecionada a opção "Público";
- Clicar em "Alocar IP elástico";
- Por fim, clicar em "Criar gateway NAT". 
<br>

### Criar tabelas de rotas
- No menu na parte esquerda, clicar em "Tabela de rotas";
- No canto superior direito, clicar em "Criar tabela de rotas";
- Criar duas tabelas de rotas, uma para ser privada e outra pública, ambas usando a VPC criada anteriormente;
- Após ter as duas tabelas crias, associar as sub-redes em suas respectivas tabelas(sub-redes públicas na tabela pública e sub-redes privadas na tabela privada);
- Configurando tabela de rotas pública:
  - Selecionar a tebela de rotas pública e clicar na opção "Rotas" e depois em "Editar rotas";
  - Clicar em "Adicionar rota", na parte de "Destino" selecionar `0.0.0.0/0` e em "Alvo" escolher "Gateway da Internet" e selecionar o gateway criado anteriormente.
- Configurando tabela de rotas privada:
  - Selecionar a tabela de rotas privada e clicar na opção "Rotas" e depois em "Editar rotas";
  - Clicar em "Adicionar rota", na parte de "Destino" selecionar `0.0.0.0/0` e em "Alvo" escolher "Gateway NAT" e selecionar o gateway criado anteriormente.
<br>

### Criar Bastion Host
- Acesse o serviço EC2 da AWS e clique em "Executar instâncias";
- Escolha um nome (por exemplo, "BASTION-HOST");
- Na parte de imagem, escolha `Amazon Linux 2023 AMI`;
- Em tipo de instância, escolha `t2.micro`;
- Em pares de chaves, você pode usar uma chave existente no formato .pem, caso contrário, crie uma nova chave do tipo RSA no formato .pem;
- Em grupos de segurança, crie um com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ------------- | ------------- | ------------- | -------------
  SSH | TCP | 22 | Meu IP

- Em armazenamento, mantenha o padrão de 1x 8 GiB gp3;
- Clique em "Executar instância".
<br>

### Criar Instância Privada (Docker - Wordpress)
- Ainda no serviço EC2 da AWS, clique novamente em "Executar instâncias";
- Escolha um nome (por exemplo, "WORDPRESS");
- Na parte de imagem, escolha `Amazon Linux 2023 AMI`;
- Em tipo de instância, escolha `t2.micro`;
- Em pares de chaves, você pode usar uma chave existente no formato .pem, caso contrário, crie uma nova chave do tipo RSA no formato .pem;
- Crie um novo grupo de segurança com as seguintes regras de entrada (Lembrando que o Load-Balance e o EFS ainda não foram criados, os endereços IP podem ser alterados/adicionados posteriormente):

  Tipo | Protocolo | Intervalo de portas | Origem
  ------------- | ------------- | ------------- | -------------
  SSH | TCP | 22 | Endereço IPv4 privado do `Bastion-Host`
  HTTP | TCP | 80 | Grupo de segurança do `Load-Balance`
  NFS | TCP | 2049 | Endereço IP privado do `EFS`
  
- Na seção de "Detalhes avançados", no último item (Dados do usuário), adicionar o seguinte scrpit:

  ```bash
  #!/bin/bash
  sudo yum update -y
  sudo yum install docker -y
  sudo systemctl start docker
  sudo systemctl enable docker
  sudo usermod -a -G docker ec2-user
  sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  ```
<br>

### Acessando sua instância privada através do bastion host de forma segura
- Para fazer o acesso na instância privada através do bastion host usaremos o `ssh-agent`, para que não seja preciso copiar a chave .pem para dentro da instância;
- Sendo assim, em sua máquina local, use o comando `ssh-add SUA_CHAVE.pem`;
- Se quiser verficiar se a chave foi adicionada, pode usar o comando `ssh-add -l`;
- Agora, para acessar o bastion host use o seguinte comando `ssh -A -i SUA_CHAVE.pem ec2-user@IP_DO_SEU_BASTION_HOST`;
- Por fim, para acessar a instância privada use o seguinte comando `ssh ec2-user@IP_DA_SUA_INSTÂNCIA`.

<br>

### Criar e configurar EFS
- No console AWS procurar pelo serviço EFS;
- Clicar em "Criar sistema de arquivos";
- Escolher um nome e escolher a mesma VPC criada anteriormente, depois clicar em "Criar";
- Voltar para o serviço EC2, na aba de "Grupos de Segurança", criar um grupo para o EFS, com as seguintes regras de entrada:
  
  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  NFS | TCP | 2049 |  CIDR IPv4 da sua VPC

- Já fazendo acesso na instância privada, através do bastion-host, usar os seguintes comandos:
  - `sudo yum -y install nfs-utils`, para instalar o pacote `nfs-utils`;
  - `sudo mkdir /mnt/nfs`, para criar um diretório;
  - `sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 172.28.1.224:/ /mnt/nfs` para montar o sistema de arquivos;
  - `sudo vim /etc/fstab` para editar o arquivo fstab;
  - Adicionar a seguinte linha: `IP ou DNS do EFS criado:/    /mnt/nfs         nfs    defaults          0   0 `.
<br>

### Criar RDS
- Antes de criar o banco de dados, precisamos criar um grupo de segurança para ele. Portanto, vá até o serviço EC2 e entre em "Grupos de segurança";
- Crie um grupo de segurança com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  MYSQL | TCP | 3306 |  CIDR IPv4 da sua VPC

- No console AWS procurar pelo serviço RDS;
- Clicar em "Criar banco de dados";
- Em "Escolher um método de criação de banco de dados", manter "Criação padrão" selecionado;
- Em "Opções do mecanismo", escolher MySQL;
- Em "Modelos", escolher "Nível gratuito";
- Em configurações:
  - Identificador da instância de banco de dados: Escolher um nome, por exemplo `wordpressdb`;
  - Configurações de credenciais: Escolher um nome de usuário principal e sua senha.
- Em configuração da instância, manter a configuração padrão;
- Em armazenamento, manter a configuração padrão;
- Em conectividade:
  - Em "Recurso de computação", manter selecionado "Não se conectar a um recurso de computação do EC2";
  - Em "Nuvem privada virtual (VPC)", selecionar a VPC criada anteriormente;
  - Em "Grupo de sub-redes de banco de dados", manter padrão;
  - Em "Acesso público", selecionar `Sim`;
  - Em "Grupo de segurança de VPC (firewall)", seleionar "Selecionar existente" e escolher o grupo criado anteriormente;
  - O restante das configurações de conectitividade manter o padrão que está;
- Em "Autenticação de banco de dados", manter selecionado "Autenticação de senha";
- Em "Monitoramento", manter desabilitado a opção "Hablitar monitoramento avançado";
- Em "Configuração adicional", apenas adicionar um nome para o banco de dados inicial, por exemplo: `wp_db`
- Clicar em "Criar banco de dados" ([Resolução para um possível erro](https://github.com/BrunoMarques1/Atividade_DOCKER/blob/main/ErroDB.md)).
<br>

### Subir o container Wordpress 
- Acessar a instância privada já configurada com Docker e Docker compose;
- Para criar o container Wordpress, iremos usar o seguinte arquivo docker-compose.yml:
  ```yml
  
  version: '3'
  services:
    wordpress:
      image: wordpress:latest
      ports:
        - "80:80"
      restart: always
      environment:
        WORDPRESS_DB_HOST: <Endpoint do banco de dados criado>
        WORDPRESS_DB_USER: <Usuário criado>
        WORDPRESS_DB_PASSWORD: <Senha criada>
        WORDPRESS_DB_NAME: <Banco de dados inicial criado>
      volumes:
        - /mnt/nfs/wordpress:/var/www/html

  ```
- Dentro do mesmo diretório em que o arquivo citado acima estiver, usar o comando docker-compose up.
<br>

### Criar Load Balancer
- Antes de criar o Load Balancer, precisamos criar um grupo de segurança para ele. Portanto, vá até o serviço EC2 e entre em "Grupos de segurança";
- Crie um grupo de segurança com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  HTTP | TCP | 80 | 0.0.0.0/0

- Além do grupo de segurança, precisamos criar um grupo de destino. Ainda no serviço EC2, entre na aba de "Grupos de destino";
- Crie um grupo de destino com as seguintes configurações:
  - Tipo de destino: Instâncias
  - Nome do grupo de destino: Escolha um nome (por exemplo, TG-AT)
  - Protocolo: HTTP | Porta: 80
  - VPC: Selecione a VPC criada anteriormente
  - Versão do protocolo: HTTP1
  - Protocolo da verificação de integridade: HTTP
- No Serviço EC2, da AWS, selecionar entrar na aba de "Load balancers" e clicar em "Criar load balancer";
- Em "Tipos de load balancer", selecionar `Application Load Balancer`;
- A configuração do load balancer será a seguinte:
  - Nome: Esolha um nome (por exemplo, LB-AT)
  - Esquema: Voltado para a internet
  - Tipo de enereço IP: IPv4
- O mapeamento de rede será:
  - VPC: VPC criada anteriormente
  - Mapeamentos: Selecionar as duas zonas de disponibilidade, com a sub-rede publica
- Em "Grupos de segurança", selecionar o criado anteriomente;
- Em "Listeners e roteamento", configurar da seguinte maneira:
  - Protocolo: HTTP
  - Porta: 80
  - Ação padrão: Selecionar o grupo de destino criado anteriormente
- Clicar em "Criar load balancer".

<br>

### Criar AMI
- Antes de criar e configurar o auto scaling, precisamos criar uma AMI com base na instância privada que criamos anteriormente;
- Ir na aba de instâncias e clicar com o botão direito na instância privada que configuramos anteriormente;
- Selecionar "Imagem e modelos" e clicar em "Criar imagem";
- Escolher um nome (por exemplo, AMI-AT) e uma descrição;
- Manter o resto das configurações que já vem padrão e clicar em "Criar imagem".

<br>

### Criar Modelo de Execução
- Além da AMI, antes de criar o auto scaling precisamos criar um `Modelo de execução`, presente no menu esquerdo do serviço EC2 da AWS;
- Clicar em "Criar modelo de execução";
- Escolher um nome (por exemplo, ME-AT) e uma descrição;
- Em "Imagens de aplicação e de sistema operacional", escolher a imagem criada anteriormente
- Em "Tipo de instância", selecionar `t2.micro`;
- Em "Pares de chaves", você pode selecionar uma chave `.pem` já criada ou criar outra;
- Em "Configurações de sub-rede", configurar da seguinte maneira:
  - Sub-rede: Não incluir no modelo de execução
  - Firewall: Selecionar o grupo de segurança criado anteriormente
- Em "Armazenamento", verificar apenas se a opção "Excluir no encerramento" está como `Sim`;
- Em "Detalhes avançados", adicionar o seguinte script em dados do usuário:

  ```bash
  #!/bin/bash
  sudo yum update -y
  sudo yum install nfs-utils -y
  sudo mkdir -p /mnt/nfs
  sudo echo "fs-05eb47462a8d30dda.efs.us-east-1.amazonaws.com:/    /mnt/nfs         nfs    defaults          0   0 " >> /etc/fstab
  sudo mount -a
  ```
- O scrip serve para montar o volume EFS nas instâncias que o Auto Scaling for subir;
- Clique em "Criar modelo de execução".
