# SecTools.Org: Las 125 mejores herramientas de seguridad de red

Durante más de una década, el Proyecto Nmap ha estado catalogando las herramientas favoritas de la comunidad de seguridad de red. En 2011, este sitio se volvió mucho más dinámico, ofreciendo calificaciones, reseñas, búsquedas, clasificaciones y un nuevo formulario de sugerencia de herramientas. Este sitio permite herramientas de código abierto y comerciales en cualquier plataforma, excepto aquellas herramientas que mantenemos (como Nmap Security Scanner, el conector de red Ncat y el manipulador de paquetes Nping).

Estamos muy impresionados por la inteligencia colectiva de la comunidad de seguridad y recomendamos encarecidamente leer la lista completa e investigar cualquier herramienta con la que no esté familiarizado. Haga clic en el nombre de cualquier herramienta para obtener más detalles sobre esa aplicación en particular, incluida la posibilidad de leer (y escribir) reseñas. Muchos elementos del sitio se explican mediante consejos de herramientas si pasa el mouse sobre ellos. ¡Disfrute!

**Ordenar por:** popularidad | calificación | fecha de lanzamiento

**Herramientas 1–10 de 125** | página siguiente →

---

### (1) ★★★★★ Wireshark (#1, 1)
**Wireshark** (conocido como Ethereal hasta una disputa de marca registrada en el verano de 2006) es un fantástico analizador de protocolos de red multiplataforma de código abierto. Le permite examinar datos de una red en vivo o de un archivo de captura en disco. Puede navegar interactivamente por los datos de captura, profundizando hasta el nivel de detalle del paquete que necesite. Wireshark tiene varias características poderosas, incluido un rico lenguaje de filtro de visualización y la capacidad de ver el flujo reconstruido de una sesión TCP. También admite cientos de protocolos y tipos de medios. Se incluye una versión de consola similar a tcpdump llamada tshark. Una advertencia es que Wireshark ha sufrido docenas de agujeros de seguridad explotables de forma remota, así que manténgase actualizado y tenga cuidado de ejecutarlo en redes no confiables u hostiles (como conferencias de seguridad). Leer 31 reseñas.

