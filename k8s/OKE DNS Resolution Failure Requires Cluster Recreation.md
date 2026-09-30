# 🌐 OKE – Falha de Resolução DNS Exige Recriação do Cluster

## 🇧🇷 Português (BR)

**issue:**
Durante a migração de uma plataforma, o provisionamento no ambiente de um cliente apresentou lentidão severa: os serviços da aplicação no cluster OKE falhavam intermitentemente ao resolver nomes DNS, tanto públicos quanto internos.

**causa raiz:**
O CoreDNS estava tratando registros DNS públicos como se fossem serviços internos do cluster, aplicando sufixos de busca incorretos (.svc.cluster.local e .cluster.local) em vez do domínio correto (.example.com) — rastreado a uma configuração de search DNS incorreta nos pods, confirmada via logs do CoreDNS, do DNS endpoint e da VCN.

**solução:**
Uma tentativa de corrigir via atualização de versão do cluster (para recriar o CoreDNS) travou no meio do processo, exigindo abertura de chamado (SR) com a Oracle Cloud para desbloquear a atualização. Diante disso, o cluster OKE inteiro foi deletado e reimplantado do zero via Terraform, com todos os serviços. Na nova implantação, a política de DNS da aplicação foi configurada explicitamente como ClusterFirst, apontando apenas para o DNS endpoint 10.0.0.20 com domínio de busca example.com — eliminando a resolução incorreta e restaurando o ambiente.

---

## 🇺🇸 English

**issue:**
During a platform migration, provisioning in a customer environment showed severe slowness: application services on the OKE cluster intermittently failed to resolve DNS names, both public and internal.

**root cause:**
CoreDNS was treating public DNS records as if they were internal cluster services, applying incorrect search suffixes (.svc.cluster.local and .cluster.local) instead of the correct domain (.example.com) — traced to an incorrect DNS search configuration on the pods, confirmed via CoreDNS logs, the DNS endpoint, and the VCN.

**solution:**
An attempt to fix it by upgrading the cluster version (to recreate CoreDNS) got stuck mid-process, requiring an Oracle Cloud support case (SR) to unblock the upgrade. Given that, the entire OKE cluster was deleted and redeployed from scratch via Terraform, including all services. In the new deployment, the application's DNS policy was explicitly set to ClusterFirst, pointing only to the 10.0.0.20 DNS endpoint with example.com as the search domain — eliminating the incorrect resolution and restoring the environment.

---

## 🇪🇸 Español

**issue:**
Durante la migración de una plataforma, el aprovisionamiento en el entorno de un cliente presentó una lentitud severa: los servicios de la aplicación en el clúster OKE fallaban intermitentemente al resolver nombres DNS, tanto públicos como internos.

**causa raíz:**
CoreDNS estaba tratando registros DNS públicos como si fueran servicios internos del clúster, aplicando sufijos de búsqueda incorrectos (.svc.cluster.local y .cluster.local) en lugar del dominio correcto (.example.com) — rastreado a una configuración de search DNS incorrecta en los pods, confirmada mediante logs de CoreDNS, del DNS endpoint y de la VCN.

**solución:**
Un intento de corregirlo actualizando la versión del clúster (para recrear CoreDNS) quedó atascado a mitad de proceso, requiriendo abrir un caso de soporte (SR) con Oracle Cloud para desbloquear la actualización. Ante esto, todo el clúster OKE fue eliminado y redesplegado desde cero vía Terraform, incluyendo todos los servicios. En el nuevo despliegue, la política de DNS de la aplicación se configuró explícitamente como ClusterFirst, apuntando solo al DNS endpoint 10.0.0.20 con dominio de búsqueda example.com — eliminando la resolución incorrecta y restaurando el entorno.
