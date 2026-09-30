# 🧾 Aplicação Java (PROD) – Disco Efêmero Esgotado por Relatório Gigante Reentregue pela Fila

## 🇧🇷 Português (BR)

**issue:**
Os jobs agendados de uma aplicação Java em produção passaram a retornar 502 (Bad Gateway) por cerca de 5 horas. A rota dos jobs era atendida por um único pod worker assíncrono. Nos logs, o worker registrou java.io.IOException: No space left on device durante a geração de um relatório e, poucos segundos depois, o health check de disco passou a indicar cerca de 20 KB livres. O pod parou de escrever logs, mas seguiu Ready e sem restarts, porque o Kubernetes não detecta disco efêmero cheio e continuou enviando tráfego a um pod inoperante.

**causa raiz:**
Um único relatório de uso extremamente grande (horas de geração, contra menos de 40 minutos nos demais relatórios dos 14 dias anteriores) gravava arquivos temporários no disco efêmero do pod, que não tinha limite de ephemeral-storage nem volume dedicado. Como a mensagem da fila só é confirmada ao final do processamento, cada restart do pod (um deploy e o restart diário agendado) fez o broker reentregar a mensagem e a geração recomeçou do zero: foram três tentativas ao longo de cerca de 14 horas, e a última rodou aproximadamente 5h37 até esgotar o disco. Agravantes: no código, o arquivo temporário do caminho principal só era removido em caso de sucesso (sem bloco finally), outros caminhos dependiam de deleteOnExit (só libera quando a JVM encerra) e o worker tinha réplica única, sem alerta de disco nem liveness considerando o health de disco. Foram descartados: saturação do banco de dados e relatório preso no banco sendo reprocessado por um job de recuperação (nenhum registro preso; o mecanismo foi a reentrega da fila).

**solução:**
O deployment do worker foi reiniciado, recriando o pod com disco limpo e restabelecendo a rota; o relatório já estava finalizado como erro e não foi reentregue novamente, e os relatórios seguintes foram processados normalmente, sem novo erro de disco. Como correção definitiva, foram recomendados: limite de tempo e de tamanho para a geração de relatórios, reentrega controlada da fila em vez de ilimitada, remoção dos arquivos temporários em bloco finally, limite de ephemeral-storage com alerta de disco, liveness considerando o health de disco e mais de uma réplica do worker. O histórico de status do relatório no banco (três transições para "gerando", uma após cada restart) foi a evidência que permitiu identificar a reentrega.

---

## 🇺🇸 English

**issue:**
The scheduled jobs of a production Java application started returning 502 (Bad Gateway) for about 5 hours. The jobs route was served by a single asynchronous worker pod. In the logs, the worker recorded java.io.IOException: No space left on device while generating a report and, a few seconds later, the disk health check reported about 20 KB free. The pod stopped writing logs but stayed Ready with no restarts, because Kubernetes does not detect a full ephemeral disk and kept sending traffic to a dead pod.

**root cause:**
A single, extremely large usage report (hours of generation, versus under 40 minutes for every other report in the previous 14 days) wrote temporary files to the pod's ephemeral disk, which had no ephemeral-storage limit and no dedicated volume. Because the queue message is only acknowledged at the end of processing, each pod restart (a deployment and the scheduled daily restart) made the broker redeliver the message and generation restarted from scratch: three attempts over roughly 14 hours, the last one running about 5h37 until the disk was exhausted. Aggravating factors: in the code, the temporary file on the main path was only deleted on success (no finally block), other paths relied on deleteOnExit (which only releases files when the JVM exits), and the worker ran as a single replica with no disk alert and no liveness check based on disk health. Ruled out: database saturation and a report stuck in the database being reprocessed by a recovery job (no stuck records; the mechanism was queue redelivery).

**solution:**
The worker deployment was restarted, recreating the pod with a clean disk and restoring the route; the report had already been finalized as an error and was not redelivered again, and later reports were processed normally with no new disk error. As the permanent fix, the recommendations were: time and size limits for report generation, controlled queue redelivery instead of unlimited, deleting temporary files in a finally block, an ephemeral-storage limit with a disk alert, a liveness check that accounts for disk health, and more than one worker replica. The report's status history in the database (three transitions to "generating", one after each restart) was the evidence that revealed the redelivery.

---

## 🇪🇸 Español

**issue:**
Los jobs programados de una aplicación Java en producción empezaron a devolver 502 (Bad Gateway) durante unas 5 horas. La ruta de los jobs era atendida por un único pod worker asíncrono. En los logs, el worker registró java.io.IOException: No space left on device al generar un reporte y, pocos segundos después, el health check de disco indicó unos 20 KB libres. El pod dejó de escribir logs pero siguió Ready y sin reinicios, porque Kubernetes no detecta un disco efímero lleno y siguió enviando tráfico a un pod inoperante.

**causa raíz:**
Un único reporte de uso extremadamente grande (horas de generación, frente a menos de 40 minutos en los demás reportes de los 14 días anteriores) escribía archivos temporales en el disco efímero del pod, que no tenía límite de ephemeral-storage ni volumen dedicado. Como el mensaje de la cola solo se confirma al final del procesamiento, cada reinicio del pod (un despliegue y el reinicio diario programado) hizo que el broker reentregara el mensaje y la generación comenzara de cero: tres intentos a lo largo de unas 14 horas, y el último corrió aproximadamente 5h37 hasta agotar el disco. Agravantes: en el código, el archivo temporal del camino principal solo se eliminaba en caso de éxito (sin bloque finally), otros caminos dependían de deleteOnExit (que solo libera al terminar la JVM) y el worker tenía una sola réplica, sin alerta de disco ni liveness que considerara el health de disco. Se descartaron: saturación de la base de datos y un reporte atascado en la base siendo reprocesado por un job de recuperación (ningún registro atascado; el mecanismo fue la reentrega de la cola).

**solución:**
Se reinició el deployment del worker, recreando el pod con disco limpio y restableciendo la ruta; el reporte ya estaba finalizado como error y no se reentregó de nuevo, y los reportes siguientes se procesaron con normalidad, sin nuevo error de disco. Como corrección definitiva se recomendaron: límite de tiempo y de tamaño para la generación de reportes, reentrega controlada de la cola en lugar de ilimitada, eliminación de los archivos temporales en un bloque finally, límite de ephemeral-storage con alerta de disco, liveness que considere el health de disco y más de una réplica del worker. El historial de estados del reporte en la base (tres transiciones a "generando", una después de cada reinicio) fue la evidencia que permitió identificar la reentrega.
