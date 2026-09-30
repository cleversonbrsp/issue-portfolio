# 📢 Zabbix × MS Teams – Alertas Param de Funcionar com Descontinuação de Webhooks

## 🇧🇷 Português (BR)

**issue:**
Os alertas do Zabbix enviados para o Microsoft Teams pararam de funcionar completamente, deixando de notificar incidentes críticos de disponibilidade e infraestrutura (aplicações, infraestrutura e Oracle Cloud).

**causa raiz:**
A Microsoft descontinuou os conectores/webhooks convencionais do Office 365 no Teams, substituindo-os por um novo modelo baseado em "fluxos de trabalho" (Power Automate). Todas as integrações do Zabbix que dependiam do webhook antigo simplesmente pararam de funcionar quando a Microsoft desativou o mecanismo legado.

**solução:**
Todos os media types de alerta do Zabbix foram migrados para o novo template oficial "MS Teams Workflow webhook", recriando a mesma estrutura lógica anterior (canais separados por aplicação e ambiente (Hom/Prod) e para alertas de infraestrutura/Oracle Cloud), cada um com sua própria URL de fluxo de trabalho gerada no Power Automate/Teams e vinculada ao usuário admin do Zabbix. A documentação do processo foi registrada em wiki interna para consulta futura.

---

## 🇺🇸 English

**issue:**
Zabbix alerts sent to Microsoft Teams completely stopped working, leaving critical availability and infrastructure incidents (aplicações, infraestrutura e Oracle Cloud) without notifications.

**root cause:**
Microsoft retired the conventional Office 365 connectors/webhooks in Teams, replacing them with a new "workflow" model based on Power Automate. Every Zabbix integration relying on the old webhook simply stopped working once Microsoft disabled the legacy mechanism.

**solution:**
All Zabbix alert media types were migrated to the new official "MS Teams Workflow webhook" template, recreating the same previous logical structure (separate channels per application and environment (Hom/Prod), plus infrastructure/Oracle Cloud alerts), each with its own workflow URL generated in Power Automate/Teams and linked to the Zabbix admin user. The process was documented in an internal wiki for future reference.

---

## 🇪🇸 Español

**issue:**
Las alertas de Zabbix enviadas a Microsoft Teams dejaron de funcionar por completo, dejando sin notificación incidentes críticos de disponibilidad e infraestructura (aplicações, infraestrutura e Oracle Cloud).

**causa raíz:**
Microsoft retiró los conectores/webhooks convencionales de Office 365 en Teams, reemplazándolos por un nuevo modelo de "flujos de trabajo" basado en Power Automate. Todas las integraciones de Zabbix que dependían del webhook antiguo simplemente dejaron de funcionar cuando Microsoft desactivó el mecanismo legado.

**solución:**
Todos los media types de alerta de Zabbix se migraron a la nueva plantilla oficial "MS Teams Workflow webhook", recreando la misma estructura lógica anterior (canales separados por aplicación y entorno (Hom/Prod), y alertas de infraestructura/Oracle Cloud), cada uno con su propia URL de flujo de trabajo generada en Power Automate/Teams y vinculada al usuario admin de Zabbix. El proceso se documentó en una wiki interna para referencia futura.
