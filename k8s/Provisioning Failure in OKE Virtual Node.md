# ☸️ OKE – Falha de Provisionamento em Virtual Node (Capacity Constraint)

## 🇧🇷 Português (BR)

**issue:**
Deployments rodando em Virtual Nodes do OKE falhavam intermitentemente ao provisionar novos Pods durante rollouts (por exemplo, após uma nova revisão de imagem), por falta temporária de capacidade na infraestrutura da OCI (região/availability domain). O erro observado era: "cannot provision pod due to insufficient capacity". Como o novo Pod nunca subia, o rollout ficava travado e eventualmente falhava com "progress deadline exceeded" — mesmo com a aplicação e a imagem corretas.

**causa raiz:**
A falha não era causada pela aplicação, mas pelo provisionamento dinâmico de capacidade dos Virtual Nodes do OKE: no momento do rollout, o fault domain/availability domain escolhido pelo scheduler estava temporariamente sem capacidade disponível. A orientação da Oracle também apontou que o shape A4 estava depreciado (recomendando o A6), e que especificar um fault domain explícito na criação do virtual node pool reduzia a flexibilidade de agendamento, tornando o problema mais provável.

**solução:**
Foi adicionado um workaround no pipeline: fazer cordon e isolar o rollout em um único node, e fazer retry apenas em falhas transitórias de provisionamento (nunca para erros de aplicação/configuração) — cada retry dispara um novo ciclo de agendamento que pode cair em capacidade disponível em outro lugar. Foi feita escalação com a Oracle, que recomendou deixar o fault domain vazio na criação dos virtual node pools, permitindo que os pods sejam agendados em qualquer fault domain com capacidade. Combinado com a lógica de retry/isolamento do pipeline, isso resolveu as falhas intermitentes.

---

## 🇺🇸 English

**issue:**
Deployments running on OKE Virtual Nodes intermittently failed to provision new Pods during rollouts (e.g., after a new image revision), due to temporary lack of capacity in the underlying OCI infrastructure (region/availability domain). The error observed was: "cannot provision pod due to insufficient capacity". Since the new Pod never started, the rollout got stuck and eventually failed with "progress deadline exceeded" — even though the application and image were fine.

**root cause:**
The failure was not caused by the application, but by OKE Virtual Nodes' dynamic capacity provisioning: at rollout time, the fault domain/availability domain the scheduler picked temporarily had no capacity available. Oracle's guidance also flagged that shape A4 was deprecated (A6 recommended), and that specifying a fault domain explicitly at virtual node pool creation reduced scheduling flexibility, making the problem more likely.

**solution:**
Added a pipeline workaround: cordon and isolate the rollout to a single node, and retry only on transient provisioning failures (never for application/config errors) — each retry triggers a new scheduling cycle that can land on available capacity elsewhere. Escalated with Oracle, who recommended leaving the fault domain empty when creating virtual node pools, letting pods be scheduled across any fault domain with capacity. Combined with the pipeline retry/isolation logic, this resolved the intermittent failures.

---

## 🇪🇸 Español

**issue:**
Los Deployments que corrían en Virtual Nodes de OKE fallaban intermitentemente al aprovisionar nuevos Pods durante los rollouts (por ejemplo, tras una nueva revisión de imagen), debido a la falta temporal de capacidad en la infraestructura de OCI (región/availability domain). El error observado era: "cannot provision pod due to insufficient capacity". Como el nuevo Pod nunca llegaba a iniciar, el rollout quedaba bloqueado y finalmente fallaba con "progress deadline exceeded" — aunque la aplicación y la imagen estaban correctas.

**causa raíz:**
La falla no era causada por la aplicación, sino por el aprovisionamiento dinámico de capacidad de los Virtual Nodes de OKE: en el momento del rollout, el fault domain/availability domain elegido por el scheduler no tenía capacidad disponible temporalmente. La orientación de Oracle también señaló que el shape A4 estaba obsoleto (se recomendaba A6), y que especificar un fault domain explícito al crear el virtual node pool reducía la flexibilidad de programación, haciendo el problema más probable.

**solución:**
Se agregó un workaround en el pipeline: hacer cordon y aislar el rollout a un único nodo, y reintentar solo ante fallas transitorias de aprovisionamiento (nunca para errores de aplicación/configuración) — cada reintento dispara un nuevo ciclo de programación que puede caer en capacidad disponible en otro lugar. Se escaló con Oracle, quien recomendó dejar el fault domain vacío al crear los virtual node pools, permitiendo que los pods se programen en cualquier fault domain con capacidad. Combinado con la lógica de retry/aislamiento del pipeline, esto resolvió las fallas intermitentes.
