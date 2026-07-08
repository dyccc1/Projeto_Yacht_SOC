# Superyacht SOC Simulation: Protecting Maritime Infrastructure

## Contexto do Projeto
Como ex-tripulante de iates, compreendo que a rede de um superiate é um ambiente crítico e único: passageiros UHNWI exigem privacidade total , enquanto sistemas de navegação (ECDIS) não podem ser comprometidos por acessos indevidos da tripulação ou malware.

Este projeto simula o ecossistema de segurança de um iate de luxo, utilizando **Wazuh SIEM**, **pfSense Firewall** e **Endpoint Hardening** para garantir a integridade da ponte de comando e a proteção de dados a bordo.

## Topologia da Rede (Simulação Marítima)
[2f1cfdf8-39d2-47b0-acc1-9dc5eec3f253_Volume_1_Segmentao_Crtica.pdf](https://github.com/user-attachments/files/29816551/2f1cfdf8-39d2-47b0-acc1-9dc5eec3f253_Volume_1_Segmentao_Crtica.pdf)



### Zonas de Segurança Implementadas:
*   **VLAN 40 (AV CONTROL):** Sistemas de Entretenimento
*   **VLAN 30 (Bridge/ECDIS):** Zona isolada para sistemas críticos de navegação. Monitorizada por Agente Wazuh e Sysmon.
*   **VLAN 20 (Crew):** Rede segmentada com regras de firewall que impedem o movimento lateral para a ponte.
*   **VLAN 10 (guest):** Acesso apenas Internet
*   **WAN (VSAT Link):** Simulação de ligação por satélite monitorizada por Syslog.
