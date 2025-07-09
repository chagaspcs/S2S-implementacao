#### 🔄 Aplicar no grupo **GLBP** (Roteadores 80.201 e 80.202)

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
