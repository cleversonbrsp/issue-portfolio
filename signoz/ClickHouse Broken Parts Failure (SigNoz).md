# 📊 SigNoz/ClickHouse – Falha por Excesso de Broken Parts (Code 231)

## 🇧🇷 Português (BR)

**issue:**
O pipeline cleanup-servers e o acesso interativo ao ClickHouse passaram a falhar ao consultar a instância do SigNoz, porque uma ou mais tabelas MergeTree (ex.: `signoz_traces.top_level_operations`) não conseguiam ser carregadas. O servidor retornava Code 231 (TOO_MANY_UNEXPECTED_DATA_PARTS): o número de partes "quebradas" excedia o limite padrão de max_suspicious_broken_parts (100). Qualquer query que tocasse `system.parts` — incluindo as queries de limpeza do pipeline — falhava assim que uma tabela não conseguia ser anexada.

**causa raiz:**
O container do ClickHouse havia reiniciado (restart do Docker daemon) com o disco do host em 96.6% de uso. Um shutdown não limpo, combinado com alta carga e pouco espaço em disco, deixou várias partes MergeTree em estado inconsistente (na maioria partes órfãs/vazias de "0.00 B"). O limite de segurança do ClickHouse então recusou anexar as tabelas afetadas para evitar trabalhar silenciosamente com dados corrompidos — isso é um comportamento proposital, não um bug.

**solução:**
Foi adicionado um snippet de configuração merge_tree.xml aumentando o max_suspicious_broken_parts (primeiro para 200, depois para 300 quando uma segunda tabela excedeu 200 partes quebradas) em `/etc/clickhouse-server/config.d/`, e o container do ClickHouse foi reiniciado para que as tabelas voltassem a anexar. A recuperação foi validada com uma query em `system.parts`. Isso é um workaround, não uma correção da corrupção subjacente — o plano seguinte era rodar OPTIMIZE TABLE ... FINAL nas tabelas afetadas e reduzir gradualmente o limite de volta ao padrão, além de liberar espaço em disco e evitar restarts abruptos no futuro.

---

## 🇺🇸 English

**issue:**
The cleanup-servers pipeline and interactive ClickHouse access started failing when querying the SigNoz ClickHouse instance, because one or more MergeTree tables (e.g. `signoz_traces.top_level_operations`) could not be loaded. The server returned Code 231 (TOO_MANY_UNEXPECTED_DATA_PARTS): the number of broken parts exceeded the default max_suspicious_broken_parts limit (100). Any query touching `system.parts` — including the pipeline's cleanup queries — failed as soon as one table couldn't attach.

**root cause:**
The ClickHouse container had restarted (Docker daemon restart) with the host disk at 96.6% usage. An unclean shutdown combined with high load and low disk space left many MergeTree parts in an inconsistent state (mostly empty/orphaned "0.00 B" parts). ClickHouse's safety limit then refused to attach the affected tables to avoid silently working with corrupted data — which is by design, not a bug.

**solution:**
Added a merge_tree.xml config snippet raising max_suspicious_broken_parts (first to 200, then to 300 when a second table exceeded 200 broken parts) under `/etc/clickhouse-server/config.d/`, then restarted the ClickHouse container so tables could attach again. Verified recovery with a query against `system.parts`. This is a workaround, not a fix for the underlying corruption — the plan afterward was to run OPTIMIZE TABLE ... FINAL on affected tables and gradually lower the limit back toward default, plus free disk space and avoid abrupt restarts going forward.

---

## 🇪🇸 Español

**issue:**
El pipeline cleanup-servers y el acceso interactivo a ClickHouse comenzaron a fallar al consultar la instancia de SigNoz, porque una o más tablas MergeTree (p. ej. `signoz_traces.top_level_operations`) no podían cargarse. El servidor devolvía Code 231 (TOO_MANY_UNEXPECTED_DATA_PARTS): el número de partes "rotas" superaba el límite predeterminado de max_suspicious_broken_parts (100). Cualquier consulta que tocara `system.parts` — incluidas las consultas de limpieza del pipeline — fallaba en cuanto una tabla no podía adjuntarse.

**causa raíz:**
El contenedor de ClickHouse se había reiniciado (restart del daemon de Docker) con el disco del host al 96.6% de uso. Un apagado no limpio, combinado con alta carga y poco espacio en disco, dejó varias partes MergeTree en un estado inconsistente (en su mayoría partes huérfanas/vacías de "0.00 B"). El límite de seguridad de ClickHouse entonces se negó a adjuntar las tablas afectadas para evitar trabajar silenciosamente con datos corruptos — esto es un comportamiento intencional, no un bug.

**solución:**
Se agregó un fragmento de configuración merge_tree.xml aumentando max_suspicious_broken_parts (primero a 200, luego a 300 cuando una segunda tabla superó 200 partes rotas) en `/etc/clickhouse-server/config.d/`, y se reinició el contenedor de ClickHouse para que las tablas volvieran a adjuntarse. La recuperación se verificó con una consulta a `system.parts`. Esto es un workaround, no una corrección de la corrupción subyacente — el plan posterior era ejecutar OPTIMIZE TABLE ... FINAL en las tablas afectadas y reducir gradualmente el límite de vuelta al valor predeterminado, además de liberar espacio en disco y evitar reinicios abruptos en el futuro.
