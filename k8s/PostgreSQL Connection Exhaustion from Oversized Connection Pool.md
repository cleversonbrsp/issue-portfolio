# 🔌 PostgreSQL – Conexões Recusadas por Esgotamento de max_connections (Pool de Conexões Superdimensionado)

## 🇧🇷 Português (BR)

**issue:**
Um banco PostgreSQL gerenciado de homologação, aparentemente indisponível, recusava novas conexões com o erro "FATAL: remaining connection slots are reserved for non-replication superuser connections", embora o serviço estivesse no ar e a porta respondesse. CPU, memória e sessões ativas estavam baixas, sem excesso de consultas. A métrica de conexões ficou travada no teto (247) durante horas.

**causa raiz:**
Para contexto: max_connections é o limite de conexões simultâneas que o PostgreSQL aceita (aqui, 250), com alguns slots reservados a superusuários; e o pool de conexões (Hikari) é o conjunto de conexões que cada instância de uma aplicação Java mantém abertas e reutiliza. O pool estava configurado com 80 conexões por instância, valor herdado de um arquivo de configuração base compartilhado entre ambientes. Com 3 réplicas, a aplicação mantinha cerca de 234 das 250 conexões, deixando quase nada para os demais clientes e para o acesso manual. Duas reinicializações do banco no mesmo dia derrubaram todas as conexões, e todas as instâncias reconectaram ao mesmo tempo, esgotando o limite. Reiniciar um pod não resolvia: ao reiniciar uma réplica, cerca de 78 conexões foram liberadas e o pod novo reabriu todas em cerca de 2 minutos, o que mostrou que o problema era o dimensionamento do pool, e não um vazamento de conexões.

**solução:**
Como contorno imediato, uma réplica foi removida, liberando cerca de 79 conexões e permitindo o acesso ao banco. Como correção definitiva, o pool foi reduzido para 50 conexões por instância apenas no arquivo de configuração do ambiente de homologação, sem alterar o arquivo base (o que evita impacto em produção), por meio de pull request, seguido de um restart gradual (rolling restart) do deployment para que os pods lessem o valor novo no servidor de configuração. Validação pela métrica de conexões do banco: de 171 (2 réplicas com pool de 80) para cerca de 111 estáveis (2 réplicas com 50 mais cerca de 11 de outros clientes), restando cerca de 139 conexões livres das 250. Com o autoscaler permitindo até 3 réplicas, o pior caso fica em cerca de 161 de 250, sem risco de esgotar o limite.

---

## 🇺🇸 English

**issue:**
A managed PostgreSQL database in a staging environment, apparently unavailable, was refusing new connections with the error "FATAL: remaining connection slots are reserved for non-replication superuser connections", even though the service was up and the port was responding. CPU, memory and active sessions were low, with no excess of queries. The connections metric stayed pinned at its ceiling (247) for hours.

**root cause:**
For context: max_connections is the limit of simultaneous connections PostgreSQL accepts (here, 250), with a few slots reserved for superusers; and the connection pool (Hikari) is the set of connections each instance of a Java application keeps open and reuses. The pool was configured with 80 connections per instance, a value inherited from a base configuration file shared across environments. With 3 replicas, the application held about 234 of the 250 connections, leaving almost nothing for other clients and manual access. Two database restarts on the same day dropped all connections, and every instance reconnected at the same time, exhausting the limit. Restarting a pod did not help: restarting one replica freed about 78 connections and the new pod reopened all of them within about 2 minutes, which showed the problem was pool sizing, not a connection leak.

**solution:**
As an immediate workaround, one replica was removed, freeing about 79 connections and restoring access to the database. As the permanent fix, the pool was reduced to 50 connections per instance only in the staging environment's configuration file, leaving the base file untouched (which avoids any production impact), via pull request, followed by a rolling restart of the deployment so the pods read the new value from the configuration server. Validated with the database connections metric: from 171 (2 replicas with a pool of 80) to a stable ~111 (2 replicas with 50 each plus ~11 from other clients), leaving about 139 of the 250 connections free. With the autoscaler allowing up to 3 replicas, the worst case is about 161 of 250, with no risk of exhausting the limit.

---

## 🇪🇸 Español

**issue:**
Una base de datos PostgreSQL gestionada de homologación, aparentemente no disponible, rechazaba nuevas conexiones con el error "FATAL: remaining connection slots are reserved for non-replication superuser connections", aunque el servicio estaba activo y el puerto respondía. La CPU, la memoria y las sesiones activas estaban bajas, sin exceso de consultas. La métrica de conexiones quedó fija en el tope (247) durante horas.

**causa raíz:**
Para contexto: max_connections es el límite de conexiones simultáneas que acepta PostgreSQL (aquí, 250), con algunos slots reservados para superusuarios; y el pool de conexiones (Hikari) es el conjunto de conexiones que cada instancia de una aplicación Java mantiene abiertas y reutiliza. El pool estaba configurado con 80 conexiones por instancia, valor heredado de un archivo de configuración base compartido entre entornos. Con 3 réplicas, la aplicación mantenía unas 234 de las 250 conexiones, dejando casi nada para los demás clientes y para el acceso manual. Dos reinicios de la base de datos el mismo día cortaron todas las conexiones, y todas las instancias se reconectaron al mismo tiempo, agotando el límite. Reiniciar un pod no lo resolvía: al reiniciar una réplica se liberaron unas 78 conexiones y el pod nuevo las reabrió todas en unos 2 minutos, lo que mostró que el problema era el dimensionamiento del pool y no una fuga de conexiones.

**solución:**
Como solución provisional inmediata se eliminó una réplica, liberando unas 79 conexiones y permitiendo el acceso a la base. Como corrección definitiva, el pool se redujo a 50 conexiones por instancia solo en el archivo de configuración del entorno de homologación, sin modificar el archivo base (lo que evita impacto en producción), mediante un pull request, seguido de un reinicio gradual (rolling restart) del deployment para que los pods leyeran el valor nuevo desde el servidor de configuración. Validación con la métrica de conexiones de la base: de 171 (2 réplicas con pool de 80) a unas 111 estables (2 réplicas con 50 más unas 11 de otros clientes), quedando unas 139 conexiones libres de las 250. Con el autoscaler permitiendo hasta 3 réplicas, el peor caso es de unas 161 de 250, sin riesgo de agotar el límite.
