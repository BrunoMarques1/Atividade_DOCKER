### Criar VPC
- Acesse o serviço VPC da AWS;
- No canto superior esquerdo, clique em "Criar VPC";
- Selecione "Somente VPC";
- Escolha um nome para a VPC (por exemplo, "VPC-AT");
- Na seção "Bloco CIDR IPv4", selecione "Entrada manual de CIDR IPv4" e, em seguida, insira o CIDR IPv4 desejado (por exemplo, 172.28.0.0/16);
- Na seção "Bloco CIDR IPv6", mantenha selecionada a opção "Nenhum bloco IPv6";
- Mantenha a opção "Padrão" selecionada em "Locação";
- Clique em "Criar VPC".

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

### Criar Gateway da internet
- No menu na parte esquerda, clique em "Gateways da Internet";
- No canto superior direito, clique em "Criar gateway da internet";
- Escolha um nome (por exemplo, "IGW-AT") e crie o gateway.;
- Selecione o gateway da internet criado e associe-o à VPC criada anteriormente.

### Criar Gateway NAT
- No menu na parte esquerda, clicar em "Gateways NAT";
- No canto superior direito, clicar em "Criar gateway NAT";
- Escolha um nome (por exemplo, "NGW-AT") e crie o gateway;
- Selecione uma sub-rede publica;
- Em "Tipo de conectividade", mantenha selecionada a opção "Público";
- Clicar em "Alocar IP elástico";
- Por fim, clicar em "Criar gateway NAT". 

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
 
### Criar Bastion Host
- Acesse o serviço EC2 da AWS e clique em "Executar instâncias";
- Escolha um nome (por exemplo, "BASTION-HOST");
- Na parte de imagem, escolha `Amazon Linux 2023 AMI`;
- Em tipo de instância, escolha `t2.micro`;
- Em pares de chaves, você pode usar uma chave existente no formato .pem, caso contrário, crie uma nova chave do tipo RSA no formato .pem;
- Em grupos de segurança, crie um com as seguintes regras de entrada:

  Tipo | Protocolo | Intervalo de portas | Origem
  ------------- | ------------- | ------------- | -------------
  SSH | TCP | 22 | 0.0.0.0/0

- Em armazenamento, mantenha o padrão de 1x 8 GiB gp3;
- Clique em "Executar instância".

### Criar Instância Privada (Wordpress)
- Ainda no serviço EC2 da AWS, clique novamente em "Executar instâncias";
- Escolha um nome (por exemplo, "WORDPRESS");
- Na parte de imagem, escolha `Amazon Linux 2023 AMI`;
- Em tipo de instância, escolha `t2.micro`;
- Em pares de chaves, você pode usar uma chave existente no formato .pem, caso contrário, crie uma nova chave do tipo RSA no formato .pem;
- Crie um novo grupo de segurança com as seguintes regras de entrada (Lembrando que o Load-Balance ainda não foi criado, os endereços IP podem ser alterados/adicionados posteriormente):

  Tipo | Protocolo | Intervalo de portas | Origem
  ------------- | ------------- | ------------- | -------------
  SSH | TCP | 22 | Endereço IPv4 privado do `Bastion-Host`
  HTTP | TCP | 80 | Endereço IP privado do `Load-Balance`
  HTTPS | TCP | 443 | Endereço IP privado do `Load-Balance`

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


