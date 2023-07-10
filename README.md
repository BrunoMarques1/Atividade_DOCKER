<div align="center">
  <h1>Atividade - AWS - Docker</h1>
</div>

# Descrição: 
### Realizar a implementação da seguinte arquitetura AWS no seu ambiente
![image](https://github.com/BrunoMarques1/Atividade_DOCKER/assets/127341401/e412bb89-811c-4f1f-a8f9-b87715a1cc59)

### Requisitos
1. instalação e configuração do DOCKER ou CONTAINERD no host EC2;
  Ponto adicional para o trabalho utilizar a instalação via script de Start Instance (user_data.sh)

2. Efetuar Deploy de uma aplicação Wordpress com: container de aplicação RDS database Mysql

3. configuração da utilização do serviço EFS AWS para estáticos do container de aplicação Wordpress

4. configuração do serviço de Load Balancer AWS para a aplicação Wordpress

### Pontos de atenção:
- Não utilizar ip público para saída do serviços WP (Evitem publicar o serviço WP via IP Público)
- Sugestão para o tráfego de internet sair pelo LB (Load Balancer)
- Pastas públicas e estáticos do wordpress sugestão de utilizar o EFS (Elastic File System)
- Fica a critério de cada integrante (ou dupla) usar Dockerfile ou Dockercompose;
- Necessário demonstrar a aplicação wordpress funcionando (tela de login)
- Aplicação Wordpress precisa estar rodando na porta 80 ou 8080;
- Utilizar repositório git para versionamento;
- Criar documentação.

<br>

# Criar/Configurar: VPC, Sub-redes, Gateway da internet, Gateway NAT e Tabela de Rotas
### Criar VPC
- Acesse o serviço VPC da AWS;
- No canto superior esquerdo, clique em "Criar VPC";
- Configure sua VPC da seguinte maneira:
  - Recursos a serem criados: `Somente VPC`;
  - Nome: Escolha um nome, por exemplo, `VPC-AT`;
  - Bloco CIDR IPv4: `Entrada manual de CIDR IPv4`;
  - CIDR IPv4: Escolha um CIDR IPv4, por exemplo, `172.28.0.0/16`;
  - Bloco CIDR IPv6: `Nenhum bloco CIDR IPv6`;
  - Locação: `Padrão`.
- Clique em "Criar VPC";
- Após criar sua VPC, selecione-a e clique em "Ações".
- Em "Configurações de DNS", marque as duas caixas e clique em "Salvar". Veja a imagem abaixo como referência:
  
  ![image](https://github.com/BrunoMarques1/Atividade_DOCKER/assets/127341401/dc17aaaf-06d2-4429-af5b-84f4696e6e3b)

<br>

### Criar Sub-Redes
- Ainda no serviço VPC, no menu do lado esquerdo, clique em "Sub-redes";
- Configure suas Sub-redes da seguinte forma:
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
- Clique em "Alocar IP elástico";
- Por fim, clique em "Criar gateway NAT". 
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

# Criar/Configurar: EFS, Bastion Host, Instância com Wordpress, Acesso à instância privada e RDS
### Criar EFS
- No console AWS procurar pelo serviço EFS;
- Clicar em "Criar sistema de arquivos";
- Escolha um nome e selecione a mesma VPC criada anteriormente, em seguida, clique em "Criar".
- Volte para o serviço EC2 e, na aba de "Grupos de Segurança", crie um grupo para o EFS com as seguintes regras de entrada:
  
  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  NFS | TCP | 2049 |  CIDR IPv4 da sua VPC

- Volte para o serviço EFS, clique no sistema de arquivos recém-criado e vá para a seção "Rede".
- Mude todos os Security Groups para o mesmo recém criado;

<br>

### Criar Bastion Host
- Acesse o serviço EC2 da AWS e clique em "Executar instâncias";
- Configure uma instância EC2 com as seguintes especificações:
  - Nome e Tags: Escolha um nome (por exemplo, `BASTION-HOST`) e possíveis tags necessárias;
  - AMI: `Amazon Linux 2023 AMI`;
  - Tipo de instância: `t2.micro`;
  - Par de chaves: Crie uma nova chave .pem;
  - VPC: Selecione a VPC criada anteriormente;
  - Sub-rede: Selecione uma das sub-redes `Públicas`, criada anteriormente;
  - Grupos de segurança: Crie um grupo de segurança com as seguintes regras de entrada:
    
      Tipo | Protocolo | Intervalo de portas | Origem
      ------------- | ------------- | ------------- | -------------
      SSH | TCP | 22 | Meu IP
  - Armazenamento: `1x8 | GiB gp3`;
- Clique em "Executar instância". 

<br>

### Criar Instância Privada (Docker - Wordpress)
- Ainda no serviço EC2 da AWS, clique novamente em "Executar instâncias";
- Configure uma instância EC2 com as seguintes especificações:
  - Nome e Tags: Escolha um nome (por exemplo, `WORDPRESS`) e possíveis tags necessárias;
  - AMI: `Amazon Linux 2023 AMI`;
  - Tipo de instância: `t2.micro`;
  - Par de chaves: Crie uma nova chave .pem ou use a mesma criada para o Bastio host;
  - VPC: Selecione a VPC criada anteriormente;
  - Sub-rede: Selecione uma das sub-redes `Privadas`, criada anteriormente;
  - Grupos de segurança: Crie um grupo de segurança com as seguintes regras de entrada (Lembrando que o Load Balancer ainda não foi criado, as informações de origem podem ser alterados/adicionados posteriormente):
    
    Tipo | Protocolo | Intervalo de portas | Origem
    ------------- | ------------- | ------------- | -------------
    SSH | TCP | 22 | Endereço IPv4 privado do `Bastion-Host`
    HTTP | TCP | 80 | Grupo de segurança do `Load-Balance`
    NFS | TCP | 2049 | Endereço IP privado do `EFS`
    
  - Armazenamento: `1x8 | GiB gp3`;
  - Detalhes avançados: Na parte de `Dados do usuário`, adicione o seguinte script:

    ```bash
    #!/bin/bash
    sudo yum update -y
    sudo yum install nfs-utils -y
    sudo mkdir -p /mnt/nfs
    sudo echo "DNS_OU_IP_DO_EFS:/    /mnt/nfs         nfs    defaults          0   0 " >> /etc/fstab
    sudo mount -a
    sudo yum install docker -y
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo usermod -a -G docker ec2-user
    sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```
- O script acima realiza a configuração necessária para montar o sistema de arquivos EFS, instalar e configurar o Docker e instalar o Docker Compose na instância EC2.
- Clique em "Executar instância".  

<br>

### Acessando sua instância privada através do bastion host de forma segura
- Para fazer o acesso na instância privada através do bastion host usaremos o `ssh-agent`, para que não seja preciso copiar a chave .pem para dentro da instância;
- Em sua máquina local, use o comando ssh-add `SUA_CHAVE.pem`;
- Caso queira verficiar se a chave foi adicionada, pode usar o comando `ssh-add -l`;
- Agora, para acessar o bastion host use o seguinte comando `ssh -A -i SUA_CHAVE.pem ec2-user@IP_DO_SEU_BASTION_HOST`;
- Por fim, para acessar a instância privada use o seguinte comando `ssh ec2-user@IP_DA_SUA_INSTÂNCIA`.

<br>

### Criar RDS
- Antes de criar o banco de dados, precisamos criar um grupo de segurança para ele. Portanto, vá até o serviço EC2 e entre em "Grupos de segurança";
- Crie um grupo de segurança com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  MYSQL | TCP | 3306 |  CIDR IPv4 da sua VPC

- No console AWS procurar pelo serviço RDS;
- Clicar em "Criar banco de dados";
- Crie um banco de dados com as seguintes configurações:
    - Método de criação: `Ciração padrão`;
    - Opções de mecanismo: `MySQL`;
    - Modelos: `Nível Gratuito`;
    - Identificador da instância de banco de dados: Escolher um nome, por exemplo `wordpressdb`;
    - Configurações de credenciais: Escolher um nome de usuário principal e sua senha;
    - Configuração da instância: Manter a configuração padrão;
    - Armazenamento: Manter a configuração padrão;
    - Recurso de computação: `Não se conectar a um recurso de computação do EC2`; 
    - Tipo de rede: `IPv4`;
    - VPC: Selecionar a VPC criada anteriormente;
    - Grupo de sub-redes de banco de dados: Manter a configuração padrão;
    - Acesso público: `Sim`;
    - Grupo de segurança de VPC: Selecionar o grupo criado anteriormente;
    - Autenticação de banco de dados: `Autenticação de senha`;
    - Monitoramento: Manter desabilitado a opção `Hablitar monitoramento avançado`;
    - Em "Configuração adicional", apenas adicionar um nome para o banco de dados inicial (por exemplo, `wp_db`).
- Clicar em "Criar banco de dados".

<br>

### Subir o container Wordpress 
- Acessar a instância privada já configurada com Docker e Docker-compose;
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
- Dentro do mesmo diretório em que o arquivo mencionado acima estiver, use o comando `docker-compose up`.

<br>

# Criar/Configurar: Load Balancer, AMI, Modelo de execução e Auto Scaling
### Criar Load Balancer
- Antes de criar o Load Balancer, precisamos criar um grupo de segurança para ele. Portanto, vá até o serviço EC2 e entre em "Grupos de segurança";
- Crie um grupo de segurança com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ---- | ---- | ---- | ----
  HTTP | TCP | 80 | 0.0.0.0/0

- Além do grupo de segurança, precisamos criar um grupo de destino. Ainda no serviço EC2, entre na aba de "Grupos de destino";
- Crie um grupo de destino com as seguintes configurações:
  - Tipo de destino: `Instâncias`;
  - Nome do grupo de destino: Escolha um nome (por exemplo, `TG-AT`);
  - Protocolo: `HTTP` | Porta: `80`;
  - VPC: Selecione a VPC criada anteriormente;
  - Versão do protocolo: `HTTP1`;
  - Protocolo da verificação de integridade: `HTTP`;
- No serviço EC2 da AWS, selecione a aba "Load Balancers" e clique em "Criar load balancer";
- Em "Tipos de load balancer", selecionar `Application Load Balancer`;
- Configure o load balancer da seguinte forma:
  - Nome: Esolha um nome (por exemplo, `LB-AT`);
  - Esquema: `Voltado para a internet`;
  - Tipo de enereço IP: `IPv4`;
- Em "Mapeamento de rede":
  - VPC: VPC criada anteriormente;
  - Mapeamentos: Selecionar as duas zonas de disponibilidade, com a sub-rede publica;
- Em "Grupos de segurança", selecionar o criado anteriomente;
- Em "Listeners e roteamento", configurar da seguinte maneira:
  - Protocolo: `HTTP`
  - Porta: `80`
  - Ação padrão: Selecionar o grupo de destino criado anteriormente;
- Clicar em "Criar load balancer".

<br>

### Criar AMI
- Antes de criar e configurar o auto scaling, precisamos criar uma AMI com base na instância privada que criamos anteriormente;
- Ir na aba de instâncias e clicar com o botão direito na instância privada configurada anteriormente;
- Selecionar "Imagem e modelos" e clicar em "Criar imagem";
- Escolher um nome (por exemplo, `AMI-AT`) e uma descrição;
- Mantenha as demais configurações padrão e clique em "Criar imagem".

<br>

### Criar Modelo de Execução
- Além da AMI, antes de criar o auto scaling precisamos criar um `Modelo de execução`, presente no menu esquerdo do serviço EC2 da AWS;
- Clicar em "Criar modelo de execução";
- Escolher um nome (por exemplo, ME-AT) e uma descrição;
- Em "Imagens de aplicação e de sistema operacional", escolher a imagem criada anteriormente;
- Em "Tipo de instância", selecionar `t2.micro`;
- Em "Pares de chaves", você pode selecionar uma chave `.pem` já criada ou criar outra;
- Em "Configurações de sub-rede", configurar da seguinte maneira:
  - Sub-rede: Não incluir no modelo de execução
  - Firewall: Selecionar o grupo de segurança `privado` criado anteriormente
- Em "Armazenamento", verificar apenas se a opção "Excluir no encerramento" está como `Sim`;
- Clique em "Criar modelo de execução".

<br>

### Criar Auto Scaling
- Ainda no painel EC2, no menu esquerdo clicar em "Grupos Auto Scaling";
- Escolher um nome (por exemplo, AS-AT) e selecionar o modelo de execução criado anteriormente;
- Em "Rede", configurar da seguinte maneira:
  - VPC: VPC criada anteriormente
  - Zonas de disponibilidade e sub-redes: Selecionar as duas sub-redes privadas criadas anteriormente
- Clicar em "Próximo";
- Configurações de "Balanceamento de carga":
  - Selecione "Anexar a um balanceador de carga existente";
  - Em "Anexar a um balanceador de carga existe", mantenha selecionado "Escolha entre seus grupos de destino de balanceador de carga" e selecione o grupo de destino criado anteriormente
- Em "Verificações de integridade", selecione `Ative as verificações de integridade do Elastic Load Balancing`;
- Clique em "Próximo";
- Configurações de "Tamanho do grupo":
  - Capacidade desejada: 2
  - Capacdade mínima: 2
  - Capacidade máxima: 4
- Em "Políticas de escalabilidade", marque "Política de dimensionamento com monitoramento do objetivo";
- Clique em "Próximo";
- Não adicione nada em notificações, clique em "Próximo";
- Não adicione nada em etiquetas, clique em "Próximo";
- No painel de "Análise", verifique se as configurações estão corretas e clique em "Criar grupo do Auto Scaling".
