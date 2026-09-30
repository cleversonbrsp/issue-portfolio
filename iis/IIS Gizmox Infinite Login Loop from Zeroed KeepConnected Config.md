# 🖧 IIS/Gizmox – Loop Infinito de Login por KeepConnected Zerado no Web.config

## 🇧🇷 Português (BR)

**issue:**
O login da aplicação web corporativa (hospedada em IIS) entrava em loop infinito de requisições POST para a página de logon, todas retornando HTTP 200 sem nunca concluir o carregamento — o clique em ENTRAR não confirmava e a tela de login ficava travada. Era uma reincidência de um problema anterior (isolado ao Firefox), mas dessa vez ocorrendo em qualquer navegador, o que invalidava a hipótese original de roteamento por User-Agent.

**causa raiz:**
Após descartar hipóteses de conflito de bindings no IIS e de um health check mal configurado de outra aplicação no mesmo servidor, a causa raiz foi encontrada inspecionando o HAR e o console do DevTools: o client ficava enviando um evento KeepConnected sem parar, e o servidor respondia "sessão fechada" instantaneamente. No Web.config da aplicação (framework Gizmox/Visual WebGui), os parâmetros KeepConnectedInterval e KeepConnectedLimitation estavam zerados (Value="0"), fazendo o servidor nunca esperar e o client nunca parar de tentar — gerando o loop.

**solução:**
Os parâmetros foram ajustados no Web.config: KeepConnectedInterval para 5000ms e KeepConnectedLimitation para 240 (dando cerca de 20 minutos de tolerância antes da sessão expirar por inatividade — um valor inicial de teste com a tolerância padrão da documentação expirava a sessão rápido demais, em ~15s). Com esse ajuste, o loop parou e o login voltou a funcionar normalmente em qualquer navegador.

---

## 🇺🇸 English

**issue:**
Login to a corporate web application (hosted on IIS) entered an infinite loop of POST requests to the logon page, all returning HTTP 200 without ever completing the page load — clicking ENTER never confirmed and the login screen stayed stuck. This was a recurrence of an earlier issue (isolated to Firefox), but this time happening in every browser, which invalidated the original User-Agent-routing hypothesis.

**root cause:**
After ruling out hypotheses around IIS binding conflicts and a misconfigured health check from another application on the same server, the root cause was found by inspecting the HAR and DevTools console: the client kept sending a KeepConnected event nonstop, and the server instantly replied "session closed." In the application's Web.config (Gizmox/Visual WebGui framework), the KeepConnectedInterval and KeepConnectedLimitation parameters were set to zero (Value="0"), causing the server to never wait and the client to never stop retrying — generating the loop.

**solution:**
The parameters were adjusted in Web.config: KeepConnectedInterval to 5000ms and KeepConnectedLimitation to 240 (giving about 20 minutes of tolerance before the session expired from inactivity — an initial test with the documentation's default tolerance expired the session too fast, in ~15s). With this adjustment, the loop stopped and login started working normally again in every browser.

---

## 🇪🇸 Español

**issue:**
El login de una aplicación web corporativa (alojada en IIS) entraba en un loop infinito de solicitudes POST a la página de logon, todas devolviendo HTTP 200 sin completar nunca la carga de la página — el clic en ENTRAR nunca se confirmaba y la pantalla de login quedaba bloqueada. Era una recurrencia de un problema anterior (aislado a Firefox), pero esta vez ocurría en cualquier navegador, lo que invalidaba la hipótesis original de enrutamiento por User-Agent.

**causa raíz:**
Después de descartar hipótesis sobre conflictos de bindings en IIS y un health check mal configurado de otra aplicación en el mismo servidor, la causa raíz se encontró inspeccionando el HAR y la consola de DevTools: el cliente seguía enviando un evento KeepConnected sin parar, y el servidor respondía "sesión cerrada" al instante. En el Web.config de la aplicación (framework Gizmox/Visual WebGui), los parámetros KeepConnectedInterval y KeepConnectedLimitation estaban en cero (Value="0"), haciendo que el servidor nunca esperara y el cliente nunca dejara de reintentar — generando el loop.

**solución:**
Los parámetros se ajustaron en el Web.config: KeepConnectedInterval a 5000ms y KeepConnectedLimitation a 240 (dando cerca de 20 minutos de tolerancia antes de que la sesión expirara por inactividad — una prueba inicial con la tolerancia por defecto de la documentación expiraba la sesión demasiado rápido, en ~15s). Con este ajuste, el loop se detuvo y el login volvió a funcionar con normalidad en cualquier navegador.
