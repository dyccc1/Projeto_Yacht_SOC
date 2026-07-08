# Superyacht SOC Simulation: Protecting Maritime Infrastructure

## Contexto do Projeto
Como ex-tripulante de iates, compreendo que a rede de um superiate é um ambiente crítico e único: passageiros UHNWI exigem privacidade total , enquanto sistemas de navegação (ECDIS) não podem ser comprometidos por acessos indevidos da tripulação ou malware.

Este projeto simula o ecossistema de segurança de um iate de luxo, utilizando **Wazuh SIEM**, **pfSense Firewall** e **Endpoint Hardening** para garantir a integridade da ponte de comando e a proteção de dados a bordo.

## Topologia da Rede (Simulação Marítima)
<img width="580" height="521" alt="Diagrama sem nome drawio (1)" src="https://github.com/user-attachments/assets/ea811379-4c12-4dc4-8bc1-6912eb321c1e" />




### Zonas de Segurança Implementadas:
*   **VLAN 40 (AV CONTROL):** Sistemas de Entretenimento
*   **VLAN 30 (Bridge/ECDIS):** Zona isolada para sistemas críticos de navegação. Monitorizada por Agente Wazuh e Sysmon.
*   **VLAN 20 (Crew):** Rede segmentada com regras de firewall que impedem o movimento lateral para a ponte.
*   **VLAN 10 (guest):** Acesso apenas Internet
*   **WAN (VSAT Link):** Simulação de ligação por satélite monitorizada por Syslog.
