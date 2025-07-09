#### 🔄 Aplicar no grupo **GLBP** (80.201 e 80.202)

#### 📋 Criar `PREFIX-LIST` com a rede remota (.87 ou .88)
~~~bash
ip prefix-list S2S_<sigla do site> seq 20 permit <rede S2S do site>/29
~~~

#### 🔁 Associar o `PREFIX-LIST` ao `ROUTE-MAP`
~~~bash
route-map REDIST_REDE_500_TO_OSPF1 permit 10 
 match ip address prefix-list S2S_<sigla do site>
 set metric 20
~~~

#### 📶 Criar `IP SLA`
~~~bash
ip sla <proximo id disponível>
 icmp-echo <ip do S2S no site remoto> source-ip 10.112.82.211 (atentar que no 80.202 o source-ip será 10.112.82.212)
 tag Monitoracao S2S <designativo do site remoto>
 frequency 5

ip sla schedule <mesmo id acima> life forever start-time now
~~~

#### 📶 Criar listas de acesso para identificar a rede operacional e administrativa do site remoto
~~~bash
ip access-list extended LAN_ADM_DTCEA_<sigla do site>
 permit ip 10.0.0.0 0.255.255.255 <endereço de rede adm do site remoto, conforme PHPIPHAM> 0.0.3.255
!
ip access-list extended LAN_OPR_DTCEA_<sigla do site>
 permit ip 10.0.0.0 0.255.255.255 <endereço de rede opr do site remoto, conforme PHPIPHAM> 0.0.3.255
~~~

#### 📶 Adicionar `ROUTE-MAP`
~~~bash
route-map PBR permit <proximo id disponivel> 
 match ip address LAN_ADM_DTCEA_<sigla do site>
 set ip next-hop verify-availability 10.112.82.215 1 track <id do `IP SLA` criado acima>
 set ip next-hop verify-availability 10.112.81.130 2 track <id do `IP SLA` usado para monitorar o mpls do site remoto>
~~~

---
