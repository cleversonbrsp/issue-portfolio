# 📊 Grafana × Zabbix – Filtro por Empresa Bloqueado por Limite de 1 Valor por Item, Resolvido com LLD

## 🇧🇷 Português (BR)

**issue:**
Um dashboard Grafana precisava permitir selecionar uma empresa específica (via dropdown) e ver o status de migração só daquela empresa, mas a abordagem inicial — um único item Zabbix retornando um array JSON com todas as empresas, tratado no Grafana via transform "Extract fields" — não funcionava: o transform não expande um array JSON em várias linhas, só extrai chaves de um objeto único.

**causa raiz:**
O Zabbix armazena um único valor por item por coleta — não existe conceito nativo de "várias linhas" dentro de um item. Para ter uma tabela real no Grafana com uma linha por empresa (e um dropdown que realmente filtre), cada empresa precisaria ser um item Zabbix separado, o que exige uma regra de Discovery (LLD, Low-Level Discovery) em vez de um único item agregado.

**solução:**
Foi criada uma Discovery rule no host do Zabbix, com uma query simples (sem agregação JSON) retornando as colunas que o Zabbix converte automaticamente em macros de descoberta, e um item prototype que gera um item por empresa descoberta (um item por empresa). No Grafana, uma variável de dashboard populada dinamicamente a partir desses itens virou o dropdown de seleção, e um painel referenciando essa variável (com transforms Extract fields + Organize fields) passou a mostrar o status detalhado só da empresa escolhida. Testado trocando a empresa no dropdown, com o painel atualizando corretamente a cada seleção.

---

## 🇺🇸 English

**issue:**
A Grafana dashboard needed to let users select a specific company (via dropdown) and see migration status for just that company, but the initial approach — a single Zabbix item returning a JSON array with all companies, handled in Grafana via the "Extract fields" transform — didn't work: that transform doesn't expand a JSON array into multiple rows, it only extracts keys from a single object.

**root cause:**
Zabbix stores a single value per item per collection cycle — there's no native concept of "multiple rows" inside one item. To get a real Grafana table with one row per company (and a dropdown that actually filters), each company needed to be its own separate Zabbix item, which requires a Discovery rule (LLD, Low-Level Discovery) instead of one aggregated item.

**solution:**
A Discovery rule was created on the Zabbix host, using a plain query (no JSON aggregation) returning columns that Zabbix automatically converts into discovery macros, plus an item prototype that generates one item per discovered company (one per company). In Grafana, a dashboard variable dynamically populated from those items became the selection dropdown, and a panel referencing that variable (with Extract Fields + Organize Fields transforms) started showing detailed status for just the chosen company. Tested by switching companies in the dropdown, with the panel updating correctly on each selection.

---

## 🇪🇸 Español

**issue:**
Un dashboard de Grafana necesitaba permitir seleccionar una empresa específica (vía dropdown) y ver el estado de migración solo de esa empresa, pero el enfoque inicial — un único item de Zabbix devolviendo un array JSON con todas las empresas, manejado en Grafana vía el transform "Extract fields" — no funcionaba: ese transform no expande un array JSON en varias filas, solo extrae claves de un único objeto.

**causa raíz:**
Zabbix almacena un único valor por item por ciclo de recolección — no existe un concepto nativo de "varias filas" dentro de un item. Para tener una tabla real en Grafana con una fila por empresa (y un dropdown que realmente filtre), cada empresa necesitaba ser su propio item de Zabbix, lo que requiere una regla de Discovery (LLD, Low-Level Discovery) en lugar de un único item agregado.

**solución:**
Se creó una regla de Discovery en el host de Zabbix, con una consulta simple (sin agregación JSON) que devuelve columnas que Zabbix convierte automáticamente en macros de descubrimiento, más un item prototype que genera un item por cada empresa descubierta (uno por empresa). En Grafana, una variable de dashboard poblada dinámicamente a partir de esos items se convirtió en el dropdown de selección, y un panel que referencia esa variable (con transforms Extract Fields + Organize Fields) pasó a mostrar el estado detallado solo de la empresa elegida. Probado cambiando de empresa en el dropdown, con el panel actualizándose correctamente en cada selección.
