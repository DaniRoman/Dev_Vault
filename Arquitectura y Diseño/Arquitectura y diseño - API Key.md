 ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization?utm_source=chatgpt.com "Authorization header - HTTP | MDN - MDN Web Docs"))

## API key

Credencial sencilla que una aplicación envía a una API para identificar el cliente que hace la petición. Sirve muy bien para casos simples como cuota, facturación, trazabilidad básica o acceso técnico controlado, pero por sí sola no siempre identifica un “principal” con permisos finos. De hecho, en la documentación de Google Cloud se explica que una API key estándar puede asociar una petición a un proyecto, pero no necesariamente identifica un principal; además, recomiendan restringirlas y protegerlas durante almacenamiento y transmisión. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys?utm_source=chatgpt.com "Manage API keys | Authentication | Google Cloud Documentation"))

En lenguaje llano: una API key responde más a “**qué aplicación está llamando**” que a “**qué usuario concreto** está actuando”. Por eso encaja mejor en integraciones server-to-server sencillas que en modelos complejos de permisos por usuario, roles o scopes. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys?utm_source=chatgpt.com "Manage API keys | Authentication | Google Cloud Documentation"))

## 2) Qué significa “public” y “private” en API key

Cuando la gente dice **public API key** y **private API key**, muchas veces no está hablando de una norma formal, sino de una convención de diseño. No existe un estándar HTTP o RFC que diga “estas son las dos clases oficiales de API key”; es una forma práctica de distinguir credenciales que pueden quedar expuestas en cliente, frente a credenciales que deben quedarse solo en backend. Las guías de seguridad insisten precisamente en no incluir claves sensibles en código cliente ni en repositorios, y en aplicar restricciones de uso. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

### Public key

Una “public key” en este contexto suele ser una clave **menos sensible**, usada desde frontend, móvil o scripts que no controlas al 100%. No significa que sea “segura por estar publicada”, sino que su diseño asume exposición parcial y se mitiga con restricciones estrictas: origen web, IP, APIs permitidas, cuotas y casos de uso muy limitados. Google Cloud recomienda precisamente restringir las claves para reducir el impacto si se filtran. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

### Private key

Una “private key” es una clave que **no debe salir del backend**. Va en servicios internos, workers, cron jobs o micros. Se guarda en variables de entorno o un secrets manager, no en frontend, no en apps empaquetadas y no en repositorios. Si se expone, cualquier tercero que la obtenga puede hacerse pasar por tu servicio hasta que la revoques o la rotes. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

## 3) Cuándo usar API key y cuándo no

