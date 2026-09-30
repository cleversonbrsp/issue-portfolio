# 🚪 Traefik >3.7 – Upgrade Falha por CRDs do Gateway API Não Instalados

## 🇧🇷 Português (BR)

**issue:**
Uma manutenção de rotina no Traefik (via `helm upgrade`) começou a falhar com o erro: `failed to create typed live object (GatewayClass): .status.supportedFeatures: element 0: associative list without keys has an element that's a map type`.

**causa raiz:**
A partir da versão 3.7, o chart Helm do Traefik parou de instalar automaticamente os CRDs do Gateway API — um aviso de depreciação já alertava sobre isso, mas passou despercebido até impactar o ambiente em uma manutenção seguinte. Sem os CRDs na versão esperada pelo Traefik, o `helm upgrade` falhava ao tentar criar/atualizar o recurso GatewayClass.

**solução:**
Os CRDs do Gateway API foram instalados manualmente antes do upgrade do Traefik, aplicando o manifesto oficial (`kubectl apply -f .../gateway-api/releases/download/v1.5.1/standard-install.yaml`). Com os CRDs na versão correta já presentes no cluster, o `helm upgrade` do Traefik passou a funcionar normalmente.

---

## 🇺🇸 English

**issue:**
A routine Traefik maintenance (via `helm upgrade`) started failing with the error: `failed to create typed live object (GatewayClass): .status.supportedFeatures: element 0: associative list without keys has an element that's a map type`.

**root cause:**
Starting with version 3.7, the Traefik Helm chart stopped automatically installing the Gateway API CRDs — a deprecation warning had already flagged this, but it went unnoticed until it impacted the environment during a later maintenance. Without the CRDs at the version Traefik expected, the `helm upgrade` failed while trying to create/update the GatewayClass resource.

**solution:**
The Gateway API CRDs were installed manually before the Traefik upgrade, applying the official manifest (`kubectl apply -f .../gateway-api/releases/download/v1.5.1/standard-install.yaml`). With the correct CRD version already present in the cluster, the Traefik `helm upgrade` started working normally again.

---

## 🇪🇸 Español

**issue:**
Un mantenimiento rutinario de Traefik (vía `helm upgrade`) comenzó a fallar con el error: `failed to create typed live object (GatewayClass): .status.supportedFeatures: element 0: associative list without keys has an element that's a map type`.

**causa raíz:**
A partir de la versión 3.7, el chart de Helm de Traefik dejó de instalar automáticamente los CRDs del Gateway API — una advertencia de depreciación ya lo indicaba, pero pasó desapercibida hasta impactar el entorno en un mantenimiento posterior. Sin los CRDs en la versión que Traefik esperaba, el `helm upgrade` fallaba al intentar crear/actualizar el recurso GatewayClass.

**solución:**
Los CRDs del Gateway API se instalaron manualmente antes de actualizar Traefik, aplicando el manifiesto oficial (`kubectl apply -f .../gateway-api/releases/download/v1.5.1/standard-install.yaml`). Con los CRDs en la versión correcta ya presentes en el clúster, el `helm upgrade` de Traefik volvió a funcionar con normalidad.
