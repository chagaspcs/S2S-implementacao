O projeto S2S provê conectividade entre o site central (C4) e sites remotos a partir de modal LEO.

# 📡 Documentação de Implementação — Roteamento Baseado em Políticas (PBR)

## 🏛️ Arquitetura de Rede

### 📍 Site Central
![Diagrama Site Central](./diagrama_site_central.png)

### 📍 Site Remoto
![Diagrama Site Remoto](./diagrama_site_remoto.png)

---

## 🧭 Definições de Roteamento Baseado em Políticas (PBR)

### 🎯 Tráfego Administrativo

**Com destino ao Site Central:**
1. 🔗 S2S  
2. 🌐 MPLS  
3. 🛰️ Satélite  

**Com destino aos Demais Sites:**
1. 🌐 MPLS  
2. 🔗 S2S  
3. 🛰️ Satélite  

---

### 🌐 Tráfego Operacional (Web, não V-200)

1. 🔗 S2S  
2. 🌐 MPLS  
3. 🛰️ Satélite  

---

### 📡 Tráfego SBT

1. 🛰️ Satélite  
2. 🔌 TOPNET  
3. 🔗 S2S  
4. 🌐 MPLS  

---

## ⚙️ Etapas de Implementação

### 🏛️ No Site Central

#### 🔄 Aplicar no grupo **GLBP**

#### 📋 Criar `PREFIX-LIST` com a rede remota (.87 ou .88)
~~~bash
ip prefix-list S2S_<sigla do site> seq 20 permit 10.115.98.24/29
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
 icmp-echo <ip do S2S no site remoto> source-ip 10.112.82.211
 tag Monitoracao S2S <designativo do site remoto>
 frequency 5

ip sla schedule <mesmo id acima> life forever start-time now
~~~

---

### 🛰️ No Site Remoto

#### 🚪 Se possuir porta disponível no roteador de borda:
~~~bash
interface GigabitEthernet?/?
 description S2S
 ip address <ip interface local S2S, disponível no PHPIPHAM> 255.255.255.248
~~~

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
 set ip next-hop verify-availability <ip CPE MPLS> 2 track 3
 set ip next-hop verify-availability <ip CPE S2S> 1 track 4
 set ip next-hop <ip TUNEL remoto>

route-map sbt permit 10
 match ip address SBT
 set ip next-hop verify-availability <ip TUNEL remoto> 1 track 1
 set ip next-hop verify-availability <ip TOPNET remoto> 2 track 2
 set ip next-hop verify-availability <ip CPE S2S> 1 track 4
 set ip next-hop verify-availability <ip CPE MPLS> 2 track 3

route-map operacional permit 10
 match ip address LAN_OPR
 set ip next-hop verify-availability <ip CPE S2S> 1 track 4
 set ip next-hop verify-availability <ip CPE MPLS> 2 track 3
 set ip next-hop verify-availability <ip TUNEL remoto> 1 track 1
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

## 🧩 Observações Finais

- 🔐 **Endereços IP e áreas OSPF devem ser obtidos do sistema PHPIPHAM oficial.**
- 🔧 Adapte os nomes das listas de acesso, route-maps e tracks conforme a topologia local.
- 🧠 Faça backup da configuração antes de aplicar qualquer mudança.
- 📎 Para testes, utilize `debug ip policy` e `show route-map`.

---

✅ **Para dúvidas, melhorias ou contribuições, abra uma *issue* ou envie um *pull request*.**