La API key es razonable cuando quieres resolver **autenticación técnica simple** entre componentes controlados, especialmente si tu modelo de permisos es pequeño. Pero cuando necesitas permisos más expresivos, expiración, scopes, delegación, auditoría fina o actuar “en nombre de” un usuario o cliente, normalmente conviene usar **tokens** más ricos, típicamente OAuth 2.0 o JWT emitidos dentro de ese modelo. OAuth 2.0 define tanto acceso en nombre de un usuario como acceso que la aplicación obtiene por sí misma. ([RFC Editor](https://www.rfc-editor.org/rfc/rfc6749?utm_source=chatgpt.com "RFC 6749: The OAuth 2.0 Authorization Framework"))

Dicho de forma práctica: si lo único que necesitas es “mi micro A puede llamar al endpoint interno B”, una API key puede valer para arrancar. Si necesitas “este servicio puede descargar firmware pero no borrarlo, solo para ciertos workspaces, con caducidad y trazabilidad clara”, ya suena más a token de aplicación con scopes o claims. ([RFC Editor](https://www.rfc-editor.org/rfc/rfc6749?utm_source=chatgpt.com "RFC 6749: The OAuth 2.0 Authorization Framework"))

## 4) La gran diferencia: API key vs token

Una **API key** suele ser estática, simple y de larga vida si no haces una buena rotación. Un **token** suele ser temporal, firmado o emitido por un sistema de identidad, y puede cargar permisos, audiencia, vencimiento y contexto. OAuth 2.0 precisamente estandariza el acceso limitado a un recurso HTTP, ya sea en nombre de un usuario o por cuenta propia de la aplicación. Las recomendaciones modernas de seguridad OAuth también endurecen prácticas antiguas y desaconsejan modos menos seguros. ([RFC Editor](https://www.rfc-editor.org/rfc/rfc6749?utm_source=chatgpt.com "RFC 6749: The OAuth 2.0 Authorization Framework"))

Por eso, cuando la pregunta es “¿API key privada o token de aplicación?”, la respuesta técnica madura suele ser: **API key para empezar simple; token para crecer sin dolor**. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

## 5) API pública vs API privada

Una **API pública** es la que expones a clientes externos, frontend, móvil o integradores. Necesita versionado cuidado, límites, validación robusta, autorización por recurso y protección contra abusos. Una **API privada** es para comunicación interna entre servicios propios. Puede no estar expuesta a internet y puede apoyarse en red interna, service mesh, allowlists o autenticación técnica entre servicios. OWASP insiste en que las APIs amplían la superficie de ataque y que el control de autorización por objeto y por función es crítico. ([owasp.org](https://owasp.org/API-Security/?utm_source=chatgpt.com "OWASP API Security Top 10"))

La trampa común es pensar que “privada” significa “no necesito securizarla”. Es al revés: aunque una API sea interna, sigue necesitando autenticación y autorización. OWASP remarca riesgos como Broken Object Level Authorization y otros fallos típicos en APIs que no desaparecen por estar detrás de la red interna. ([owasp.org](https://owasp.org/API-Security/editions/2023/en/0x11-t10/?utm_source=chatgpt.com "OWASP Top 10 API Security Risks – 2023"))

## 6) Cómo se transportan estas credenciales

En HTTP, las credenciales suelen ir en la cabecera `Authorization`. Para tokens se usa habitualmente `Authorization: Bearer <token>`. También verás API keys en cabeceras propias como `x-api-key`, aunque desde un punto de vista de interoperabilidad el ecosistema HTTP está muy orientado a `Authorization` y a mecanismos anunciados con `WWW-Authenticate`. MDN documenta el papel de estas cabeceras en la autenticación HTTP. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization?utm_source=chatgpt.com "Authorization header - HTTP | MDN - MDN Web Docs"))

A nivel de seguridad operativa, exponer claves en query string suele considerarse peor práctica que enviarlas en cabecera, porque se propagan con más facilidad a logs, proxies e historiales. Google recomienda evitar usar parámetros de consulta para enviar API keys. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?hl=es-419&utm_source=chatgpt.com "Prácticas recomendadas para administrar las claves de API"))

## 7) Ejemplos claros para tus apuntes

### Ejemplo A: clave “pública” de frontend

Un frontend necesita acceder a un endpoint poco sensible, con fuerte rate limit y restricciones por origen. La clave está restringida por dominio y por APIs concretas, y nunca da acceso a operaciones críticas. Si se filtra, el daño está acotado por esas restricciones. Esto encaja con la idea de “public key” operacional, no porque sea inocua, sino porque el diseño asume exposición parcial y compensa con controles. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

### Ejemplo B: clave privada server-to-server

Tu micro de actualización llama a un endpoint interno de la API para resolver y descargar firmware. La credencial vive solo en backend, se rota, se registra su uso y jamás se incrusta en cliente. Esto ya es una **private API key** de facto. Funciona bien si el número de servicios es pequeño y los permisos son simples. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

### Ejemplo C: token de aplicación

Un servicio interno obtiene un token con permisos como `firmware:resolve` y `firmware:download`, con expiración corta y audiencia concreta. La API valida firma, expiración y scopes antes de servir el recurso. Esto encaja mucho mejor cuando quieres crecimiento, separación fina de permisos y mejor trazabilidad. OAuth 2.0 contempla acceso que la aplicación obtiene por sí misma, y la práctica moderna refuerza este enfoque con guías de seguridad actuales. ([RFC Editor](https://www.rfc-editor.org/rfc/rfc6749?utm_source=chatgpt.com "RFC 6749: The OAuth 2.0 Authorization Framework"))

## 8) Errores típicos

Error uno: meter una API key privada en frontend, móvil, Electron o firmware visible. Error dos: reutilizar la misma clave para todos los entornos y todos los servicios. Error tres: no restringir la clave. Error cuatro: no rotarla. Error cinco: usar la misma credencial para identificar a un servicio y para representar permisos de usuario. Las buenas prácticas de gestión de claves insisten en restricciones, no subirlas a repositorios, eliminar las innecesarias y protegerlas tanto en reposo como en tránsito. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

Otro error muy típico es confiar solo en autenticación y olvidarte de autorización por recurso. Que una llamada venga de un servicio válido no implica que ese servicio deba acceder a cualquier objeto o workspace. OWASP destaca precisamente la autorización por objeto como una de las áreas más críticas en seguridad API. ([owasp.org](https://owasp.org/API-Security/editions/2023/en/0x11-t10/?utm_source=chatgpt.com "OWASP Top 10 API Security Risks – 2023"))

## 9) Qué usar en tu caso concreto

Para tu escenario API ↔ micro, la progresión sensata es esta. Primera fase: **API privada** con red interna restringida y **private API key** por servicio, rotación y logs. Segunda fase: pasar a **token de aplicación** con permisos concretos, expiración y quizá audience por servicio. Si el sistema va a crecer, la segunda opción suele salir más barata a medio plazo en mantenibilidad y seguridad. ([Google Cloud Documentation](https://docs.cloud.google.com/docs/authentication/api-keys-best-practices?utm_source=chatgpt.com "Best practices for managing API keys | Authentication | Google Cloud ..."))

Si además esa API sirve binarios o acciones sensibles, separa claramente endpoints públicos e internos, y aplica autorización a nivel de recurso: no basta con que “el micro sea válido”; también debe poder acceder a **ese** workspace, **ese** artefacto y **esa** acción. Esa distinción está muy alineada con el foco de OWASP en autorización fina para APIs. ([owasp.org](https://owasp.org/API-Security/editions/2023/en/0x11-t10/?utm_source=chatgpt.com "OWASP Top 10 API Security Risks – 2023"))

## 10) Regla mental para tomar apuntes

Una regla útil es esta. **API key**: credencial simple, buena para arranque controlado. **Private API key**: solo backend, nunca cliente. **Public API key**: expuesta o potencialmente expuesta, pero extremadamente restringida y nunca válida para operaciones sensibles. **Token de aplicación**: credencial más madura, temporal y expresiva. **API pública**: pensada para terceros o clientes. **API privada**: pensada para backend interno, pero igualmente securizada. Todo esto se monta sobre autenticación HTTP y autorización por recurso/función. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization?utm_source=chatgpt.com "Authorization header - HTTP | MDN - MDN Web Docs"))

## 11) Orden de estudio recomendado

Empieza por entender **HTTP Authorization** y cómo se envían credenciales. Luego estudia buenas prácticas de **API keys**. Después da el salto a **OAuth 2.0** y al concepto de acceso de aplicación a aplicación. Cierra con **OWASP API Security Top 10** para aprender qué suele romperse en la realidad. Ese orden va de “cómo viaja la credencial” a “cómo diseño el sistema sin agujeros”. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization?utm_source=chatgpt.com "Authorization header - HTTP | MDN - MDN Web Docs"))

## 12) Enlaces fiables para estudiar

Para base HTTP, MDN sobre `Authorization` y autenticación HTTP. Para claves, Google Cloud sobre gestión y buenas prácticas de API keys. Para diseño de tokens, el RFC de OAuth 2.0 y la guía de mejores prácticas de seguridad OAuth. Para seguridad de API real, OWASP API Security Project y API Security Top 10. Son fuentes muy usadas y suficientemente serias para tomar apuntes técnicos. ([MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization?utm_source=chatgpt.com "Authorization header - HTTP | MDN - MDN Web Docs"))

Si te va bien, en el siguiente mensaje te hago unos **apuntes tipo temario**, con títulos, definiciones cortas y ejemplos, listos para copiar en tu libreta.