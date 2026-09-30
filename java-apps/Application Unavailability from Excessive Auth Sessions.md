# 🔐 Aplicação de Cliente – Indisponibilidade por Excesso de Sessões de Autenticação

## 🇧🇷 Português (BR)

**issue:**
Uma aplicação de cliente ficou indisponível porque o usuário de integração "svc-integration" estava realizando um volume excessivo de autenticações, fazendo a tabela spring_session_attributes do banco da aplicação crescer descontroladamente (chegando a 45GB). Essa taxa de inserts dificultava a validação do Flyway migration na inicialização da aplicação, impedindo o start.

**causa raiz:**
O usuário de integração svc-integration estava abrindo sessões em excesso (muitas threads/autenticações simultâneas), inflando a tabela de sessões do Spring muito além do esperado.

**solução:**
Para permitir um VACUUM FULL na tabela e limpar os registros desnecessários, foi necessário bloquear as autenticações do usuário — desativá-lo na aplicação e no Keycloak não foi suficiente, pois o "disabled" do Keycloak não impedia novas sessões; a solução foi excluir o usuário diretamente via banco de dados (DELETE em user_entity). Após o time responsável ajustar a quantidade de threads/forma de autenticação do robô, o usuário foi recriado no Keycloak (INSERT em user_entity) e reativado na aplicação.

---

## 🇺🇸 English

**issue:**
A customer application became unavailable because the "svc-integration" integration user was performing an excessive volume of authentications, causing the application database's spring_session_attributes table to grow uncontrollably (reaching 45GB). That insert rate made Flyway migration validation at application startup fail, blocking the app from starting.

**root cause:**
The svc-integration integration user was opening an excessive number of sessions (too many concurrent threads/authentications), bloating the Spring sessions table far beyond expected size.

**solution:**
To run a VACUUM FULL on the table and clean up the unnecessary records, the user's authentications had to be blocked first — disabling it in the application and in Keycloak wasn't enough, since Keycloak's "disabled" flag didn't stop new sessions from being created; the fix was to delete the user directly via the database (a DELETE on user_entity). After the responsible team adjusted the bot's thread count/authentication pattern, the user was recreated in Keycloak (an INSERT on user_entity) and re-enabled in the application.

---

## 🇪🇸 Español

**issue:**
Una aplicación de cliente quedó indisponible porque el usuario de integración "svc-integration" realizaba un volumen excesivo de autenticaciones, haciendo que la tabla spring_session_attributes de la base de la aplicación creciera sin control (llegando a 45GB). Esa tasa de inserts dificultaba la validación del Flyway migration al iniciar la aplicación, impidiendo que arrancara.

**causa raíz:**
El usuario de integración svc-integration estaba abriendo sesiones en exceso (demasiados hilos/autenticaciones simultáneas), inflando la tabla de sesiones de Spring muy por encima de lo esperado.

**solución:**
Para poder ejecutar un VACUUM FULL en la tabla y limpiar los registros innecesarios, primero fue necesario bloquear las autenticaciones del usuario — desactivarlo en la aplicación y en Keycloak no fue suficiente, ya que el "disabled" de Keycloak no impedía nuevas sesiones; la solución fue eliminar el usuario directamente vía base de datos (un DELETE en user_entity). Después de que el equipo responsable ajustara la cantidad de hilos/forma de autenticación del bot, el usuario fue recreado en Keycloak (un INSERT en user_entity) y reactivado en la aplicación.
