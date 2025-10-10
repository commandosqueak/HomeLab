# 🧭 LAN Address Plan — 192.168.8.0/24

| IP Address    | Hostname / Device | Static / DHCP | Ports / Services | Notes                    | Tags                         | Assigned ✅ | Link                 |
| ------------- | ----------------- | ------------- | ---------------- | ------------------------ | ---------------------------- | ---------- | -------------------- |
| 192.168.8.1   | OPNsense Router   | Static        | 443, 22          | Primary LAN gateway      | #core #infra #firewall       | [ ]        | [[OPNsense Config]]  |
| 192.168.8.2   | Proxmox Host      | Static        | 8006             | Hypervisor Mgmt          | #core #infra #hypervisor     | [ ]        | [[Proxmox Host]]     |
| 192.168.8.3   | Pi-hole           | Static        | 53, 80           | DNS & Ad-block           | #core #infra #dns #container | [ ]        | [[Pi-hole]]          |
| 192.168.8.4   | Portainer         | Static        | 9000             | Docker Mgmt              | #infra #container            | [ ]        | [[Portainer]]        |
| 192.168.8.5   | Uptime Kuma       | Static        | 3001             | Monitoring Dashboard     | #monitoring #container       | [ ]        | [[Uptime Kuma]]      |
| 192.168.8.6   | LanguageTool      | Static        | 8010             | Grammar Checker API      | #vm #service                 | [ ]        | [[LanguageTool]]     |
| 192.168.8.7   | WireGuard         | Static        | 51820            | VPN Endpoint             | #network #vpn #security      | [ ]        | [[WireGuard Config]] |
| 192.168.8.8   | Heimdall          | Static        | 80, 443          | App Dashboard            | #ui #portal #container       | [ ]        | [[Heimdall]]         |
| 192.168.8.9   | DuckDNS Updater   | Static        |                  | Dynamic DNS              | #infra #service              | [ ]        | [[DuckDNS]]          |
| 192.168.8.10  | TrueNAS           | Static        | 443, 22          | Storage/NAS              | #storage #core #vm           | [ ]        | [[TrueNAS]]          |
| 192.168.8.11  | Linkding          | Static        | 9090             | Bookmark Manager         | #container #webapp           | [ ]        | [[Linkding]]         |
| 192.168.8.12  | HomeBox           | Static        | 7745             | Inventory / Assets       | #container #productivity     | [ ]        | [[HomeBox]]          |
| 192.168.8.13  | Ollama LLM        | Static        | 11434            | Local AI Server          | #ai #vm #service             | [ ]        | [[Ollama]]           |
| 192.168.8.14  | Vaultwarden       | Static        | 8080             | Password Manager         | #security #container         | [ ]        | [[Vaultwarden]]      |
| 192.168.8.15  | Plex Server       | Static        | 32400            | Media Streaming          | #media #vm #server           | [ ]        | [[Plex Server]]      |
| 192.168.8.16  | Ansible Node      | Static        | 22               | Automation / Config Mgmt | #automation #infra           | [ ]        | [[Ansible]]          |
| 192.168.8.17  | phpIPAM           | Static        | 80, 443          | IP Address Mgmt          | #infra #network #database    | [ ]        | [[phpIPAM]]          |
| 192.168.8.18  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.19  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.20  | SqueakStation     | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.21  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.22  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.23  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.24  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.25  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.26  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.27  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.28  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.29  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.30  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.31  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.32  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.33  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.34  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.35  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.36  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.37  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.38  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.39  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.40  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.41  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.42  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.43  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.44  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.45  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.46  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.47  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.48  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.49  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.50  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.51  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.52  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.53  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.54  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.55  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.56  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.57  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.58  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.59  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.60  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.61  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.62  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.63  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.64  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.65  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.66  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.67  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.68  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.69  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.70  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.71  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.72  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.73  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.74  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.75  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.76  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.77  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.78  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.79  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.80  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.81  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.82  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.83  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.84  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.85  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.86  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.87  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.88  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.89  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.90  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.91  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.92  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.93  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.94  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.95  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.96  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.97  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.98  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.99  |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.100 | DHCP Start        | DHCP          |                  | Pool Start               | #dhcp                        |            |                      |
| 192.168.8.101 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.102 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.103 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.104 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.105 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.106 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.107 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.108 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.109 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.110 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.111 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.112 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.113 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.114 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.115 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.116 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.117 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.118 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.119 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.120 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.121 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.122 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.123 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.124 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.125 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.126 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.127 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.128 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.129 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.130 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.131 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.132 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.133 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.134 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.135 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.136 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.137 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.138 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.139 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.140 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.141 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.142 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.143 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.144 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.145 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.146 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.147 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.148 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.149 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.150 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.151 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.152 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.153 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.154 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.155 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.156 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.157 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.158 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.159 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.160 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.161 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.162 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.163 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.164 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.165 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.166 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.167 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.168 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.169 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.170 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.171 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.172 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.173 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.174 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.175 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.176 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.177 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.178 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.179 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.180 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.181 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.182 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.183 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.184 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.185 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.186 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.187 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.188 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.189 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.190 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.191 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.192 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.193 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.194 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.195 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.196 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.197 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.198 |                   | DHCP          |                  |                          | #dhcp                        |            |                      |
| 192.168.8.199 | DHCP End          | DHCP          |                  | Pool End                 | #dhcp                        |            |                      |
| 192.168.8.200 | Home Assistant    | Static        | 8123             | Automation Hub           | #automation #core #vm        | [ ]        | [[Home Assistant]]   |
| 192.168.8.201 | Zigbee Bridge     | Static        |                  | Zigbee2MQTT / ZHA Bridge | #automation #iot             | [ ]        | [[Zigbee Bridge]]    |
| 192.168.8.202 | ESPHome Node 1    | Static        |                  | Temperature Sensor       | #automation #iot             | [ ]        | [[ESPHome Node 1]]   |
| 192.168.8.203 | ESPHome Node 2    | Static        |                  | Future sensor            | #automation #iot             | [ ]        |                      |
| 192.168.8.204 | Camera 1          | Static        | 554              | Indoor Camera            | #iot #camera                 | [ ]        |                      |
| 192.168.8.205 | Camera 2          | Static        | 554              | Outdoor Camera           | #iot #camera                 | [ ]        |                      |
| 192.168.8.206 | RubyCam           | Static        | 554              | CatCam                   | #iot #camera #fun            | [ ]        | [[RubyCam]]          |
| 192.168.8.207 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.208 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.209 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.210 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.211 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.212 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.213 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.214 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.215 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.216 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.217 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.218 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.219 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.220 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.221 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.222 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.223 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.224 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.225 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.226 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.227 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.228 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.229 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.230 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.231 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.232 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.233 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.234 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.235 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.236 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.237 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.238 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.239 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.240 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.241 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.242 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.243 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.244 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.245 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.246 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.247 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.248 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.249 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.250 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.251 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.252 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.253 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.254 |                   | Static        |                  |                          | #reserved                    |            |                      |
| 192.168.8.255 | Broadcast | — | — | — | #system |  |  |
---

### 🗝️ Legend
- ✅ **Assigned** → tick `[x]` once configured & confirmed active  
- **Tags** → use consistent tags for filtering across Pi-hole, Kuma, etc.  
- **Ports / Services** → major mgmt or access ports for quick recall  
- **Link** → Obsidian note link or external dashboard (`[[Proxmox Host]]`, `[Web](https://proxmox.home.lab:8006)`)

### 🧩 Tag Scheme
| Tag | Meaning |
|------|---------|
| #core | Essential infrastructure (router, DNS, hypervisor) |
| #infra | Backend / system-level services |
| #vm | Virtual machine |
| #container | Docker / LXC container |
| #automation | Home Assistant, Ansible, etc. |
| #iot | Smart home / embedded devices |
| #security | VPNs, password managers |
| #media | Plex, streaming |
| #monitoring | Uptime Kuma, metrics |
| #ai | LLM / inference services |
| #reserved | Future expansion |
| #dhcp | Dynamic lease range |

