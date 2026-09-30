# 🔒 IIS – Hardening de TLS 1.2 e Remoção de Cipher Suites Fracas

## 🇧🇷 Português (BR)

**issue:**
Um scan de vulnerabilidades apontou que o servidor (IIS, aplicação web corporativa) suportava cipher suites fracas em TLS 1.2 — incluindo CBC sem PFS, RSA sem Perfect Forward Secrecy, 3DES e RC4 — um achado de segurança que expõe a conexão a ataques conhecidos (ex.: BEAST, Lucky13).

**causa raiz:**
A configuração padrão do Windows Server/IIS mantinha TLS 1.0 e 1.1 habilitados e não restringia a ordem/lista de cipher suites, permitindo negociação com algoritmos legados e inseguros sempre que um cliente os oferecesse.

**solução:**
TLS 1.0 e TLS 1.1 foram desabilitados (client e server) via registro do Windows, mantendo apenas TLS 1.2 habilitado, e a ordem de cipher suites foi restringida para priorizar ECDHE + AES-GCM (Perfect Forward Secrecy), removendo suporte a CBC sem PFS, RSA sem PFS, 3DES e RC4. A mudança foi validada tanto localmente (openssl s_client confirmando handshake TLS 1.2 com cipher ECDHE-RSA-AES256-GCM-SHA384) quanto externamente via SSL Labs, sem impacto na aplicação (a negociação TLS ocorre antes da requisição chegar ao código .NET/IIS).

---

## 🇺🇸 English

**issue:**
A vulnerability scan flagged that the server (IIS, corporate web application) supported weak cipher suites under TLS 1.2 — including CBC without PFS, RSA without Perfect Forward Secrecy, 3DES, and RC4 — a security finding that exposes the connection to known attacks (e.g. BEAST, Lucky13).

**root cause:**
The default Windows Server/IIS configuration kept TLS 1.0 and 1.1 enabled and didn't restrict the cipher suite list/order, allowing negotiation down to legacy, insecure algorithms whenever a client offered them.

**solution:**
TLS 1.0 and TLS 1.1 were disabled (client and server) via the Windows registry, leaving only TLS 1.2 enabled, and the cipher suite order was restricted to prioritize ECDHE + AES-GCM (Perfect Forward Secrecy), removing support for CBC without PFS, RSA without PFS, 3DES, and RC4. The change was validated both locally (openssl s_client confirming a TLS 1.2 handshake with cipher ECDHE-RSA-AES256-GCM-SHA384) and externally via SSL Labs, with no impact on the application (TLS negotiation happens before the request reaches the .NET/IIS code).

---

## 🇪🇸 Español

**issue:**
Un escaneo de vulnerabilidades detectó que el servidor (IIS, aplicación web corporativa) soportaba cipher suites débiles bajo TLS 1.2 — incluyendo CBC sin PFS, RSA sin Perfect Forward Secrecy, 3DES y RC4 — un hallazgo de seguridad que expone la conexión a ataques conocidos (p. ej. BEAST, Lucky13).

**causa raíz:**
La configuración predeterminada de Windows Server/IIS mantenía TLS 1.0 y 1.1 habilitados y no restringía la lista/orden de cipher suites, permitiendo negociar algoritmos legados e inseguros cada vez que un cliente los ofrecía.

**solución:**
Se deshabilitaron TLS 1.0 y TLS 1.1 (cliente y servidor) vía el registro de Windows, dejando solo TLS 1.2 habilitado, y se restringió el orden de cipher suites para priorizar ECDHE + AES-GCM (Perfect Forward Secrecy), eliminando el soporte a CBC sin PFS, RSA sin PFS, 3DES y RC4. El cambio se validó tanto localmente (openssl s_client confirmando un handshake TLS 1.2 con cipher ECDHE-RSA-AES256-GCM-SHA384) como externamente vía SSL Labs, sin impacto en la aplicación (la negociación TLS ocurre antes de que la solicitud llegue al código .NET/IIS).
