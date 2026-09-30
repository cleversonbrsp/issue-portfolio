# ☁️ CloudFlare × Kubernetes – Conflito de Protocolo HTTP/HTTPS no Ingress

## 🇧🇷 Português (BR)

**issue:**
Não acessa o site. O CloudFlare recebia a requisição, mas entendia que a comunicação com o cluster Kubernetes não estava criptografada — quando na verdade estava. Como o cluster está forçando o uso de HTTPS, o acesso via HTTP era recusado, enquanto o CloudFlare insistia em tentar acessar via HTTP, gerando um conflito entre os dois lados ("é http..." / "não, é https..." / "não é não, é http..." / "to dizendo que é https...").

**causa raiz:**
Ao criar o Ingress, o CloudFlare cria a entrada inicialmente com proxy habilitado e, em seguida, a remove. Quando isso acontece, se o ACME já havia capturado aquele endereço, ocorre um timeout de 1 hora (ou de alguns minutos), fazendo com que ele tente alcançar um endereço que não responde mais — o endereço de proxy inicial que o próprio CloudFlare já havia descartado logo em seguida.

**solução:**
A solução definitiva foi fechar a criptografia end-to-end no CloudFlare (HTTPS fim a fim) e atribuir a annotation `external-dns.alpha.kubernetes.io/cloudflare-proxied` como `"true"` no recurso de Ingress.

---

## 🇺🇸 English

**issue:**
The site was not accessible. CloudFlare was receiving the traffic, but it believed the communication with the Kubernetes cluster was not encrypted — when in fact it was. Since the cluster was enforcing HTTPS, HTTP access was rejected, while CloudFlare kept trying to reach it over HTTP, creating a back-and-forth conflict between the two sides ("it's http..." / "no, it's https..." / "no it's not, it's http..." / "I'm telling you it's https...").

**root cause:**
When the Ingress was created, CloudFlare initially creates the entry with the proxy enabled and then removes it shortly after. When this happens, if ACME had already picked up that address, a timeout of 1 hour (or a few minutes) occurs, causing it to try reaching an address that no longer responds — the initial proxy address that CloudFlare itself had already discarded right afterward.

**solution:**
The definitive fix was to enable end-to-end encryption in CloudFlare (full HTTPS) and set the `external-dns.alpha.kubernetes.io/cloudflare-proxied` annotation to `"true"` on the Ingress resource.

---

## 🇪🇸 Español

**issue:**
No se podía acceder al sitio. CloudFlare recibía el tráfico, pero entendía que la comunicación con el clúster de Kubernetes no estaba cifrada, cuando en realidad sí lo estaba. Como el clúster estaba forzando el uso de HTTPS, el acceso por HTTP era rechazado, mientras que CloudFlare insistía en intentar acceder por HTTP, generando un conflicto entre ambos lados ("es http..." / "no, es https..." / "no, es http..." / "te digo que es https...").

**causa raíz:**
Al crear el Ingress, CloudFlare crea la entrada inicialmente con el proxy habilitado y luego la elimina. Cuando esto ocurre, si el ACME ya había capturado esa dirección, se produce un timeout de 1 hora (o de algunos minutos), haciendo que intente alcanzar una dirección que ya no responde — la dirección de proxy inicial que el propio CloudFlare ya había descartado justo después.

**solución:**
La solución definitiva fue habilitar el cifrado end-to-end en CloudFlare (HTTPS de extremo a extremo) y asignar la anotación `external-dns.alpha.kubernetes.io/cloudflare-proxied` como `"true"` en el recurso de Ingress.
