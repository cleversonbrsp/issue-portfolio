# ⚙️ Kubernetes – Timeout no Rollout de Deployment (Progress Deadline Exceeded)

## 🇧🇷 Português (BR)

**issue:**
Estamos enfrentando uma falha durante o processo de rollback do deployment app-prod-dp no namespace prod. O processo tenta reiniciar o deployment diversas vezes, mas falha em todas as tentativas devido ao erro deployment "app-prod-dp" exceeded its progress deadline.

```
Reiniciando deployment app-prod-dp com tentativas...
⚙️ Tentativa 1 de 10
deployment.apps/app-prod-dp restarted
Waiting for deployment "app-prod-dp" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "app-prod-dp" rollout to finish: 1 old replicas are pending termination...
error: deployment "app-prod-dp" exceeded its progress deadline
Error: Process completed with exit code 1.
```

**causa raiz:**
O Deployment tinha um PodDisruptionBudget com minAvailable: 2 configurado para 2 réplicas — ou seja, exigia que as duas réplicas antigas continuassem disponíveis durante todo o rollout. Isso impedia o Kubernetes de finalizar o pod antigo para dar lugar ao novo, deixando o rollout preso aguardando uma substituição que nunca podia acontecer, até estourar o progressDeadlineSeconds e o pipeline falhar.

**solução:**
Ajustei o Deployment e o PodDisruptionBudget porque o pipeline faz um rollout restart automático e o Kubernetes precisa de regras claras para conseguir trocar os pods sem travar o processo. Adicionei o progressDeadlineSeconds: 600 (10 minutos) no Deployment para deixar explícito o tempo máximo que o Kubernetes pode levar para concluir o rollout. Também alterei o PodDisruptionBudget para minAvailable: 1 — porque, com duas réplicas, manter minAvailable: 2 impedia o Kubernetes de finalizar o pod antigo durante o rollout. Também ajustei o pipeline para aguardar explicitamente o rollout status do deployment, em vez de depender de tempo fixo ou tentativas cegas.

---

## 🇺🇸 English

**issue:**
We were facing a failure during the rollback process of the app-prod-dp deployment in the prod namespace. The process tried to restart the deployment multiple times, but failed on every attempt with the error: deployment "app-prod-dp" exceeded its progress deadline.

```
Reiniciando deployment app-prod-dp com tentativas...
⚙️ Tentativa 1 de 10
deployment.apps/app-prod-dp restarted
Waiting for deployment "app-prod-dp" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "app-prod-dp" rollout to finish: 1 old replicas are pending termination...
error: deployment "app-prod-dp" exceeded its progress deadline
Error: Process completed with exit code 1.
```

**root cause:**
The Deployment had a PodDisruptionBudget with minAvailable: 2 set for 2 replicas — meaning both old replicas had to stay available throughout the entire rollout. This prevented Kubernetes from retiring the old pod to make room for the new one, leaving the rollout stuck waiting for a replacement that could never happen, until progressDeadlineSeconds was exceeded and the pipeline failed.

**solution:**
I adjusted the Deployment and the PodDisruptionBudget because the pipeline runs an automatic rollout restart, and Kubernetes needs clear rules to be able to swap pods without getting stuck. I added progressDeadlineSeconds: 600 (10 minutes) to the Deployment to make explicit the maximum time Kubernetes can take to complete the rollout. I also changed the PodDisruptionBudget to minAvailable: 1 — because, with two replicas, keeping minAvailable: 2 prevented Kubernetes from finishing off the old pod during the rollout. I also updated the pipeline to explicitly wait for the deployment's rollout status, instead of relying on fixed time windows or blind retries.

---

## 🇪🇸 Español

**issue:**
Estábamos enfrentando una falla durante el proceso de rollback del deployment app-prod-dp en el namespace prod. El proceso intentaba reiniciar el deployment varias veces, pero fallaba en todos los intentos con el error: deployment "app-prod-dp" exceeded its progress deadline.

```
Reiniciando deployment app-prod-dp com tentativas...
⚙️ Tentativa 1 de 10
deployment.apps/app-prod-dp restarted
Waiting for deployment "app-prod-dp" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "app-prod-dp" rollout to finish: 1 old replicas are pending termination...
error: deployment "app-prod-dp" exceeded its progress deadline
Error: Process completed with exit code 1.
```

**causa raíz:**
El Deployment tenía un PodDisruptionBudget con minAvailable: 2 configurado para 2 réplicas — es decir, exigía que ambas réplicas antiguas permanecieran disponibles durante todo el rollout. Esto impedía que Kubernetes retirara el pod antiguo para dar paso al nuevo, dejando el rollout atascado esperando un reemplazo que nunca podía ocurrir, hasta agotar el progressDeadlineSeconds y que el pipeline fallara.

**solución:**
Ajusté el Deployment y el PodDisruptionBudget porque el pipeline ejecuta un rollout restart automático y Kubernetes necesita reglas claras para poder reemplazar los pods sin bloquear el proceso. Agregué progressDeadlineSeconds: 600 (10 minutos) en el Deployment para dejar explícito el tiempo máximo que Kubernetes puede tardar en completar el rollout. También cambié el PodDisruptionBudget a minAvailable: 1, porque con dos réplicas, mantener minAvailable: 2 impedía que Kubernetes finalizara el pod antiguo durante el rollout. También ajusté el pipeline para esperar explícitamente el rollout status del deployment, en vez de depender de tiempos fijos o intentos ciegos.
