# 💽 SigNoz – Disco Cheio por Tabelas de Sistema Órfãs no ClickHouse

## 🇧🇷 Português (BR)

**issue:**
Alarme do Zabbix disparou para o host signoz-vm por espaço em disco: o filesystem / estava em 100% de uso, restando apenas cerca de 45MB livres.

**causa raiz:**
O volume Docker do ClickHouse (169.9GB) concentrava quase todo o uso de /var/lib/docker. Cerca de 86GB eram dados legítimos (traces e métricas do SigNoz, com TTL configurado). Os outros ~70GB eram tabelas internas de log do próprio ClickHouse (query_log, trace_log, metric_log, part_log, etc.) que o ClickHouse havia renomeado automaticamente para os sufixos _0 e _1 durante duas trocas de versão anteriores — essas tabelas renomeadas ficaram sem TTL configurado e acumulando dados indefinidamente, sem nenhuma limpeza automática.

**solução:**
Foram removidas as 12 tabelas órfãs do ClickHouse (query_log_0/1, trace_log_0/1, metric_log_0/1, part_log_0/1, asynchronous_metric_log_1, query_views_log_0/1, processors_profile_log_0), liberando o espaço em disco de 2.4GB (99% de uso) para 70GB livres (65% de uso). Como prevenção, ficou registrada a necessidade de configurar TTL explícito nas tabelas de sistema do ClickHouse para evitar que o problema se repita em upgrades futuros.

---

## 🇺🇸 English

**issue:**
A Zabbix alarm fired for the signoz-vm host over disk space: the / filesystem was at 100% usage, with only about 45MB free.

**root cause:**
The ClickHouse Docker volume (169.9GB) accounted for almost all of /var/lib/docker's usage. About 86GB was legitimate application data (SigNoz traces and metrics, with TTL configured). The other ~70GB were ClickHouse's own internal log tables (query_log, trace_log, metric_log, part_log, etc.) that ClickHouse had automatically renamed with _0/_1 suffixes during two previous version upgrades — those renamed tables ended up without any TTL configured and kept accumulating data indefinitely, with no automatic cleanup.

**solution:**
Removed the 12 orphaned ClickHouse tables (query_log_0/1, trace_log_0/1, metric_log_0/1, part_log_0/1, asynchronous_metric_log_1, query_views_log_0/1, processors_profile_log_0), freeing up disk space from 2.4GB free (99% used) to 70GB free (65% used). As prevention, it was flagged that explicit TTLs need to be configured on ClickHouse's system tables to avoid the same issue recurring after future upgrades.

---

## 🇪🇸 Español

**issue:**
Se disparó una alarma de Zabbix para el host signoz-vm por espacio en disco: el filesystem / estaba al 100% de uso, con apenas unos 45MB libres.

**causa raíz:**
El volumen Docker de ClickHouse (169.9GB) concentraba casi todo el uso de /var/lib/docker. Cerca de 86GB eran datos legítimos (trazas y métricas de SigNoz, con TTL configurado). Los otros ~70GB eran tablas internas de log del propio ClickHouse (query_log, trace_log, metric_log, part_log, etc.) que ClickHouse había renombrado automáticamente con sufijos _0 y _1 durante dos actualizaciones de versión anteriores — esas tablas renombradas quedaron sin TTL configurado y acumulando datos indefinidamente, sin ninguna limpieza automática.

**solución:**
Se eliminaron las 12 tablas huérfanas de ClickHouse (query_log_0/1, trace_log_0/1, metric_log_0/1, part_log_0/1, asynchronous_metric_log_1, query_views_log_0/1, processors_profile_log_0), liberando espacio en disco de 2.4GB libres (99% de uso) a 70GB libres (65% de uso). Como prevención, quedó registrada la necesidad de configurar TTL explícito en las tablas de sistema de ClickHouse para evitar que el problema se repita en futuras actualizaciones.
