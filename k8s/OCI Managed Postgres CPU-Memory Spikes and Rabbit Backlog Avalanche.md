# 🐘 OCI Postgres × RabbitMQ – Avalanche de Backlog e Picos de CPU/Memória

## 🇧🇷 Português (BR)

**issue:**
Um backend Postgres gerenciado da OCI ficou instável sob carga (picos de CPU/memória, indisponibilidade intermitente) enquanto a aplicação continuava tentando reconectar e consumindo do RabbitMQ, criando um efeito avalanche: o backlog do RabbitMQ crescia, as reconexões aumentavam, e o banco oscilava ainda mais a cada restart da aplicação, já que a aplicação "pegava tudo de uma vez" e derrubava o sistema novamente.

**causa raiz:**
Vários loops de retroalimentação se reforçavam mutuamente: uma tempestade de retries (a aplicação tentando agressivamente após erros do banco), uma avalanche de backlog (burst de "catch-up" do RabbitMQ quando a aplicação voltava) e o dimensionamento do connection pool (tamanho do pool × réplicas × workers excedendo o que o banco gerenciado suportava) — sem ramp-up ou backpressure, quando a aplicação reiniciava ela retomava o consumo total instantaneamente e disparava o colapso de novo.

**solução:**
O incidente foi contido reduzindo os workers/consumers da aplicação para zero para tirar a pressão, aguardando o Postgres estabilizar, e então limpando (purge) a fila problemática do RabbitMQ para remover o gatilho de backlog. Estados de fatura "travados" no Postgres foram corrigidos via SQL (resetados para reprocessamento ou marcados como erro), e a aplicação voltou gradualmente (ramp-up), monitorando conexões do banco, profundidade da fila e taxa de erro antes de aumentar mais réplicas. Mitigações de longo prazo: backpressure/ramp-up nos consumers, retry com backoff exponencial + jitter, pools de conexão limitados, e uma rota de DLQ/quarentena para mensagens "venenosas".

---

## 🇺🇸 English

**issue:**
An OCI managed Postgres backend became unstable under load (CPU/memory spikes, intermittent unavailability) while the application kept retrying and consuming from RabbitMQ, creating an avalanche effect: the RabbitMQ backlog grew, reconnections surged, and the database flapped even harder with every app restart, since the application would "pick everything at once" and collapse the system again.

**root cause:**
Multiple reinforcing loops fed each other: a retry storm (the application retrying aggressively on DB errors), a backlog avalanche (RabbitMQ catch-up burst when the app came back), and connection pool sizing (pool size × replicas × workers exceeding what the managed DB could sustain) — with no ramp-up or backpressure when the app restarted, it resumed full consumption instantly and re-triggered the collapse.

**solution:**
Contained the incident by scaling the application's workers/consumers down to stop the pressure, waited for Postgres to stabilize, then purged the problematic RabbitMQ queue to remove the backlog trigger. Fixed inconsistent "stuck" invoice states in Postgres via SQL (reset for reprocessing or marked as error), then brought the application back gradually (ramp-up), watching DB connections, queue depth and error rate before increasing replicas further. Longer-term mitigations: backpressure/ramp-up on consumers, retry with exponential backoff + jitter, bounded connection pools, and a DLQ/quarantine path for poison messages.

---

## 🇪🇸 Español

**issue:**
Un backend de Postgres administrado por OCI se volvió inestable bajo carga (picos de CPU/memoria, indisponibilidad intermitente) mientras la aplicación seguía reintentando y consumiendo de RabbitMQ, generando un efecto avalancha: el backlog de RabbitMQ crecía, las reconexiones aumentaban, y la base de datos fluctuaba aún más con cada reinicio de la aplicación, ya que la aplicación "tomaba todo de una vez" y volvía a colapsar el sistema.

**causa raíz:**
Varios bucles de retroalimentación se reforzaban entre sí: una tormenta de reintentos (la aplicación reintentando agresivamente ante errores de la base de datos), una avalancha de backlog (ráfaga de "catch-up" de RabbitMQ cuando la aplicación volvía) y el dimensionamiento del connection pool (tamaño del pool × réplicas × workers superando lo que la base de datos administrada podía soportar) — sin ramp-up ni backpressure, al reiniciar la aplicación retomaba el consumo total de inmediato y volvía a disparar el colapso.

**solución:**
El incidente se contuvo reduciendo los workers/consumers de la aplicación a cero para quitar la presión, esperando a que Postgres se estabilizara, y luego purgando la cola problemática de RabbitMQ para eliminar el disparador del backlog. Los estados de factura "atascados" en Postgres se corrigieron vía SQL (reseteados para reprocesamiento o marcados como error), y la aplicación volvió gradualmente (ramp-up), monitoreando conexiones de la base de datos, profundidad de la cola y tasa de error antes de aumentar más réplicas. Mitigaciones a largo plazo: backpressure/ramp-up en los consumers, retry con backoff exponencial + jitter, pools de conexión acotados, y una ruta de DLQ/cuarentena para mensajes "envenenados".
