# COMO INSTALAR E CONFIGURAR ZABBIX AGENTE NO OPENBSD

> Manual para instalar e configurar agente zabbix no OpenBSD

**1. Logado no OpenBSD torne-se root para seguir os proximos passos.**

```
su -
```

**1.1. O arquivo vai ter uma dos dois repositorios comente o repositorio atual e adicione um novo**

```
vi /etc/installurl
```
> caso o arquivo tenha os dois repositorios faça um teste com cada um e veja qual vai funcionar para voce.
{.is-warning}

```
	#https://cloudflare.cdn.openbsd.org/pub/OpenBSD/
	http://ftp.eu.openbsd.org/pub/OpenBSD/
	
	https://openbsd.c3sl.ufpr.br/pub/OpenBSD	# Repositorio nativo OpenBSD 7.5 (2024)
```

**2. Instalar o Agente do Zabbix**

```
pkg_add -U zabbix-agent
```

**2.1. Editar o arquivo de configuração do agent `/etc/zabbix/zabbix_agentd.conf`.**

```
vi /etc/zabbix/zabbix_agentd.conf
```

```
	# Alterar o Server, ServerActive, Hostname.
	Server= <seu ip>
	ServerActive=<seu ip>
	Hostname=srv-openbsd #(***nome do host local***)

	# Adicionar as linhas abaixo na categoria UserParameter.
	UserParameter=sistema.operacional, uname -sr
	UserParameter=kernel, uname -v
	UserParameter=discos, dmesg | egrep -e sd.?: | cut -f 1 -d ','
	UserParameter=ip.internal, ifconfig vic0 | grep 'inet' | awk -F ' ' '{ print $2 }' #OK
```

**3. Fazer liberação no firewall**

```
vi /etc/pf.conf
```

**3.1. Configurar  o pf.conf**

> Em # SERVIDORES
> server_zabbix = "seu ip"		# server_gerencia2 = "seu ip"
> Em # PORTAS
> zabbix = 10050

>	# Permite Monitoramento (Zabbix)
>	pass in quick log on $ext_if proto icmp from $server_zabbix to $ext_ip
>	pass in quick log on $ext_if proto tcp from $server_zabbix to $ext_ip port $zabbix
>	pass in quick log on $ext_if proto udp from $server_zabbix to $ext_ip port snmp


**3.2. Lê e carrega as regras definidas no pf.conf**

```
pfctl -f /etc/pf.conf
```

**3.3. Ativar o seviço Zabbix Agent.**

```
rcctl enable zabbix_agentd
```

**3.4. Reiniciar o serviço Zabbix Agent.**

```
rcctl restart zabbix_agentd
```
**3.5. Checar o serviço Zabbix Agent.**

```
rcctl check zabbix_agentd
```
