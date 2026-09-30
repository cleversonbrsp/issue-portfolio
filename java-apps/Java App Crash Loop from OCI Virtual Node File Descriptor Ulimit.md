# 🗂️ Aplicação Java (HOM) – Crash Loop da JVM por Limite de File Descriptors (ulimit -n)

## 🇧🇷 Português (BR)

**issue:**
A aplicação app-hom, em ambiente de homologação, ficou indisponível (bad gateway), com o pod em crash loop constante, sempre travando durante a inicialização da JVM, antes de qualquer lógica da aplicação ser executada.

**causa raiz:**
Para contexto: um file descriptor é a forma como o sistema operacional identifica qualquer recurso aberto por um processo, como um arquivo ou uma conexão de rede; o ulimit -n é o limite de quantos desses um processo pode manter abertos ao mesmo tempo; e o heap da JVM é a área de memória reservada especificamente para os objetos criados pela aplicação Java, separada de outras alocações internas feitas pela própria JVM.

O nó virtual do cluster Kubernetes, baseado em OCI Container Instances, aplica por padrão um limite de descritores de arquivo (ulimit -n) extremamente alto, superior a 1 bilhão, bem fora do valor típico esperado por aplicações comuns (na casa de milhares). Logo na inicialização, antes mesmo de qualquer lógica da aplicação rodar, a JVM utilizada pela aplicação tenta alocar nativamente uma tabela de controle proporcional a esse limite, e essa alocação falha por falta de memória, derrubando o processo. Uma redução do heap máximo da JVM, aplicada inicialmente como hipótese de correção, pareceu resolver por coincidência, mas o mesmo erro voltou a ocorrer horas depois, confirmando que o heap não era a causa real — o problema ocorria numa alocação nativa fora do heap, antes mesmo de o heap entrar em uso.

**solução:**
O comando de inicialização do container foi sobrescrito no manifesto de deployment para reduzir explicitamente o limite de descritores de arquivo (ulimit -n) para um valor razoável antes da JVM subir, eliminando a alocação problemática. A correção foi validada diretamente no cluster antes de ser registrada na configuração versionada. Uma correção mais abrangente, ajustando o entrypoint diretamente na imagem da aplicação para cobrir todos os ambientes, foi identificada e deixada para uma etapa futura, após confirmação de estabilidade da correção aplicada.

---

## 🇺🇸 English

**issue:**
The app-hom application, in the staging (homolog) environment, became unavailable (bad gateway), with the pod stuck in a constant crash loop, always failing during JVM startup, before any application logic ran.

**root cause:**
For context: a file descriptor is how the operating system identifies any resource a process has open, such as a file or a network connection; ulimit -n is the limit on how many of those a process can hold open at once; and the JVM heap is the memory area reserved specifically for the Java application's objects, separate from other internal allocations the JVM itself makes.

The Kubernetes virtual node, backed by OCI Container Instances, applies an extremely high default file descriptor limit (ulimit -n), over 1 billion, well beyond the typical value expected by ordinary applications (in the thousands). Right at startup, before any application logic runs, the JVM used by the application tries to natively allocate a control table sized proportionally to that limit, and this allocation fails due to insufficient memory, crashing the process. A reduction of the JVM's maximum heap size, applied first as a candidate fix, appeared to resolve the issue by coincidence, but the same error recurred hours later, confirming heap size was not the real cause — the problem occurred in a native allocation outside the heap, before the heap was even in use.

**solution:**
The container's startup command was overridden in the deployment manifest to explicitly cap the file descriptor limit (ulimit -n) to a reasonable value before the JVM starts, removing the problematic allocation. The fix was validated directly on the cluster before being recorded in version-controlled configuration. A broader fix, adjusting the entrypoint directly in the application image to cover all environments, was identified and left for a future step, pending confirmation that the applied fix holds up over time.

---

## 🇪🇸 Español

**issue:**
La aplicación app-hom, en el entorno de homologación, quedó indisponible (bad gateway), con el pod en un bucle constante de caídas, fallando siempre durante el arranque de la JVM, antes de que se ejecutara cualquier lógica de la aplicación.

**causa raíz:**
Para contexto: un file descriptor es la forma en que el sistema operativo identifica cualquier recurso abierto por un proceso, como un archivo o una conexión de red; el ulimit -n es el límite de cuántos de esos puede mantener abiertos un proceso a la vez; y el heap de la JVM es el área de memoria reservada específicamente para los objetos creados por la aplicación Java, separada de otras asignaciones internas que hace la propia JVM.

El nodo virtual del clúster de Kubernetes, respaldado por OCI Container Instances, aplica por defecto un límite de descriptores de archivo (ulimit -n) extremadamente alto, superior a 1000 millones, muy por encima del valor típico esperado por aplicaciones comunes (del orden de miles). Justo al arrancar, antes de que se ejecute cualquier lógica de la aplicación, la JVM utilizada por la aplicación intenta asignar de forma nativa una tabla de control proporcional a ese límite, y esa asignación falla por falta de memoria, provocando la caída del proceso. Una reducción del heap máximo de la JVM, aplicada primero como hipótesis de corrección, pareció resolver el problema por coincidencia, pero el mismo error volvió a ocurrir horas después, confirmando que el heap no era la causa real — el problema ocurría en una asignación nativa fuera del heap, antes incluso de que el heap entrara en uso.

**solución:**
El comando de inicio del contenedor fue sobrescrito en el manifiesto de deployment para reducir explícitamente el límite de descriptores de archivo (ulimit -n) a un valor razonable antes de iniciar la JVM, eliminando la asignación problemática. La corrección fue validada directamente en el clúster antes de registrarse en la configuración versionada. Una corrección más amplia, ajustando el entrypoint directamente en la imagen de la aplicación para cubrir todos los entornos, fue identificada y quedó para una etapa futura, a la espera de confirmar la estabilidad de la corrección aplicada.
