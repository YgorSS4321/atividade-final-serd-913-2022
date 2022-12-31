# samba server in Ubuntu-server

  *configurações basicas para VMs (configuração do netplan, openvpn3, ssh para as VMs remotas): <a href="" >link para as configurações padrão<a/>
  
```bash
$ sudo apt update
```
```bash
$ sudo apt install samba
```
 
```bash
$ whereis samba
```
 
 <img src="/prints_de_tela/samba/Captura de tela de 2022-12-23 07-46-29.png" width=1000/>
 
```bash
$ sudo systemctl status smbd.service
```
 
  <img src="/prints_de_tela/samba/Captura de tela de 2022-12-23 07-47-04.png" width=1000/>
 
```bash
 $ netstat -an | grep LISTEN
```
 
<img src="/prints_de_tela/samba/Captura de tela de 2022-12-23 07-58-05.png" width=1000/>
 
```bash
$ sudo nano /etc/samba/smb.conf
```
 
 Adicionando ao arquivo de configuração ```/etc/smb.conf```, em ```interfaces``` de global, os nomes dos adaptadores de rede, que no nosso caso será ```ens160 ens192``` 
 E Adicionando no final do arquivo as seguintes linhas: 
 ```
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```
 Para então reestartar o serviço samba com:
```bash
$ sudo systemctl restart smbd
```

<img src="/prints_de_tela/samba/Captura de tela de 2022-12-23 07-47-08.png" width=1000/>
<img src="/prints_de_tela/samba/Captura de tela de 2022-12-23 07-47-13.png" width=1000/>
 
 Para acessar os arquivos do servidor samba será necessário que um usuário do servidor samba esteja cadastrado no samba e esteja também cadastrado no grupo o qual o servidor samba é especificado em ```valid user = sambashare```, isso é feito da seguinte forma:
 
```bash 
$ sudo smbpasswd -a aluno
```
   em que ```aluno``` é um usuário do servidor ubuntu-server que hospeda o ```samba```

```
$ sudo usermod -aG sambashare aluno
```
   - adiciona o ```aluno``` no grupo ```sambashare``` (o grupo especificado em ```valid users = sambashare```)
 
é preciso que a pasta a ser compartilhada no samba esteja com as permissões configuradas para acesso remoto:
 
```bash
$ sudo chown -R nobody:nogroup /samba/public
$ sudo chmod -R 0775 /samba/public
$ sudo chgrp sambashare /samba/public
```
   - exemplo de configuração de permissão para uma pasta ´´´/samba/public```
 
```bash
$ sudo systemctl enable smbd.service
```

```bash
$ sudo systemctl status service
```
 
 *pronto, agora está funcionando
 
 - agora pelo explorador de arquivos, digite na barra do path o ip do host da VM que hospeda o servidor samba na porta 445 ou 139
```
smb://10.9.13.105
```
 
se tudo der certo, aparecerá os arquivos no gerenciador de tarefas



*Obs: se o gateway estiver implementado na rede local, será necessário verificar o gateway na VM do servidor samba pelo 
 ```
 $ telnet 10.9.13.105 445
 ```
 depois escrever o ip do gateway na porta 445
 
```
smb://10.9.13.109
``` 
 
 <img src="/prints_de_tela/samba/Captura de tela em 2022-12-23 18-06-14.png" width=1000/>

 
 
 
 [voltar para o README.md](https://github.com/YgorSS4321/atividade-final-serd-913-2022/blob/main/README.md)

 
 
 
