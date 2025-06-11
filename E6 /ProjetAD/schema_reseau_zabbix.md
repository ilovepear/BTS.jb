
# Schéma Réseau – Projet Zabbix BTS SIO SISR

```
                     +----------------------+
                     |  Routeur / Box (NAT) |
                     |  IP : 192.168.114.1  |
                     +----------+-----------+
                                |
                                | Réseau local : 192.168.114.0/24
                                |
       +------------------------+------------------------+
       |                                                 |
+--------------------+                          +--------------------+
| SRV-ZAB-GUA        |                          | SRV-DC-GUA         |
| (Zabbix Server)    |                          | (Windows Server)   |
| Ubuntu Server 22.04|                          | Contrôleur AD + DNS|
| IP : 192.168.114.50|                          | IP : 192.168.114.88|
| Rôles :            |                          | Zabbix Agent actif |
| - Zabbix Server    |                          | Port : 10050 TCP   |
| - Apache / MariaDB |                          +--------------------+
+--------------------+

Zabbix Server ↔ Agent communication :
- Port 10050 : utilisé par SRV-DC-GUA pour recevoir les checks passifs
- Port 10051 : utilisé par Zabbix Server pour recevoir les checks actifs (optionnel)
```

## 🔐 Pare-feu à configurer :

- **SRV-DC-GUA** :
  - Autoriser **port 10050 TCP entrant**
- **SRV-ZAB-GUA** :
  - Aucun port entrant nécessaire pour l’agent, sauf si agent local
