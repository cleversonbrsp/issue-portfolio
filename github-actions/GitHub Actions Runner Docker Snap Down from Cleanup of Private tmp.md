# 🐳 CI/CD – Docker (snap) não inicia no runner após limpeza automática de /tmp

## 🇧🇷 Português (BR)

**issue:**
Os builds de um runner self-hosted de homologação falhavam no passo de build/push de imagem com "failed to connect to the docker API at unix:///var/run/docker.sock: no such file or directory", e em outro runner com "cannot create temporary directory for the root file system". O serviço Docker (instalado via snap) estava parado e não voltava sozinho; a máquina tinha disco, inodes, CPU e memória normais.

**causa raiz:**
O Docker via snap usa um diretório /tmp privado (/tmp/snap-private-tmp/snap.docker). A pipeline de limpeza executava diariamente find /tmp -xdev -type d -empty -delete, que removia esse diretório vazio. Na reinicialização seguinte do serviço (refresh automático do snap), o dockerd não conseguia recriá-lo ("mktemp: failed to create directory via template /tmp/tmp.XXXXXXXXXX"), esgotava as tentativas do systemd e permanecia parado até um reboot.

**solução:**
O comando de limpeza passou a ignorar o diretório do snap: find /tmp -xdev -type d -empty -not -path '/tmp/snap-private-tmp*' -delete, aplicado em todos os jobs do workflow. O serviço foi restabelecido com reboot do runner. A correção foi validada executando a pipeline manualmente e reiniciando o snap do Docker em seguida (o dockerd subiu normalmente e o diretório foi preservado). Recomenda-se aplicar o mesmo ajuste ao workflow dos runners de produção, que possui o mesmo comando.

---

## 🇺🇸 English

**issue:**
Builds on a self-hosted CI runner in a staging environment failed at the image build/push step with "failed to connect to the docker API at unix:///var/run/docker.sock: no such file or directory", and on another runner with "cannot create temporary directory for the root file system". The Docker service (installed as a snap) was stopped and did not recover by itself, although disk, inodes, CPU and memory were normal.

**root cause:**
Snap-installed Docker uses a private /tmp directory (/tmp/snap-private-tmp/snap.docker). The cleanup pipeline ran find /tmp -xdev -type d -empty -delete daily, which removed that empty directory. On the next service restart (automatic snap refresh), dockerd could not recreate it ("mktemp: failed to create directory via template /tmp/tmp.XXXXXXXXXX"), exhausted systemd's restart attempts and stayed down until a reboot.

**solution:**
The cleanup command now skips the snap directory: find /tmp -xdev -type d -empty -not -path '/tmp/snap-private-tmp*' -delete, applied to every job of the workflow. The service was restored by rebooting the runner. The fix was validated by running the pipeline manually and then restarting the Docker snap (dockerd started normally and the directory was preserved). It is recommended to apply the same change to the production runners' workflow, which has the same command.

---

## 🇪🇸 Español

**issue:**
Los builds de un runner self-hosted de un entorno de homologación fallaban en el paso de build/push de imagen con "failed to connect to the docker API at unix:///var/run/docker.sock: no such file or directory", y en otro runner con "cannot create temporary directory for the root file system". El servicio Docker (instalado vía snap) estaba detenido y no se recuperaba solo; el disco, los inodes, la CPU y la memoria estaban normales.

**causa raíz:**
Docker instalado vía snap usa un directorio /tmp privado (/tmp/snap-private-tmp/snap.docker). El pipeline de limpieza ejecutaba a diario find /tmp -xdev -type d -empty -delete, que eliminaba ese directorio vacío. En el siguiente reinicio del servicio (refresh automático del snap), dockerd no podía recrearlo ("mktemp: failed to create directory via template /tmp/tmp.XXXXXXXXXX"), agotaba los reintentos de systemd y permanecía detenido hasta un reinicio.

**solución:**
El comando de limpieza pasó a ignorar el directorio del snap: find /tmp -xdev -type d -empty -not -path '/tmp/snap-private-tmp*' -delete, aplicado en todos los jobs del workflow. El servicio se restableció reiniciando el runner. La corrección se validó ejecutando el pipeline manualmente y reiniciando después el snap de Docker (dockerd arrancó con normalidad y el directorio se conservó). Se recomienda aplicar el mismo cambio al workflow de los runners de producción, que tiene el mismo comando.
