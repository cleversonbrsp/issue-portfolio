# ☕ Aplicação Java (HOM) – OOM Recorrente por Job de Relatório Travado

## 🇧🇷 Português (BR)

**issue:**
O ambiente de homologação app-hom apresentava instabilidade recorrente: erro 500 no fluxo de SSO, falhas 404 no job agendado de integração com SAP, e OutOfMemoryError (Java heap space) recorrente em um dos pods, afetando threads HTTP e o listener do RabbitMQ.

**causa raiz:**
Um job de relatório em lote (processamento de pagamento de um cliente) ficou travado desde vários dias antes, com registros presos em status "GENERATING"/"WAITING_QUEUE" e "PENDING"/"PROCESSING"/"PARTIAL" sem nunca finalizar. Esse processamento preso manteve pressão contínua sobre o heap da JVM, que estava configurado com -Xmx2048m — insuficiente frente a essa carga — gerando efeito cascata sobre outros fluxos da aplicação (SSO, integração SAP). O banco de dados foi descartado como causa após verificação (sem saturação de conexões, sem lock, sem query travada).

**solução:**
Os registros travados nas tabelas de negócio foram atualizados para status de erro, liberando o job. A memória do container foi aumentada de 6Gi para 8Gi, com ajuste correspondente de -Xms/-Xmx da JVM, e o pod foi reiniciado. Como ponto de processo, ficou definido que a remediação de dados de negócio (mudar status de job/registro) deve ser escalada ao time de desenvolvimento da aplicação em ocorrências futuras, mantendo devops restrito a diagnóstico e ajuste de infraestrutura.

---

## 🇺🇸 English

**issue:**
The app-hom staging environment showed recurring instability: 500 errors in the SSO login flow, recurring 404 failures in the scheduled SAP integration job, and a recurring OutOfMemoryError (Java heap space) on one of the pods, affecting HTTP threads and the RabbitMQ listener.

**root cause:**
A batch report job (a customer's payment processing batch) had been stuck for several days, with records stuck in "GENERATING"/"WAITING_QUEUE" and "PENDING"/"PROCESSING"/"PARTIAL" status, never completing. This stuck processing kept continuous pressure on the JVM heap, which was configured at -Xmx2048m — insufficient for that load — cascading into other application flows (SSO, SAP integration). The database was ruled out as the cause after verification (no connection saturation, no locks, no stuck queries).

**solution:**
The stuck records in the business tables were updated to an error status, releasing the job. Container memory was increased from 6Gi to 8Gi, with a corresponding JVM -Xms/-Xmx adjustment, and the pod was restarted. As a process improvement, it was established that business-data remediation (changing a job/record's status) should be escalated to the application's development team in future occurrences, keeping DevOps scoped to diagnostics and infrastructure adjustments.

---

## 🇪🇸 Español

**issue:**
El entorno de homologación app-hom presentaba inestabilidad recurrente: errores 500 en el flujo de SSO, fallas 404 recurrentes en el job programado de integración con SAP, y un OutOfMemoryError (Java heap space) recurrente en uno de los pods, afectando los threads HTTP y el listener de RabbitMQ.

**causa raíz:**
Un job de reporte por lotes (procesamiento de pago de un cliente) quedó atascado desde varios días antes, con registros atrapados en estado "GENERATING"/"WAITING_QUEUE" y "PENDING"/"PROCESSING"/"PARTIAL" sin finalizar nunca. Ese procesamiento atascado mantuvo presión continua sobre el heap de la JVM, configurado en -Xmx2048m — insuficiente para esa carga — generando un efecto cascada sobre otros flujos de la aplicación (SSO, integración SAP). La base de datos se descartó como causa tras la verificación (sin saturación de conexiones, sin locks, sin queries atascadas).

**solución:**
Los registros atascados en las tablas de negocio se actualizaron a estado de error, liberando el job. Se aumentó la memoria del contenedor de 6Gi a 8Gi, con el ajuste correspondiente de -Xms/-Xmx de la JVM, y se reinició el pod. Como mejora de proceso, se estableció que la remediación de datos de negocio (cambiar el estado de un job/registro) debe escalarse al equipo de desarrollo de la aplicación en futuras ocurrencias, manteniendo a DevOps limitado a diagnóstico y ajustes de infraestructura.
