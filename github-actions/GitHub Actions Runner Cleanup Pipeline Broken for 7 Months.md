# 🔧 GitHub Actions – Pipeline de Limpeza de Runners Quebrada há 7 Meses sem Alerta

## 🇧🇷 Português (BR)

**issue:**
Durante uma checagem de disco nos runners self-hosted do GitHub Actions na OCI, foi identificado que uma das três pipelines de limpeza automática (daily-runner-cleanup.yml) estava falhando todos os dias com erro de autenticação SSH (Permission denied), cobrindo 8 instâncias que ficaram sem nenhuma limpeza automática de disco/cache.

**causa raiz:**
Investigando o histórico completo de execuções, a falha não era recente: a pipeline falhava diariamente desde 7 de fevereiro de 2026 — mais de 7 meses seguidos — sem que ninguém percebesse, porque a estratégia de matrix não tinha fail-fast: false configurado, então a falha no primeiro servidor abortava a tentativa nos demais sem gerar alerta visível. A causa raiz técnica foi uma provável rotação do secret RUNNER_SSH_KEY no GitHub Actions (ou troca das chaves autorizadas nos servidores) por volta dessa data, já que nenhuma mudança havia sido feita no código do workflow.

**solução:**
Foi aberto e mergeado um PR desativando o agendamento automático dessa pipeline quebrada (mantendo apenas execução manual via workflow_dispatch). Em paralelo, a cobertura de limpeza dessas instâncias órfãs foi migrada para o padrão mais seguro já usado pelas outras duas pipelines de limpeza (que protegem explicitamente o home do runner e nunca derrubam containers em execução, ao contrário da pipeline antiga que fazia docker kill indiscriminado e rm -rf do diretório de trabalho). A nova cobertura foi validada de ponta a ponta com execução manual bem-sucedida em todas as instâncias, e o cron foi ajustado para um horário definido a partir de dados reais de uso (evitando colisão com outras rotinas diárias).

---

## 🇺🇸 English

**issue:**
During a disk-space check on GitHub Actions self-hosted runners in OCI, it was found that one of the three automated cleanup pipelines (daily-runner-cleanup.yml) was failing every day with an SSH authentication error (Permission denied), leaving 8 instances with no automatic disk/cache cleanup at all.

**root cause:**
Investigating the full run history, the failure wasn't recent: the pipeline had been failing daily since February 7, 2026 — over 7 months straight — without anyone noticing, because the matrix strategy didn't have fail-fast: false set, so a failure on the first server aborted the attempt on the rest without producing a visible alert. The underlying technical cause was a likely rotation of the RUNNER_SSH_KEY secret in GitHub Actions (or a change of authorized keys on the servers) around that date, since no change had been made to the workflow's code.

**solution:**
A PR was opened and merged to disable the broken pipeline's automatic schedule (keeping only manual execution via workflow_dispatch). In parallel, cleanup coverage for those orphaned instances was migrated to the safer pattern already used by the other two cleanup pipelines (which explicitly protect the runner's home directory and never kill running containers, unlike the old pipeline's indiscriminate docker kill and rm -rf of the working directory). The new coverage was validated end-to-end with a successful manual run across all instances, and the cron schedule was tuned based on real usage data to avoid colliding with other daily routines.

---

## 🇪🇸 Español

**issue:**
Durante una revisión de espacio en disco en los runners self-hosted de GitHub Actions en OCI, se identificó que uno de los tres pipelines de limpieza automática (daily-runner-cleanup.yml) fallaba todos los días con un error de autenticación SSH (Permission denied), dejando 8 instancias sin ninguna limpieza automática de disco/caché.

**causa raíz:**
Investigando el historial completo de ejecuciones, la falla no era reciente: el pipeline fallaba a diario desde el 7 de febrero de 2026 — más de 7 meses seguidos — sin que nadie lo notara, porque la estrategia de matrix no tenía fail-fast: false configurado, por lo que la falla en el primer servidor abortaba el intento en los demás sin generar ninguna alerta visible. La causa raíz técnica fue probablemente una rotación del secret RUNNER_SSH_KEY en GitHub Actions (o un cambio de claves autorizadas en los servidores) alrededor de esa fecha, ya que no se había hecho ningún cambio en el código del workflow.

**solución:**
Se abrió y fusionó un PR desactivando el schedule automático de ese pipeline roto (dejando solo ejecución manual vía workflow_dispatch). En paralelo, la cobertura de limpieza de esas instancias huérfanas se migró al patrón más seguro ya usado por los otros dos pipelines de limpieza (que protegen explícitamente el home del runner y nunca detienen contenedores en ejecución, a diferencia del pipeline antiguo que hacía docker kill indiscriminado y rm -rf del directorio de trabajo). La nueva cobertura se validó de punta a punta con una ejecución manual exitosa en todas las instancias, y el cron se ajustó a un horario definido a partir de datos reales de uso, evitando colisión con otras rutinas diarias.