**Último lanzamiento:** versión 1.12.7 el 12 de agosto de 2015 (hace 10 años, 6 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** sniffers

### (2) ★★★★½ Metasploit (#2, 3)
**Metasploit** tomó el mundo de la seguridad por asalto cuando se lanzó en 2004. Es una plataforma avanzada de código abierto para desarrollar, probar y usar código de explotación. El modelo extensible a través del cual se pueden integrar cargas útiles, codificadores, generadores de no-op y exploits ha hecho posible utilizar el Marco Metasploit como una salida para la investigación de explotación de vanguardia. Se entrega con cientos de exploits, como puede ver en su lista de módulos. Esto hace que escribir sus propios exploits sea más fácil, y ciertamente supera el recorrer los rincones más oscuros de Internet en busca de shellcode ilícito de dudosa calidad. Un extra gratuito es Metasploitable, una máquina virtual Linux intencionalmente insegura que puede usar para probar Metasploit y otras herramientas de explotación sin afectar servidores en vivo.

Metasploit era completamente gratuito, pero el proyecto fue adquirido por Rapid7 en 2009 y pronto surgieron variantes comerciales. El Marco en sí sigue siendo gratuito y de código abierto, pero ahora también ofrecen una edición comunitaria gratuita pero limitada, una edición Express más avanzada ($5,000 por año por usuario) y una edición Pro con todas las funciones. Otras herramientas de explotación pagas a considerar son Core Impact (más caro) y Canvas (menos).

El Marco Metasploit ahora incluye una GUI oficial basada en Java y también el excelente Armitage de Raphael Mudge. Las ediciones Community, Express y Pro tienen GUI basadas en web. Leer 15 reseñas.

**Último lanzamiento:** versión 4.11 el 18 de diciembre de 2014 (hace 11 años, 1 mes).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** sploits

### (3) ★★★ Nessus (#3, 2)
**Nessus** es uno de los escáneres de vulnerabilidades más populares y capaces, particularmente para sistemas UNIX. Inicialmente era gratuito y de código abierto, pero cerraron el código fuente en 2005 y eliminaron la versión gratuita "Registered Feed" en 2008. Ahora cuesta $2,190 por año, lo que aún supera a muchos de sus competidores. También está disponible una versión gratuita "Nessus Home", aunque está limitada y solo tiene licencia para uso en red doméstica.

Nessus se actualiza constantemente, con más de 70,000 complementos. Las características clave incluyen verificaciones de seguridad remotas y locales (autenticadas), una arquitectura cliente/servidor con una interfaz basada en web y un lenguaje de scripting integrado para escribir sus propios complementos o comprender los existentes. Leer 20 reseñas.

**Último lanzamiento:** versión 6.3.3 el 16 de marzo de 2015 (hace 10 años, 11 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** vuln-scanners

### (4) ★★★★½ Aircrack (#4, 17)
**Aircrack** es un conjunto de herramientas para el craqueo de WEP y WPA de 802.11a/b/g. Implementa los algoritmos de craqueo más conocidos para recuperar claves inalámbricas una vez que se han recopilado suficientes paquetes cifrados. El conjunto comprende más de una docena de herramientas discretas, incluidas airodump (un programa de captura de paquetes 802.11), aireplay (un programa de inyección de paquetes 802.11), aircrack (craqueo estático WEP y WPA-PSK) y airdecap (descifra archivos de captura WEP/WPA). Leer 15 reseñas.

**Último lanzamiento:** versión 1.1 el 24 de abril de 2010 (hace 15 años, 9 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** pass-audit, wireless

### (5) ★★★★★ Snort (#5, 2)
Este sistema de detección y prevención de intrusiones en la red se destaca en el análisis de tráfico y el registro de paquetes en redes IP. A través del análisis de protocolos, la búsqueda de contenido y varios preprocesadores, Snort detecta miles de gusanos, intentos de explotación de vulnerabilidades, escaneos de puertos y otros comportamientos sospechosos. Snort utiliza un lenguaje flexible basado en reglas para describir el tráfico que debe recopilar o dejar pasar, y un motor de detección modular. También consulte el Basic Analysis and Security Engine (BASE) gratuito, una interfaz web para analizar las alertas de Snort.

Si bien Snort en sí es gratuito y de código abierto, la empresa matriz SourceFire ofrece sus reglas certificadas por VRT por $499 por sensor por año y una línea de productos complementaria de software y dispositivos con más funciones de nivel empresarial. Sourcefire también ofrece un feed gratuito con 30 días de retraso. Leer 2 reseñas.

**Último lanzamiento:** versión 2.9.7.5 el 23 de julio de 2015 (hace 10 años, 6 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** ids

### (6) ★★★½ Cain and Abel (#6, 3)
Los usuarios de UNIX a menudo afirman con presunción que las mejores herramientas de seguridad gratuitas son compatibles primero con su plataforma, y los puertos de Windows son a menudo una ocurrencia tardía. Por lo general tienen razón, pero **Cain & Abel** es una excepción evidente. Esta herramienta de recuperación de contraseñas solo para Windows maneja una enorme variedad de tareas. Puede recuperar contraseñas rastreando la red, descifrando contraseñas cifradas utilizando ataques de diccionario, fuerza bruta y criptoanálisis, grabando conversaciones VoIP, decodificando contraseñas codificadas, revelando cuadros de contraseña, descubriendo contraseñas en caché y analizando protocolos de enrutamiento. También está bien documentado. Leer 17 reseñas.

**Último lanzamiento:** versión 4.9.56 el 7 de abril de 2014 (hace 11 años, 10 meses).

* **Plataformas:** Windows
* **Categoría:** pass-audit, sniffers

### (7) ★★★★ BackTrack (#7, 25)
Esta excelente distribución de Linux en Live CD de arranque proviene de la fusión de Whax y Auditor. Cuenta con una gran variedad de herramientas de seguridad y análisis forense y proporciona un rico entorno de desarrollo. Se enfatiza la modularidad del usuario para que la distribución pueda ser fácilmente personalizada por el usuario para incluir scripts personales, herramientas adicionales, kernels personalizados, etc. BackTrack es sucedido por Kali Linux. Leer 22 reseñas.

**Último lanzamiento:** versión 5 R3 el 13 de agosto de 2012 (hace 13 años, 6 meses).

* **Plataformas:** Linux
* **Categoría:** sec-distros

### (8) ★★★★½ Netcat (#8, 4)
Esta sencilla utilidad lee y escribe datos a través de conexiones de red TCP o UDP. Está diseñada para ser una herramienta de back-end confiable para usar directamente o para ser manejada fácilmente por otros programas y scripts. Al mismo tiempo, es una herramienta de depuración y exploración de red rica en funciones, ya que puede crear casi cualquier tipo de conexión que necesite, incluida la vinculación de puertos para aceptar conexiones entrantes.

El Netcat original fue lanzado por Hobbit en 1995, pero no se ha mantenido a pesar de su popularidad. A veces incluso puede ser difícil encontrar una copia del código fuente v1.10. La flexibilidad y utilidad de esta herramienta impulsaron al Proyecto Nmap a producir Ncat, una reimplementación moderna que admite SSL, IPv6, SOCKS y proxies http, intermediación de conexiones y más. Otras versiones de esta herramienta clásica incluyen la increíblemente versátil Socat, nc de OpenBSD, Cryptcat, Netcat6, pnetcat, SBD y el llamado GNU Netcat. Leer 13 reseñas.

**Último lanzamiento:** versión 1.10 el 20 de marzo de 1996 (hace 29 años, 11 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** general, packet-crafters

### (9) ★★★★½ tcpdump (#9, 1)
**Tcpdump** es el sniffer de red que todos usábamos antes de que Wireshark entrara en escena, y muchos de nosotros continuamos usándolo con frecuencia. Puede que no tenga las campanas y silbatos (como una bonita GUI y lógica de análisis para cientos de protocolos de aplicación) que tiene Wireshark, pero hace bien el trabajo y con menos riesgo de seguridad. También requiere menos recursos del sistema. Si bien Tcpdump no recibe nuevas funciones a menudo, se mantiene activamente para corregir errores y problemas de portabilidad. Es ideal para rastrear problemas de red o monitorear la actividad. Hay un puerto de Windows separado llamado WinDump. tcpdump es la fuente de la biblioteca de captura de paquetes Libpcap/WinPcap, que es utilizada por Nmap y muchas otras herramientas. Leer 3 reseñas.

**Último lanzamiento:** versión 4.7.4 el 22 de abril de 2015 (hace 10 años, 9 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** sniffers

### (10) ★★★★★ John the Ripper (#10, sin cambios)
**John the Ripper** es un rápido descifrador de contraseñas para UNIX/Linux y Mac OS X. Su propósito principal es detectar contraseñas débiles de Unix, aunque también admite hashes para muchas otras plataformas. Hay una versión oficial gratuita, una versión mejorada por la comunidad (con muchos parches contribuidos pero no tanta garantía de calidad) y una versión pro económica. Probablemente querrá comenzar con algunas listas de palabras, que puede encontrar aquí, aquí o aquí. Leer 7 reseñas.

**Último lanzamiento:** versión 1.8.0 el 30 de mayo de 2013 (hace 12 años, 8 meses).

* **Plataformas:** Linux, OS X, Windows
* **Categoría:** pass-audit

---

### Categorías
* Antimalware (3)
* Escáneres específicos de aplicaciones (3)
* Relacionados con el navegador web (4)
* Herramientas de cifrado (8)
* Depuradores (5)
* Firewalls (2)
* Análisis forense (4)
* Fuzzers (4)
* Herramientas de propósito general (8)
* Sistemas de detección de intrusiones (6)
* Herramientas de creación de paquetes (6)
* Auditoría de contraseñas (12)
* Escáneres de puertos (4)
* Detectores de rootkit (5)
* Sistemas operativos orientados a la seguridad (5)
* Sniffers de paquetes (14)
* Herramientas de explotación de vulnerabilidades (11)
* Herramientas de monitoreo de tráfico (10)
* Escáneres de vulnerabilidades (11)
* Proxies web (4)
* Escáneres de vulnerabilidades web (20)
* Herramientas inalámbricas (5)
