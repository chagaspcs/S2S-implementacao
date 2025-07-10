O projeto S2S provê conectividade entre o site central (C4) e sites remotos a partir de modal LEO.

# 📡 Documentação de Implementação — Roteamento Baseado em Políticas (PBR)

## 🏛️ Arquitetura de Rede

### 📍 Site Central
[Diagrama Site Central](./diagrama_site_central.png)

### 📍 Site Remoto
[Diagrama Site Remoto](./diagrama_site_remoto.png)

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
[Configuração Site Central](./site_central.md)

### 🏛️ No Site Remoto
[Configuração Site Remoto](./site_remoto.md)

## 🧩 Observações Finais

- 🔐 **Endereços IP e áreas OSPF devem ser obtidos do sistema PHPIPHAM oficial.**
- 🔧 Adapte os nomes das listas de acesso, route-maps e tracks conforme a topologia local.
- 🧠 Faça backup da configuração antes de aplicar qualquer mudança.
- 📎 Para testes, utilize `debug ip policy` e `show route-map`.

---

✅ **Para dúvidas, melhorias ou contribuições, abra uma *issue* ou envie um *pull request*.**
