A preferência é utilizar porta disponível no roteador de borda (roteador SBT) do site remoto.

#### 🚪 Se possuir porta disponível no roteador de borda:
~~~bash
interface GigabitEthernet?/?
 description S2S
 ip address <ip interface local S2S, disponível no PHPIPHAM> 255.255.255.248
~~~
Na sequência aplicar as configurações de "Listas de Acesso" em diante.

#### 🔀 Caso contrário (via switch SBT):
~~~bash
interface GigabitEthernet0/23
 description Uplink S2S
 switchport access vlan 175
 switchport mode access
~~~

#### 💡 No roteador de borda do site remoto:

##### Subinterface:
~~~bash
interface GigabitEthernet0/0.175
 description S2S
 encapsulation dot1Q 175
 ip address <ip interface local S2S> 255.255.255.248
~~~

##### 📑 Listas de Acesso:
~~~bash
ip access-list extended LAN_ADM
 permit ip <rede_adm> <máscara_wildcard> any

ip access-list extended LAN_OPR
 permit ip <rede_opr> <máscara_wildcard> any

ip access-list extended SBT
 permit ip 10.115.92.0 0.0.3.255 10.115.92.0 0.0.3.255
 permit ip 10.115.92.0 0.0.3.255 10.115.96.0 0.0.3.255
 permit ip 10.115.96.0 0.0.3.255 10.115.92.0 0.0.3.255
 permit ip 10.115.96.0 0.0.3.255 10.115.96.0 0.0.3.255

ip access-list extended SSH_TELNET_ICMP
 permit tcp any any eq 22
 permit tcp any any eq telnet
 permit icmp any any

ip access-list extended LAN_C4
 permit ip any 10.112.0.0 0.0.255.255
~~~

##### 🔁 OSPF para interface S2S local:
~~~bash
router ospf 1
 network <rede local S2S> 0.0.0.7 area <área OSPF>
~~~

##### 📶 IP SLA e Track:
~~~bash
ip sla 4
 icmp-echo 10.112.82.215 source-ip <ip S2S local>
 frequency 5
ip sla schedule 4 life forever start-time now

track 4 ip sla 4 reachability
 delay down 2 up 10
~~~

##### 📌 Route-Map:
~~~bash
route-map administrativo permit 10
 match ip address LAN_C4
 set ip next-hop verify-availability <ip CPE S2S> 1 track 4
 set ip next-hop verify-availability <ip CPE MPLS> 2 track 3
 set ip next-hop <ip TUNEL remoto>

route-map administrativo permit 20
 match ip address LAN_ADM
 set ip next-hop verify-availability <ip CPE MPLS> 1 track 3
 set ip next-hop verify-availability <ip CPE S2S> 2 track 4
 set ip next-hop <ip TUNEL remoto>

route-map sbt permit 10
 match ip address SBT
 set ip next-hop verify-availability <ip TUNEL remoto> 1 track 1
 set ip next-hop verify-availability <ip TOPNET remoto> 2 track 2
 set ip next-hop verify-availability <ip CPE S2S> 3 track 4
 set ip next-hop verify-availability <ip CPE MPLS> 4 track 3

route-map operacional permit 10
 match ip address LAN_OPR
 set ip next-hop verify-availability <ip CPE S2S> 1 track 4
 set ip next-hop verify-availability <ip CPE MPLS> 2 track 3
 set ip next-hop verify-availability <ip TUNEL remoto> 3 track 1
~~~

##### 🚧 Aplicar os `ROUTE-MAP` nas interfaces LAN:
~~~bash
interface GigabitEthernet0/0
 description LAN SBT
 ip policy route-map sbt

interface GigabitEthernet0/1
 description LAN OPR
 ip policy route-map operacional

interface GigabitEthernet0/2
 description LAN ADM
 ip policy route-map administrativo
~~~
---
