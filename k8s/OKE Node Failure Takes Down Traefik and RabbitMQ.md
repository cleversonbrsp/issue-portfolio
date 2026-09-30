# 🖥️ OKE – Queda de Node Derruba Traefik e RabbitMQ

## 🇧🇷 Português (BR)

**issue:**
Um node do cluster OKE prod-customer (10.0.0.10) caiu abruptamente, derrubando de uma só vez as duas réplicas do Traefik (ingress) e o pod do RabbitMQ que rodavam nele, gerando um impacto em cascata: outage de ingress de ~8 minutos, RabbitMQ indisponível por ~23 minutos, e o serviço async-worker-dp reiniciando 6 vezes por depender do RabbitMQ. A recuperação foi automática via self-healing do Kubernetes e cluster-autoscaler, sem necessidade de ação manual.

**causa raiz:**
Falha abrupta da instância de compute do node (zero heartbeats de kubelet durante toda a janela, sem drain gracioso e sem ação manual de terminação). O fator que agravou o incidente foi o Deployment do Traefik rodar com apenas 2 réplicas sem podAntiAffinity/topologySpreadConstraint configurado — nada impedia as duas réplicas de caírem no mesmo node, o que transformou a perda de 1 node em um outage completo de ingress em vez de uma degradação parcial. Aberto chamado no Oracle Support, que confirmou que a instância foi terminada automaticamente pelo protocolo padrão do OKE ao detectar o host sem resposta (self-healing), sem causa de infraestrutura identificável do lado deles.

**solução:**
A recuperação foi automática (Kubernetes + cluster-autoscaler), sem intervenção manual durante o incidente. Como ação preventiva, foi recomendada a adição de podAntiAffinity (ou topologySpreadConstraint) no Deployment do Traefik em prod-customer, para impedir que as réplicas caiam no mesmo node e evitar que uma futura perda de node cause outage completo de ingress novamente.

---

## 🇺🇸 English

**issue:**
A node in the prod-customer OKE cluster (10.0.0.10) failed abruptly, taking down both Traefik (ingress) replicas and the RabbitMQ pod running on it at once, causing a cascading impact: an ~8-minute ingress outage, RabbitMQ unavailable for ~23 minutes, and the async-worker-dp service restarting 6 times due to its RabbitMQ dependency. Recovery was fully automatic via Kubernetes self-healing and the cluster-autoscaler, with no manual action required.

**root cause:**
An abrupt compute instance failure (zero kubelet heartbeats throughout the whole window, no graceful drain, no manual termination action). What made the incident worse was that the Traefik Deployment ran with only 2 replicas and no podAntiAffinity/topologySpreadConstraint — nothing prevented both replicas from landing on the same node, which turned the loss of a single node into a full ingress outage instead of a partial degradation. A support case was opened with Oracle, who confirmed the instance was automatically terminated by OKE's standard self-healing protocol after detecting an unresponsive host, with no identifiable infrastructure-side root cause on their end.

**solution:**
Recovery was fully automatic (Kubernetes + cluster-autoscaler); no manual intervention was needed during the incident. As a preventive action, adding podAntiAffinity (or a topologySpreadConstraint) to the Traefik Deployment in prod-customer was recommended, to prevent both replicas from landing on the same node and avoid a full ingress outage on a future node loss.

---

## 🇪🇸 Español

**issue:**
Un nodo del clúster OKE prod-customer (10.0.0.10) falló abruptamente, tumbando de una sola vez las dos réplicas de Traefik (ingress) y el pod de RabbitMQ que corrían en él, generando un impacto en cascada: un outage de ingress de ~8 minutos, RabbitMQ no disponible durante ~23 minutos, y el servicio async-worker-dp reiniciando 6 veces por depender de RabbitMQ. La recuperación fue automática mediante self-healing de Kubernetes y el cluster-autoscaler, sin necesidad de intervención manual.

**causa raíz:**
Falla abrupta de la instancia de cómputo del nodo (cero heartbeats de kubelet durante toda la ventana, sin drain elegante y sin acción manual de terminación). El factor que agravó el incidente fue que el Deployment de Traefik corría con solo 2 réplicas y sin podAntiAffinity/topologySpreadConstraint configurado — nada impedía que ambas réplicas cayeran en el mismo nodo, lo que convirtió la pérdida de 1 nodo en un outage completo de ingress en vez de una degradación parcial. Se abrió un caso con el soporte de Oracle, que confirmó que la instancia fue terminada automáticamente por el protocolo estándar de OKE al detectar el host sin respuesta (self-healing), sin causa de infraestructura identificable de su lado.

**solución:**
La recuperación fue automática (Kubernetes + cluster-autoscaler), sin intervención manual durante el incidente. Como acción preventiva, se recomendó agregar podAntiAffinity (o topologySpreadConstraint) al Deployment de Traefik en prod-customer, para impedir que las réplicas caigan en el mismo nodo y evitar que una futura pérdida de nodo cause un outage completo de ingress nuevamente.
