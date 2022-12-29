# DNS Master

instalando os pacotes referentes ao dns

```
sudo apt-get install bind9 dnsutils bind9-doc 
```

```
sudo systemctl status bind9.service
```

se não estiver funcionando, o `bind9` pode ser ativados pelo seguinte código:
```
sudo systemctl enable bind9 
```

## Configurar os arquivos bind do DNS Master

 * Os arquivos do bind ficam na no diretório **/etc/bind**. 
```bash
ls /etc/bind
```
```
total 64
drwxr-sr-x  3 root bind 4096 Oct 15 09:06 .
drwxr-xr-x 94 root root 4096 Oct  8 23:29 ..
-rw-r--r--  1 root root 2761 Aug  7 14:43 bind.keys
-rw-r--r--  1 root root  237 Aug  7 14:43 db.0
-rw-r--r--  1 root root  271 Aug  7 14:43 db.127
-rw-r--r--  1 root root  237 Aug  7 14:43 db.255
-rw-r--r--  1 root root  353 Aug  7 14:43 db.empty
-rw-r--r--  1 root root  270 Aug  7 14:43 db.local
-rw-r--r--  1 root root 3171 Aug  7 14:43 db.root
-rw-r--r--  1 root bind  463 Aug  7 14:43 named.conf
-rw-r--r--  1 root bind  490 Aug  7 14:43 named.conf.default-zones
-rw-r--r--  1 root bind  468 Oct 15 05:42 named.conf.local
-rw-r--r--  1 root bind  881 Sep 19 15:08 named.conf.options
-rw-r-----  1 bind bind   77 Sep 19 14:48 rndc.key
-rw-r--r--  1 root root 1317 Aug  7 14:43 zones.rfc1918
```
### Zonas
as zonas são encontradas na pasta **/etc/bind/zones**, em arquivos **db.**
```
sudo mkdir /etc/bind/zones
```
### criar arquivos db.
* os arquivos db. armazenam localmente no servidor informações para a resolção de nomes sites, tendo, por exemplo, **db.site.com** e **db.outrosite.com**

#### criar zona direta
* nesse exemplo será usado o nome de domínio **grupo2.turma913.ifalara.local**
* para isso será usado uma cópia do modelo padrão de arquivo **db.** encontrado em **/etc/bind/db.empty**
```
sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo2.turma913.ifalara.local 
```
#### criar zona reversa
* utilizado para fazer a tradução nome --> ip
* para isso será usado uma cópia de modelo padrão de arquivo **.rev**, encontrado em **/etc/bind/db.127**
* o ip  da sub-rede desse exemplo é `10.9.13.0`, que será colocado no nome do arquivo reverso da seguinte forma:

```
sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.13.rev
```

##### editando arquivos db

```
sudo nano db.grupo2.turma913.ifalaralocal 
```

para preencher o arquivo **/etc/bind/zones/db.grupo2.turma913.ifalaralocal** será da seguinte forma:

```
;
; BIND data file for internal network
;
$ORIGIN <domainName>.
$TTL	3h
@	IN	SOA	ns1.<domainName>. root.<domainName>. (
			      2		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.<domainName>.
@	IN	NS	ns2.<domainName>.
;hosts
ns1.<domainName>.	  IN	A	<respectivoIP>
ns2.<domainName>.	  IN	A	<respectivoIP>
gw.<domainName>.	  IN    A	<respectivoIP>
smb.<domainName>.	  IN    A	<respectivoIP>
www.<domainName>.	  IN	A	<respectivoIP>
bd.<domainName>.	  IN    A	<respectivoIP>

```
* isso substituindo `<domainName>` por `grupo2.turma913.ifalara.local.`
  e `<respectivoIP>` pelos IPs das máquinas, o arquivo é preenchido da seguinte forma:
```
;
; BIND data file for internal network
;
$ORIGIN grupo2.turma913.ifalara.local.
$TTL	3h
@	IN	SOA	ns1.grupo2.turma913.ifalara.local. root.grupo2.turma913.ifalara.local. (
			      2		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.grupo2.turma913.ifalara.local.
@	IN	NS	ns2.grupo2.turma913.ifalara.local.
;hosts
ns1.grupo2.turma913.ifalara.local.	  IN	A	10.9.13.127
ns2.grupo2.turma913.ifalara.local.	  IN	A	10.9.13.134
gw.grupo2.turma913.ifalara.local.	  IN    A	10.9.13.109
smb.grupo2.turma913.ifalara.local.	  IN    A	10.9.13.105
www.grupo2.turma913.ifalara.local.	  IN	A	10.9.13.213
bd.grupo2.turma913.ifalara.local.	  IN    A	10.9.13.214

```
#### zona reversa: db.10.9.13.rev
* adicionar as linhas da zonas reversa

