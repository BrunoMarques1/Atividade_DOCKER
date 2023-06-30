### Criar VPC
- Ir no serviço VPC da AWS;
- No canto superior esquerdo, clicar em "Criar VPC";
- Selecionar "Somente VPC";
- Escolher um nome para a VPC (no meu caso o nome escolhido foi "VPC-AT");
- Na parte de Bloco CIDR IPv4 escolher "Entrada manual de CIDR IPv4" e logo abaixo indicar o CIDR IPv4 (no meu caso escolhi 172.28.0.0/16);
- Na parte de Bloco CIDR IPv6 manter selecionado "Nenhum bloco IPv6";
- Em locação também manter a opção "Padrão" selecionada;
- Clicar em "Criar VPC".

### Criar Sub-Redes
- Ainda no serviço de VPC, no menu do lado esquerdo clicar em "Sub-redes";
- Clicar em "Criar sub-rede, no canto superior na direirta;
- Em "ID da VPC" selecionar a VPC criada anteriormente;
- Configruar quatro sub-redes, com as seguintes configurações:

Nome:  | SUB-privada01  | SUB-publica01  | SUB-privada02  | SUB-publica02
------------- | ------------- | ------------- | ------------- | -------------
Zona de disponibilidade:  |  `us-east-1a`  |  `us-east-1a`  |  `us-east-1b`  |  `us-east-1b` 
Bloco CIDR IPv4:  | `172.28.0.0/24`  | `172.28.1.0/24`  | `172.28.3.0/24`  | `172.28.2.0/24`
- Após criar todas sub-redes, selecionar umas sub-redes publicas e clicar em "ações", depois em "Editar configurações de sub-rede", selecionar a caixa "Habilitar endereço IPv4 público de atribuição automática" e salvar;
- Fazer os mesmos passos anterires na outra sub-rede publica;

### Criar tabelas de rotas
