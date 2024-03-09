# Guia do Trabalho

# Configuração das redes

- Escolha a classe de endereço que quer usar para os PCs (A, B ou C). Para os roteadores o ip é falso (192.168.0.x), então não use esse valor de IP. 
- Coloque todas as portas entre os switches no modo trunk. A conexao entre o roteador da rede e do switch core também precisa ser trunk. 

## Faça a divisão das vlans e roteamento interno dentro de cada rede

### No switch core de cada rede

- Para cada vlan
Switch(config)#vlan x
- Rodar esses comandos uma única vez
Switch(config)#vtp v 2
Switch(config)#vtp m s
Switch(config)#vtp d nome_da_rede
<!--- Você escolhe o nome da rede, pode ser qualquer coisa que voce preferir --->

### No roteador de cada rede
	
Router#vlan database
- Para cada vlan:
Router(vlan)#vlan x
- Depois, execute o comando para cada vlan depois de ter selecionado a interface gigabit que conecta o roteador no core:
Router(config)#int gi0/0.x
Router(config-if)#enc dot1Q x
Router(config-if)#ip add a.b.c.d mascara
<!--- a, b, c e d são os octetos da sua classe de ip. Eu usei a classe A para os enderecos, então por exemplo ficaria 30.0.11.1, com mascara 255.255.255.0, mas você que tem que definir como vai dividir as vlans e qual mascara vai usar pra poder escolher esses valores. É a mesma coisa do trabalho anterior, mas com IPs verdadeiros --->
Router(config-if)#ip help endereco_server
<!--- O IP do server precisa ser estático. Escolha um endereço e coloque ele nesse campo ---> 

### Em todos os outros switches

- Se vc rodar o comando "sh vl br" e as vlans da rede não aparecerem:
Switch(config)#vtp m c

### Nos roteadores da AS e do backbone (que ligam as AS's)

- Faça a divisão de subredes para IPs falsos de sua escolha (192.168.x.y)
- Para cada enlace entre os roteadores, coloque um IP dentro de cada subrede pra cada um
- Coloque o ip na porta que liga um roteador ao outro. Ex:
Router(config)#int se0/0/0
Router(config-if)#ip add 192.168.0.1 255.255.255.0
- Se tiver duvida, use a imagem de referência e aplique a estrutura 192.168.vlan.id
<!--- vlan aqui é o numero da vlan de comunicação entre os roteadores e id é o ultimo octeto que identifica qual roteador é qual --->

# Configuração dos Roteadores da AS

## Em todos os roteadores da AS (Router 1 1, Router 1 5, ...) 
	
Router(config)#route ospf prox_id
<!--- Eu usei o prox_id como o numero da AS nos roteadores dentro das ASs --->
Router(config-router)#redistribute bgp 1 subnets
<!--- Esse comando distribui os IPs das outras redes pras subredes dentro desse roteador --->
- Para cada subrede que faz uma conexão entre os roteadores
Router(config-router)#network 192.168.x.0 mask_wildcard area area_num

### Legenda

- prox_id: id da AS (você define)
- area_num: numero de enderecamento para os roteadores
- mask_wildcard: mascara invertida. Ex: 0.0.0.255 ao inves de 255.255.255.0 para as redes
										0.0.0.15 ao inves de 255.255.255.240 para subredes de roteadores
- x: numero da VLAN

### Observações: 

- Conectar todas vlans que se conectam naquele roteador
- usar "sh ip ro" pra conferir se todos os IPs estão funcionando
- Se houver uma subrede entre os roteadores, usa-se a marca invertida com subtração

# Configuração dos Roteadores de Borda e do backbone

## Conexões

- Usar dois roteadores intermediarios para conectar as AS'se

### Para comunicação entre os roteadores de borda
Router(config)#router bgp id_bgp
- Para cada roteador que está conectado com o roteador atual
Router(config-router)#neighbor ip_roteador remote-as id_bgp
<!--- o id_bgp nesse caso é o do roteador que está sendo referenciado --->

- Usa o mesmo conceito de apontar para quem ir se quiser ir para uma AS como a gente faz no roteamento

### Legenda

- id_bgp: id da AS (se for roteador de borda) ou do backbone
- ip_roteador: ip da porta do roteador está conectada com o roteador que está sendo configurado. 
  Ex: se a porta é a serial 0/0/0 e essa porta ta ligada na serial 0/1/1 de outro roteador, usar ip_roteador como o ip configurado na 0/1/1 do outro roteador

### adicional (não necessário, mas o Moro colocou no projeto dele)
Router(config-router)#default-information originate
