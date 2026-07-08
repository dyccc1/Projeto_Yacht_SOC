# Superyacht SOC Simulation: Protecting Maritime Infrastructure

## Contexto do Projeto
Como ex-tripulante de iates, compreendo que a rede de um superiate é um ambiente crítico e único: passageiros UHNWI exigem privacidade total , enquanto sistemas de navegação (ECDIS) não podem ser comprometidos por acessos indevidos da tripulação ou malware.

Este projeto simula o ecossistema de segurança de um iate de luxo, utilizando **Wazuh SIEM**, **pfSense Firewall** e **Endpoint Hardening** para garantir a integridade da ponte de comando e a proteção de dados a bordo.

## Topologia da Rede (Simulação Marítima)
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
  
### Documentação Completa
 relatório técnico detalhado com todos os passos, logs e prints aqui:
   Descarregar Relatório Técnico - Volume 1 (PDF): [relatorio_tecnico_vol1.pdf](https://github.com/user-attachments/files/29817418/relatorio_tecnico_vol1.pdf)

  
### Próximos volumes: 

- Implementação de Redundância WAN (Failover Starlink/4G).
- VPN Site-to-Site via WireGuard para ligação Iate-Escritório.
- Implementação de Acesso Remoto Seguro com Apache Guacamole.