```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	grupo2.turma913.ifalara.local. root.grupo2.turma913.ifalara.local. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;name servers
@	IN	NS	ns1.grupo2.turma913.ifalara.local.
@	IN	NS	ns2.grupo2.turma913.ifalara.local.

;PTR Records
127	IN	PTR	ns1.grupo2.turma913.ifalara.local.      ;10.9.13.127
134	IN	PTR	ns2.grupo2.turma913.ifalara.local.      ;10.9.13.134
213	IN	PTR	www.grupo2.turma913.ifalara.local.      ;10.9.13.213
109	IN	PTR	gw.grupo2.turma913.ifalara.local.      ;10.9.13.109
105	IN	PTR	smb.grupo2.turma913.ifalara.local.      ;10.9.13.105
214	IN	PTR	bd.grupo2.turma913.ifalara.local.      ;10.9.13.214
```

### configuração do arquivo named.conf
* para ativar as zonas descritas, deve-se alterar o arquivo **/etc/bind/named.conf.local**
  
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "grupo2.turma913.ifalara.local" {
	type master;
	file "/etc/bind/zones/db.grupo2.turma913.ifalara.local";
        //file "/etc/bind/zones/db.labredes.ifalarapiraca.local";
	allow-transfer{ 10.9.13.134; };  
	allow-query{any;};
};

zone "13.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.13.rev";
	allow-transfer{ 10.9.13.134; };
};

```
#### verificar sintaxe do arquivo named.conf.local
```
sudo named-checkconf
```


#### verificar sintaxe dos arquivos db.

```
$ cd /etc/bind/zones
$ sudo named-checkzone grupo2.turma913.ifalara.local db.grupo2.turma913.ifalara.local
zone grupo2.turma913.ifalara.local/IN: loaded serial 1
OK
$ sudo named-checkzone 13.9.10.in-addr.arpa db.10.9.13.rev
zone 13.9.10.in-addr.arpa/IN: loaded serial 1
OK
```

### configurar para resolver apenas IPs IPv4

```
sudo nano /etc/default/named
```
e adicione a linha **OPTIONS="-4 -u bind"**:
```
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
```


## Configuração dos clientes

* adicionar no arquivo **/etc/netplan/00-installer-config.yaml**, na interface `ens192` os nameservers:
```
nameservers:
         addresses:
           - 192.168.13.27
           - 192.168.13.28
         search: [grupo2.turma913.ifalara.local]

```

## testando servidor DNS

```
systemd-resolve --status ens160
```

```
Link 2 (ens160)
      Current Scopes: DNS                          
DefaultRoute setting: yes                          
       LLMNR setting: yes                          
MulticastDNS setting: no                           
  DNSOverTLS setting: no                           
      DNSSEC setting: no                           
    DNSSEC supported: no                           
  Current DNS Server: 10.9.13.134                  
         DNS Servers: 10.9.13.127                  
                      10.9.13.134                  
          DNS Domain: grupo2.turma913.ifalara.local
```

## testando o DNS para o ns1

```
dig ns1.grupo2.turma913.ifalara.local
```

```
; <<>> DiG 9.16.1-Ubuntu <<>> ns1.grupo2.turma913.ifalara.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38459
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;ns1.grupo2.turma913.ifalara.local. IN	A

;; ANSWER SECTION:
ns1.grupo2.turma913.ifalara.local. 6714	IN A	10.9.13.127

;; Query time: 4 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Thu Dec 29 11:49:30 UTC 2022
;; MSG SIZE  rcvd: 78
```

### testando o DNS Reverso para a máquina ns1

```
dig -x 10.9.13.127
```

```
; <<>> DiG 9.16.1-Ubuntu <<>> -x 10.9.13.127
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21247
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;127.13.9.10.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
127.13.9.10.in-addr.arpa. 604800 IN	PTR	ns1.grupo2.turma913.ifalara.local.

;; Query time: 52 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Thu Dec 29 11:50:06 UTC 2022
;; MSG SIZE  rcvd: 100

```

### DNS reverso para o ns2

```
dig -x 10.9.13.134
```

```
; <<>> DiG 9.16.1-Ubuntu <<>> -x 10.9.13.134
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62335
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;134.13.9.10.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
134.13.9.10.in-addr.arpa. 6544	IN	PTR	ns2.grupo2.turma913.ifalara.local.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Thu Dec 29 11:52:47 UTC 2022
;; MSG SIZE  rcvd: 100

```











