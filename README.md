# Superyacht SOC Simulation: Protecting Maritime Infrastructure

## Contexto do Projeto
Como ex-tripulante de iates, compreendo que a rede de um superiate é um ambiente crítico e único: passageiros UHNWI exigem privacidade total , enquanto sistemas de navegação (ECDIS) não podem ser comprometidos por acessos indevidos da tripulação ou malware.

Este projeto simula o ecossistema de segurança de um iate de luxo, utilizando **Wazuh SIEM**, **pfSense Firewall** e **Endpoint Hardening** para garantir a integridade da ponte de comando e a proteção de dados a bordo.

### Documentação Completa
 relatório técnico detalhado com todos os passos, logs e prints aqui:
   Descarregar Relatório Técnico - Volume 1 (PDF): [relatorio_tecnico_vol1.pdf](https://github.com/user-attachments/files/29817418/relatorio_tecnico_vol1.pdf)
   Descarregar Relatório Técnico - Volume 2 (PDF):

## Topologia da Rede 
<img width="580" height="521" alt="Diagrama sem nome drawio (1)" src="https://github.com/user-attachments/assets/ea811379-4c12-4dc4-8bc1-6912eb321c1e" />


# Security Implementation (Volume 1)

### 1. Hardening de Firewall (pfSense)
Implementei políticas de **Micro-segmentação** utilizando Aliases para simplificar a gestão de regras. 
  
<img width="1040" height="681" alt="image" src="https://github.com/user-attachments/assets/5ea77d69-0e25-446e-8fd6-0b0e54b976aa" />

*   **Política Crítica:** Bloqueio explícito de tráfego entre a rede da tripulação (CREW) e a rede de navegação (BRIDGE) com logging ativo para auditoria no SIEM.

<img width="705" height="682" alt="image" src="https://github.com/user-attachments/assets/8e5206ff-1445-46c9-bc10-9c4775eaa22b" />

### Zonas de Segurança Implementadas:
*   **VLAN 40 (AV CONTROL):** Sistemas de Entretenimento
*   **VLAN 30 (Bridge/ECDIS):** Zona isolada para sistemas críticos de navegação. Monitorizada por Agente Wazuh e Sysmon.
*   **VLAN 20 (Crew):** Rede segmentada com regras de firewall que impedem o movimento lateral para a ponte.
*   **VLAN 10 (guest):** Acesso apenas Internet
*   **WAN (VSAT Link):** Simulação de ligação por satélite monitorizada por Syslog.

  
### 2. Inteligência de Deteção (Wazuh SIEM)
Desenvolvi uma **Hierarquia de Regras (Parent/Child)** para identificar a origem exata de tentativas de invasão na rede. 

```xml
<group name="yacht_cyber_security,">
 
  <rule id="100030" level="10">
    <if_sid>87701</if_sid>
    <dstip>10.0.30.0/24</dstip>
    <match>22|3389</match>
    <description>pfSense: Tentativa de acesso aos sistemas da PONTE detetada.</description>
    <mitre><id>T1021</id></mitre>
  </rule>

  
  <rule id="100031" level="12">
    <if_matched_sid>100030</if_matched_sid>
    <srcip>10.0.10.0/24</srcip>
    <description>ALERTA CRÍTICO: Tentativa de invasão dos GUESTS à PONTE!</description>
  </rule>

  
  <rule id="100032" level="12">
    <if_matched_sid>100030</if_matched_sid>
    <srcip>10.0.20.0/24</srcip>
    <description>ALERTA CRÍTICO: Tentativa de invasão da TRIPULAÇÃO à PONTE!</description>
  </rule>

  
  <rule id="100033" level="12">
    <if_matched_sid>100030</if_matched_sid>
    <srcip>10.0.40.0/24</srcip>
    <description>ALERTA CRÍTICO: Dispositivo de ENTRETENIMENTO (AV) a tentar aceder à PONTE!</description>
  </rule>
</group>
 ```

### Troubleshooting & Lessons Learned 

Este laboratório proporcionou desafios técnicos que exigiram diagnósticos profundos como:

- Conflito de Tags 802.1Q: Identifiquei uma falha de conetividade após a ativação das VLANs, resolvi através da configuração da interface LAN via CLI para aceitar tráfego de gestão untagged.
- Análise de Sintaxe XML: Superei erros de mapeamento de IPs no motor de análise do Wazuh, ajustando as regras para a notação CIDR correta.
  

# Redundância WAN (volume 2):

### Alta Disponibilidade em Alto Mar
Em operações marítimas , a internet não é apenas um serviço, é uma utilidade crítica. O link de satélite (**Starlink**) pode sofrer interrupções devido a condições atmosféricas ou localização geográfica. Para garantir que o iate nunca fique offline, implementei um sistema de **Multi-WAN Failover** no pfSense.

### Configuração da Infraestrutura
Para simular este ambiente, configurei o firewall com dois gateways redundantes:
*   **WAN 1 (Starlink):** Definido como o link principal (Tier 1).
*   **WAN 2 (4G/5G Backup):** Definido como link de reserva (Tier 2).

### Implementação Técnica
1.  **Gateway Groups:** Criei um grupo de balanceamento chamado `yacht_internet_failover´.
2.  **Mecanismo de Trigger:** O failover é ativado automaticamente pelo gatilho **"Member Down"**. O pfSense monitoriza continuamente a perda de pacotes e a latência do Starlink.
3.  **Policy Routing:** Forcei o tráfego das VLANs críticas a utilizar este grupo através das regras de firewall. 


### Prova de Conceito (PoC):
<img width="1096" height="384" alt="image" src="https://github.com/user-attachments/assets/4cad3727-cc9e-40fd-ae9b-c221430cc7a0" />
<img width="1021" height="663" alt="image" src="https://github.com/user-attachments/assets/86c8b1ad-e0fd-4dbd-acc0-5e3219d7dc7d" />

### Liçoes aprendidas:

* **Monitorização de IP Externo:** Aprendi que, para um failover eficaz, o firewall deve monitorizar um IP externo (como o `8.8.8.8`) e não apenas o Gateway local, para detetar falhas reais na internet e não apenas "cabos desligados".
* **Persistência de Sessão:** A configuração correta garantiu que, durante a troca de link, as comunicações críticas de telemetria para o **Wazuh** não fossem interrompidas, mantendo a visibilidade de segurança 24/7.

### Próximos volumes: 
- VPN Site-to-Site via WireGuard para ligação Iate-Escritório.
- Implementação de Acesso Remoto Seguro com Apache Guacamole.
