# 🔐 OCI – Pipeline de limpeza sem SSH entre runners após remoção manual de regra de rede

## 🇧🇷 Português (BR)

**issue:**
A pipeline de limpeza dos runners self-hosted de produção passou a falhar em todas as execuções agendadas, com "ssh: connect to host ... port 22: Connection timed out" em 9 dos 10 servidores. As máquinas estavam em execução e os runners online no GitHub.

**causa raiz:**
Durante uma limpeza manual de regras de IPs externos feita pelo Console, a regra de entrada TCP/22 com origem na sub-rede dos runners foi removida da Default Security List da VCN, junto com regras de IPs de terceiros. Essa regra permitia o SSH entre os runners (o job executa em um runner e conecta nos demais); os NSGs dos runners só liberavam a porta 22 para a VPN. A VCN e a lista não eram gerenciadas por Terraform, então não houve diff nem revisão. A mudança foi localizada pelo log de Audit da OCI (evento UpdateSecurityList, com o estado das regras antes e depois).

**solução:**
Foi adicionada, de forma aditiva, uma regra de entrada TCP/22 restrita ao /32 de cada runner de origem nos NSGs dos servidores de destino, sem reabrir a sub-rede inteira. O grupo de runners possui dois runners e os jobs são distribuídos entre eles, então ambas as origens precisaram ser liberadas (o primeiro teste cobriu só uma e 4 de 10 jobs seguiram falhando). Após liberar as duas origens, a execução manual da pipeline terminou com 10 de 10 jobs com sucesso. Como débito técnico, as regras devem ser levadas ao Terraform, com NSG dedicado para os servidores que usavam um NSG compartilhado.

---

## 🇺🇸 English

**issue:**
The cleanup pipeline for the production self-hosted runners started failing on every scheduled run, with "ssh: connect to host ... port 22: Connection timed out" on 9 of 10 servers. The machines were running and the runners were online in GitHub.

**root cause:**
During a manual cleanup of external IP rules done in the Console, the ingress TCP/22 rule whose source was the runners' subnet was removed from the VCN's Default Security List, together with third-party IP rules. That rule allowed SSH between runners (the job runs on one runner and connects to the others); the runners' NSGs only allowed port 22 from the VPN. The VCN and the list were not managed by Terraform, so there was no diff or review. The change was located through the OCI Audit log (UpdateSecurityList event, with the rule state before and after).

**solution:**
An additive ingress TCP/22 rule restricted to the /32 of each source runner was added to the destination servers' NSGs, without reopening the whole subnet. The runner group has two runners and jobs are distributed between them, so both sources had to be allowed (the first test covered only one and 4 of 10 jobs kept failing). After allowing both sources, a manual run of the pipeline finished with 10 of 10 jobs successful. As technical debt, the rules should be moved to Terraform, with a dedicated NSG for the servers that used a shared NSG.

---

## 🇪🇸 Español

**issue:**
El pipeline de limpieza de los runners self-hosted de producción empezó a fallar en todas las ejecuciones programadas, con "ssh: connect to host ... port 22: Connection timed out" en 9 de 10 servidores. Las máquinas estaban en ejecución y los runners en línea en GitHub.

**causa raíz:**
Durante una limpieza manual de reglas de IPs externas hecha desde la Consola, la regla de entrada TCP/22 con origen en la subred de los runners se eliminó de la Default Security List de la VCN, junto con reglas de IPs de terceros. Esa regla permitía el SSH entre runners (el job se ejecuta en un runner y se conecta a los demás); los NSG de los runners solo permitían el puerto 22 desde la VPN. La VCN y la lista no estaban gestionadas por Terraform, por lo que no hubo diff ni revisión. El cambio se localizó mediante el log de Audit de OCI (evento UpdateSecurityList, con el estado de las reglas antes y después).

**solución:**
Se agregó, de forma aditiva, una regla de entrada TCP/22 restringida al /32 de cada runner de origen en los NSG de los servidores de destino, sin reabrir toda la subred. El grupo de runners tiene dos runners y los jobs se distribuyen entre ellos, por lo que fue necesario permitir ambos orígenes (la primera prueba cubrió solo uno y 4 de 10 jobs siguieron fallando). Tras permitir los dos orígenes, la ejecución manual del pipeline terminó con 10 de 10 jobs exitosos. Como deuda técnica, las reglas deben llevarse a Terraform, con un NSG dedicado para los servidores que usaban un NSG compartido.
