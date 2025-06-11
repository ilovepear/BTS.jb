
# Schéma Réseau du Projet

```
                         Internet
                             |
                             | NAT (temporaire)
                             |
                 +-----------|-----------+
                 |                       |
                 |    Réseau Interne     |
                 |     192.168.10.0/24   |
                 |                       |
                 +-----------+-----------+
                             |
                             |
+-------------------------------------------------------------+
|                   Serveur Contrôleur AD                     |
|                  SRV-DC-GUA.hn.gua.local                    |
|                       192.168.10.1                          |
|                                                             |
| - Windows Server 2022                                       |
| - Active Directory Domain Services (AD DS)                  |
| - DNS Server (primaire)                                     |
| - DHCP Server                                               |
| - Group Policy Management                                   |
| - Windows Server Backup                                     |
| - UFW / Windows Defender Firewall                           |
+-------------------------------------------------------------+
                             |
                             |
                  +----------+-----------+
                  |                      |
                  |                      |
+---------------------------+   +-----------------------------+
|     Machine Ubuntu        |   |  Autres machines clientes   |
|      192.168.10.X         |   |        (Windows/Ubuntu)     |
|                           |   |        192.168.10.x         |
| - Carte réseau interne    |   |                             |
| - DNS : 192.168.10.1      |   | - DNS : 192.168.10.1        |
| - Jonction au domaine     |   | - Jonction au domaine       |
|   via SSSD, Realmd, ADCLI,|   | - Application des GPO       |
|   kinit                   |   |                             |
+---------------------------+   +-----------------------------+
