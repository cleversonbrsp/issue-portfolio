# 🐙 GitHub Actions – Fila de Workflows Travada por Label de Runner Ausente

## 🇧🇷 Português (BR)

**issue:**
Workflows do GitHub Actions no repositório app-repo ficaram presos em fila (Queued/Pending) por dezenas de minutos — alguns por quase 24h antes de serem cancelados por um push mais recente na mesma branch — atrasando deploys. O workflow afetado usava runs-on: prod-runner.

**causa raiz:**
Nenhum runner self-hosted da organização estava com o label prod-runner exigido pelos workflows sonar.yml e deploy-hml.yml. Sem nenhum runner com esse label, os jobs ficavam presos indefinidamente sem gerar nenhum erro visível. Os runners existentes (runner-1-sonar e runner-2-sonar) estavam saudáveis, online e ociosos o tempo todo — não era um problema de infraestrutura/disponibilidade das VMs, e sim de configuração de label no GitHub.

**solução:**
O label prod-runner foi adicionado diretamente aos dois runners existentes, e os jobs voltaram a ser despachados normalmente. Durante a validação, surgiu uma falha secundária de build por uso de actions/cache@v1 (descontinuado) — corrigido para actions/cache@v4 via PR. Uma terceira falha (incompatibilidade de versão do Java) foi identificada como responsabilidade do time de desenvolvimento (configuração da aplicação), não do runner/CI.

---

## 🇺🇸 English

**issue:**
GitHub Actions workflows in the app-repo repository got stuck in queue (Queued/Pending) for dozens of minutes — some for almost 24h before being canceled by a newer push to the same branch — delaying deployments. The affected workflow used runs-on: prod-runner.

**root cause:**
No self-hosted runner in the organization had the prod-runner label required by the sonar.yml and deploy-hml.yml workflows. With no runner carrying that label, jobs stayed queued indefinitely without producing any visible error. The existing runners (runner-1-sonar and runner-2-sonar) were healthy, online, and idle the whole time — it wasn't an infrastructure/VM-availability problem, but a GitHub label configuration issue.

**solution:**
The prod-runner label was added directly to the two existing runners, and jobs started being dispatched normally again. During validation, a secondary build failure surfaced from using the deprecated actions/cache@v1 — fixed to actions/cache@v4 via a PR. A third failure (a Java version mismatch) was identified as the development team's responsibility (application configuration), not a runner/CI issue.

---

## 🇪🇸 Español

**issue:**
Los workflows de GitHub Actions en el repositorio app-repo quedaron atascados en cola (Queued/Pending) durante decenas de minutos — algunos casi 24h antes de ser cancelados por un push más reciente en la misma rama — retrasando los despliegues. El workflow afectado usaba runs-on: prod-runner.

**causa raíz:**
Ningún runner self-hosted de la organización tenía la etiqueta prod-runner requerida por los workflows sonar.yml y deploy-hml.yml. Sin ningún runner con esa etiqueta, los jobs quedaban en cola indefinidamente sin generar ningún error visible. Los runners existentes (runner-1-sonar y runner-2-sonar) estaban saludables, en línea y ociosos todo el tiempo — no era un problema de infraestructura/disponibilidad de las VMs, sino de configuración de etiquetas en GitHub.

**solución:**
Se agregó la etiqueta prod-runner directamente a los dos runners existentes, y los jobs volvieron a despacharse con normalidad. Durante la validación surgió una falla secundaria de build por el uso de actions/cache@v1 (descontinuado) — corregida a actions/cache@v4 mediante un PR. Una tercera falla (incompatibilidad de versión de Java) se identificó como responsabilidad del equipo de desarrollo (configuración de la aplicación), no del runner/CI.
