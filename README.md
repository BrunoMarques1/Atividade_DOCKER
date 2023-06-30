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

Nome:  | SUB-privada01  | SUB-publica01  | SUB-privada02  | SUB-publica02
------------- | ------------- | ------------- | ------------- | -------------
Zona de disponibilidade:  |  `us-east-1a`  |  `us-east-1a`  |  `us-east-1b`  |  `us-east-1b` 
Bloco CIDR IPv4:  | `172.28.0.0/24`  | `172.28.1.0/24`  | `172.28.3.0/24`  | `172.28.2.0/24`
- Após criar todas as sub-redes, selecionar uma sub-redes pública e clicar em "ações", depois em "Editar configurações de sub-rede", selecionar a caixa "Habilitar endereço IPv4 público de atribuição automática" e salvar;
- Realizar os mesmos passos anteriores para a outra sub-rede pública.

### Criar Gateway da internet
- No menu na parte esquerda, clicar em "Gateways da Internet";
- No canto superior direito, clicar em "Criar gateway da internet";
- Escolher um nome(no meu caso coloquei "IGW-AT"), e criar o gateway;
- Selecionar o gateway da internet e associa-lo à VPC criada anteriormente.

### Criar Gateway NAT
- No menu na parte esquerda, clicar em "Gateways NAT";
- No canto superior direito, clicar em "Criar gateway NAT";
- Escolher um nome(no meu caso coloquei "NGW-AT"), e criar o gateway;
- Selecionar uma sub-rede publica;
- Em "Tipo de conectividade" manter "Público" selecionado;
- Clicar em "Alocar IP elástico";
- Por fim, clicar em "Criar gateway NAT". 

### Criar tabelas de rotas
- No menu na parte esquerda, clicar em "Tabela de rotas";
- No canto superior direito, clicar em "Criar tabela de rotas";
- Criar duas tabelas de rotas, uma para ser privada e outra publica, ambas usando a VPC criada anteriormente;
- Após ter as duas tabelas crias, associar as sub-redes em suas respectivas tabelas(com as sub-redes publicas na tabela publica e as sub-redes privadas na tabela privada);
- Configurando tabela de rotas publica:
  - Selecionar a tebela de rotas publica e clicar na opção "Rotas" e depois em "Editar rotas";
  - Clicar em "Adicionar rota", na parte de "Destino" selecionar `0.0.0.0/0` e em "Alvo" escolher "Gateway da Internet" e selecionar o gateway criado anteriormente.
- Configurando tabela de rotas privada:
  - Selecionar a tebela de rotas privada e clicar na opção "Rotas" e depois em "Editar rotas";
  - Clicar em "Adicionar rota", na parte de "Destino" selecionar `0.0.0.0/0` e em "Alvo" escolher "Gateway NAT" e selecionar o gateway criado anteriormente.
