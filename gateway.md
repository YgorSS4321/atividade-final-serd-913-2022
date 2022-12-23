# gateway in Ubuntu-server

  *configurações basicas para VMs (configuração do netplan, openvpn3, ssh para as VMs remotas): <a href="" >link para as configurações padrão<a/>
  
```bash
$ sudo apt update
```
*configurar ufw
 
*Configurar as interfaces de rede (netplan)
```
$ sudo nano /etc/netplan/50-cloud-init.yaml 
```

```
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [10.0.0.1/24]
            dhcp4: false              
    version: 2
```

```
$ sudo netplan apply
```

```
$ ifconfig -a
```

*configurar ```/etc/rc.local```

```
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i ens160 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i ens192 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i ens160 -o ens192 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i ens192 -o ens160 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE



#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 445 -j DNAT --to 10.9.13.105:445
iptables -A FORWARD -p tcp -d 10.9.13.105 --dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 139 -j DNAT --to 10.9.13.105:139
iptables -A FORWARD -p tcp -d 10.9.13.105 --dport 139 -j ACCEPT


#DNS
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 53 -j DNAT --to 10.9.13.127:53
iptables -A FORWARD -p udp -d 10.9.13.127 --dport 53 -j ACCEPT

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```

*reconfigurar ufw (...)

*configurar outras máquinas para acessarem o gateway







 [voltar para o README.md](https://github.com/YgorSS4321/atividade-final-serd-913-2022/blob/main/README.md)

 
 
 
