# Visión General y Demostración de Nmap

A veces, la mejor manera de entender algo es verlo en acción. Esta sección incluye ejemplos de Nmap utilizados en circunstancias (en su mayoría) ficticias pero típicas. Los novatos en Nmap no deben esperar entender todo de una vez. Esto es simplemente una visión general amplia de las características que se describen en profundidad en capítulos posteriores. Las "soluciones" incluidas a lo largo de este libro demuestran muchas otras tareas comunes de Nmap para auditores de seguridad y administradores de redes.

## Avatar Online

Felix llega puntualmente al trabajo el 15 de diciembre, aunque no espera muchas tareas estructuradas. La pequeña firma de pruebas de penetración de San Francisco para la que trabaja ha estado tranquila últimamente debido a las inminentes vacaciones. Felix pasa las horas laborales persiguiendo su último pasatiempo de construir potentes antenas Wi-Fi para evaluaciones inalámbricas y exploración de *war driving*. Sin embargo, Felix espera más negocios. El hacking ha sido su pasatiempo y fascinación desde una infancia dedicada a aprender todo lo que pudo sobre redes, seguridad, Unix y sistemas telefónicos. Ocasionalmente, su curiosidad lo llevó demasiado lejos, y Felix casi fue arrastrado por las acusaciones de la Operación Sundevil de 1990. Afortunadamente, Felix salió de la adolescencia sin antecedentes penales, conservando su conocimiento experto de las debilidades de seguridad. Como profesional, es capaz de realizar los mismos tipos de intrusiones en la red que antes, pero con el beneficio añadido de la inmunidad contractual contra el enjuiciamiento ¡e incluso un cheque de pago! En lugar de mantener sus hazañas creativas en secreto, puede presumir de ellas ante la dirección del cliente al presentar sus informes. Así que Felix no se decepcionó cuando su jefe interrumpió su soldadura de antena para anunciar que el departamento de ventas había cerrado un acuerdo de pruebas de penetración con la compañía de juegos Avatar Online.

Avatar Online (AO) es una pequeña empresa que trabaja para crear la próxima generación de juegos de rol multijugador masivos en línea (MMORPG). Su producto, inspirado en el Metaverso imaginado en *Snow Crash* de Neil Stevenson, es fascinante pero aún altamente confidencial. Después de presenciar la filtración de alto perfil del código fuente del próximo juego de Valve Software, AO contrató rápidamente a los consultores de seguridad. La tarea de Felix es iniciar una evaluación de vulnerabilidad externa (desde fuera del firewall) mientras sus socios trabajan en la seguridad física, auditoría de código fuente, ingeniería social, y demás. A Felix se le permite explotar cualquier vulnerabilidad encontrada.

El primer paso en una evaluación de vulnerabilidad es el descubrimiento de la red. Esta etapa de reconocimiento determina qué rangos de direcciones IP está utilizando el objetivo, qué hosts están disponibles, qué servicios ofrecen esos hosts, detalles generales de la topología de la red y qué políticas de firewall/filtrado están vigentes.

Determinar los rangos de IP a escanear sería normalmente un proceso elaborado que involucraría búsquedas en ARIN (u otro registro geográfico), consultas DNS e intentos de transferencia de zona, varias técnicas de investigación web y más. Pero en este caso, Avatar Online especificó explícitamente qué redes querían que se probaran: la red corporativa en 6.209.24.0/24 y sus sistemas de producción/DMZ residentes en 6.207.0.0/22. Felix verifica los registros whois de IP de todos modos y confirma que estos rangos de IP están asignados a AO[1]. Felix decodifica subconscientemente la notación CIDR[2] y reconoce esto como 1.280 direcciones IP. Sin problema.

Siendo del tipo cuidadoso, Felix comienza primero con lo que se conoce como un escaneo de lista de Nmap (opción `-sL`). Esta característica simplemente enumera cada dirección IP en el bloque(s) de red objetivo dado y realiza una búsqueda inversa de DNS (a menos que se especifique `-n`) en cada una. Una razón para hacer esto primero es el sigilo. Los nombres de los hosts pueden insinuar vulnerabilidades potenciales y permitir una mejor comprensión de la red objetivo, todo sin levantar alarmas[3]. Felix está haciendo esto por otra razón: para verificar dos veces que los rangos de IP son correctos. El administrador de sistemas que proporcionó las IPs podría haber cometido un error, y escanear la compañía equivocada sería un desastre. ¡El contrato firmado con Avatar Online puede actuar como una tarjeta de "salga libre de la cárcel" para penetrar sus redes, pero no ayudará si Felix compromete accidentalmente el servidor de otra compañía! El comando que utiliza y un extracto de los resultados se muestran en el Ejemplo 1.1.

**Ejemplo 1.1. Escaneo de lista de Nmap contra direcciones IP de Avatar Online**
```
felix> nmap -sL 6.209.24.0/24 6.207.0.0/22

Starting Nmap ( https://nmap.org )
Nmap scan report for 6.209.24.0
Nmap scan report for fw.corp.avataronline.com (6.209.24.1)
Nmap scan report for dev2.corp.avataronline.com (6.209.24.2)
Nmap scan report for 6.209.24.3
Nmap scan report for 6.209.24.4
...
Nmap scan report for dhcp-21.corp.avataronline.com (6.209.24.21)
Nmap scan report for dhcp-22.corp.avataronline.com (6.209.24.22)
Nmap scan report for dhcp-23.corp.avataronline.com (6.209.24.23)
...
Nmap scan report for 6.207.0.0
Nmap scan report for gw.avataronline.com (6.207.0.1)
Nmap scan report for ns1.avataronline.com (6.207.0.2)
Nmap scan report for ns2.avataronline.com (6.207.0.3)
Nmap scan report for ftp.avataronline.com (6.207.0.4)
Nmap scan report for 6.207.0.5
Nmap scan report for 6.207.0.6
Nmap scan report for www.avataronline.com (6.207.0.7)
Nmap scan report for 6.207.0.8
...
Nmap scan report for cluster-c120.avataronline.com (6.207.2.120)
Nmap scan report for cluster-c121.avataronline.com (6.207.2.121)
Nmap scan report for cluster-c122.avataronline.com (6.207.2.122)
...
Nmap scan report for 6.207.3.255
Nmap done: 1280 IP addresses (0 hosts up) scanned in 331.49 seconds
felix>
```

Leyendo los resultados, Felix encuentra que todas las máquinas con entradas de DNS inverso se resuelven a Avatar Online. Ningún otro negocio parece compartir el espacio IP. Además, estos resultados le dan a Felix una idea aproximada de cuántas máquinas están en uso y una buena idea de para qué se utilizan muchas de ellas. Ahora está listo para ser un poco más intrusivo y probar un escaneo de puertos. Utiliza características de Nmap que intentan determinar la aplicación y el número de versión de cada servicio que escucha en la red. También solicita que Nmap intente adivinar el sistema operativo remoto a través de una serie de sondas TCP/IP de bajo nivel conocidas como identificación de SO (*OS fingerprinting*). Este tipo de escaneo no es en absoluto sigiloso, pero eso no le preocupa a Felix. Está interesado en si los administradores de AO siquiera notan estos escaneos flagrantes. Después de un poco de consideración, Felix se decide por el siguiente comando:

`nmap -sS -p- -PE -PP -PS80,443 -PA3389 -PU40125 -A -T4 -oA avatartcpscan-%D 6.209.24.0/24 6.207.0.0/22`

Estas opciones se describen en capítulos posteriores, pero aquí hay un resumen rápido de ellas.

**-sS**
Habilita la técnica eficiente de escaneo de puertos TCP conocida como escaneo SYN. Felix habría añadido una U al final si también quisiera hacer un escaneo UDP, pero está guardando eso para más tarde. El escaneo SYN es el tipo de escaneo predeterminado, pero indicarlo explícitamente no hace daño.

**-p-**
Solicita que Nmap escanee cada puerto del 1 al 65535. Esto es más completo que el valor predeterminado, que es escanear solo los 1.000 puertos que hemos encontrado más comúnmente accesibles en pruebas de Internet a gran escala. Este formato de opción es simplemente un atajo para `-p1-65535`. Felix podría haber especificado `-p0-65535` si quisiera escanear también el puerto cero, bastante ilegítimo. La opción `-p` tiene una sintaxis muy flexible, permitiendo incluso la especificación de un conjunto diferente de puertos UDP y TCP.

**-PE -PP -PS80,443 -PA3389 -PU40125**
Estas son todas técnicas de descubrimiento de hosts (tipos de ping) utilizadas en combinación para determinar qué objetivos en una red están realmente disponibles y evitar perder mucho tiempo escaneando direcciones IP que no están en uso. Este encantamiento particular envía paquetes de solicitud de eco y solicitud de marca de tiempo ICMP; paquetes TCP SYN a los puertos 80 y 443; paquetes TCP ACK al puerto 3389; y un paquete UDP al puerto 40,125. Si Nmap recibe una respuesta de un host objetivo a cualquiera de estas sondas, considera que el host está activo y disponible para escanear. Esta es la combinación de seis sondas más efectiva que hemos encontrado en pruebas empíricas a gran escala para el descubrimiento de hosts contra objetivos en Internet. Es más extensa que el valor predeterminado de Nmap, que es `-PE -PS443 -PA80 -PP`. En una situación de prueba de penetración, a menudo querrás escanear cada host incluso si no parecen estar activos. Después de todo, podrían estar fuertemente filtrados de tal manera que las sondas que seleccionaste se ignoren pero algún otro puerto oscuro pueda estar disponible. Para escanear cada IP ya sea que muestre un host disponible o no, especifique la opción `-Pn` en lugar de todo lo anterior. Felix inicia tal escaneo en segundo plano, aunque puede tardar muchas horas en completarse.

**-A**
Esta opción de atajo activa características Avanzadas y Agresivas como la detección de SO y servicios. En el momento de escribir esto, es equivalente a `-sV -sC -O --traceroute` (detección de versiones, Nmap Scripting Engine con el conjunto predeterminado de scripts, detección remota de SO y traceroute). Se pueden añadir más características a `-A` más adelante.

**-T4**
Ajusta la temporización al nivel agresivo (n.º 4 de 5). Esto es lo mismo que especificar `-T aggressive`, pero es más fácil de escribir y deletrear. En general, se recomienda la opción `-T4` si la conexión entre usted y las redes objetivo es razonablemente rápida y confiable.

**-oA avatartcpscan-%D**
Muestra los resultados en todos los formatos (normal, XML, grepeable) en archivos llamados avatartcpscan-<fecha>.<extensión> donde las extensiones son .nmap, .xml y .gnmap respectivamente. La <fecha> da el mes, día y año en un formato como 073010. Todos los formatos de salida incluyen la fecha y hora de inicio, pero a Felix le gusta anotar la fecha explícitamente en el nombre del archivo. La salida normal y los errores todavía se envían a stdout[4] también.

**6.209.24.0/24 6.207.0.0/22**
Estos son los bloques de red de Avatar Online discutidos anteriormente. Se dan en notación CIDR, pero Nmap permite especificarlos en muchos otros formatos. Por ejemplo, 6.209.24.0/24 podría especificarse en su lugar como 6.209.24.0-255.

Dado que un escaneo tan completo contra más de mil direcciones IP podría tardar un tiempo, Felix simplemente comienza a ejecutarlo y reanuda el trabajo en su antena Yagi. Un par de horas más tarde nota que ha terminado y echa un vistazo a los resultados. El Ejemplo 1.2 muestra una de las máquinas descubiertas.

**Ejemplo 1.2. Resultados de Nmap contra un firewall de AO**
```
Nmap scan report for fw.corp.avataronline.com (6.209.24.1)
(The 65530 ports scanned but not shown below are in state: filtered)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 3.7.1p2 (protocol 1.99)
| ssh-hostkey: 1024 7c:14:2f:92:ca:61:90:a4:11:3c:47:82:d5:8e:a9:6b (DSA)
|_2048 41:cf7d:839d:7f66:0ae1:8331:7fd4:5a97:5a (RSA)
|_sshv1: Server supports SSHv1
53/tcp   open   domain     ISC BIND 9.2.1
110/tcp  open   pop3       Courier pop3d
113/tcp  closed auth
143/tcp  open   imap       Courier Imap 1.6.X - 1.7.X
3128/tcp open   http-proxy Squid webproxy 2.2.STABLE5
Device type: general purpose
Running: Linux 2.4.X|2.5.X
OS details: Linux Kernel 2.4.0 - 2.5.20
Uptime 3.134 days
```

Para el ojo entrenado, esto transmite información sustancial sobre la postura de seguridad de AO. Felix primero nota el nombre DNS inverso: esta máquina aparentemente está destinada a ser un firewall para su red corporativa. La siguiente línea es importante, pero con demasiada frecuencia se ignora. Indica que la gran mayoría de los puertos en esta máquina están en el estado filtrado (filtered). Esto significa que Nmap no puede llegar al puerto porque está bloqueado por reglas de firewall. El hecho de que todos los puertos excepto unos pocos elegidos estén en este estado es una señal de competencia en seguridad. Denegar por defecto es un mantra de seguridad por buenas razones: significa que incluso si alguien dejara accidentalmente SunRPC (puerto 111) abierto en esta máquina, las reglas del firewall evitarían que los atacantes se comunicaran con él.

Felix luego mira cada línea de puerto a su vez. El primer puerto es Secure Shell (OpenSSH). La versión 3.7.1p2 es común, ya que muchos administradores actualizaron a esta versión debido a errores de gestión de búfer potencialmente explotables que afectaban a versiones anteriores. Nmap también nota que el script NSE `sshv1` informa que el protocolo SSHv1 menos seguro es compatible. Un administrador de sistemas verdaderamente paranoico solo permitiría conexiones SSH desde ciertas direcciones IP de confianza, pero se puede argumentar a favor del acceso abierto en caso de que el administrador necesite acceso de emergencia mientras está lejos de casa. La seguridad a menudo implica compensaciones, y esta puede ser justificable. Felix toma nota de probar el cracker de autenticación de red por fuerza bruta Ncrack y especialmente su herramienta privada de enumeración de usuarios SSH basada en temporización contra el servidor.

Felix también nota el puerto 53. Está ejecutando ISC BIND, que tiene una larga historia de agujeros de seguridad explotables remotamente. Visite la página de seguridad de BIND para más detalles. BIND 9.2.1 incluso tiene un desbordamiento de búfer potencialmente explotable, aunque la compilación predeterminada no es vulnerable. Felix verifica y encuentra que este servidor no es vulnerable al problema de libbind, pero eso es irrelevante. Este servidor casi seguramente no debería estar ejecutando un servidor de nombres accesible externamente. Un firewall solo debería ejecutar lo esencial para minimizar el riesgo de un compromiso desastroso. Además, este servidor no es autoritativo para ningún dominio: los servidores de nombres reales están en la red de producción. Un administrador probablemente solo pretendía que los clientes dentro del firewall contactaran con este servidor de nombres, pero no se molestó en bloquearlo solo para la interfaz interna. Felix intentará más tarde recopilar información importante de este servidor innecesario utilizando solicitudes de transferencia de zona y consultas intrusivas. También puede intentar el envenenamiento de caché. Falsificando la IP de windowsupdate.microsoft.com u otro servidor de descarga importante, Felix puede ser capaz de engañar a usuarios clientes internos desprevenidos para que ejecuten un programa troyano que le proporcione acceso total a la red detrás del firewall.

Los siguientes dos puertos abiertos son 110 (POP3) y 143 (IMAP). Tenga en cuenta que el 113 (auth) entre ellos está cerrado en lugar de abierto. POP3 e IMAP son servicios de recuperación de correo que, como BIND, no tienen lugar legítimo en este servidor. También son un riesgo de seguridad ya que generalmente transfieren el correo y (aún peor) las credenciales de autenticación sin cifrar. Los usuarios probablemente deberían conectarse por VPN y revisar su correo desde un servidor interno. Estos puertos también podrían estar envueltos en cifrado SSL. Nmap habría listado entonces los servicios como ssl/pop3 y ssl/imap. Felix intentará sus ataques de enumeración de usuarios y adivinación de contraseñas en estos servicios, que probablemente serán mucho más efectivos que contra SSH.

El último puerto abierto es un proxy Squid. Este es otro servicio que puede haber sido destinado para uso interno del cliente y no debería ser accesible desde el exterior (y particularmente no en el firewall). La opinión inicialmente positiva de Felix sobre los administradores de seguridad de AO cae aún más. Felix probará si puede abusar de este proxy para conectarse a otros sitios en Internet. Los spammers y los hackers maliciosos a menudo usan proxies de esta manera para ocultar sus huellas. Aún más crítico, Felix intentará usar el proxy para entrar en la red interna. Este ataque común es cómo Adrian Lamo entró en la red interna del New York Times en 2002. Lamo fue atrapado después de llamar a reporteros para presumir de sus hazañas contra el NY Times y otras compañías en detalle.

Las siguientes líneas revelan que esta es una caja Linux, lo cual es información valiosa al intentar la explotación. El bajo tiempo de actividad de tres días fue detectado durante la identificación del SO enviando varias sondas para el valor de la opción de marca de tiempo TCP y extrapolando la línea de vuelta a cero.

Felix luego examina la salida de Nmap para otra máquina, como se muestra en el Ejemplo 1.3.

**Ejemplo 1.3. Otra máquina interesante de AO**
```
Nmap scan report for dhcp-23.corp.avataronline.com (6.209.24.23)
(The 65526 ports scanned but not shown below are in state: closed)
PORT      STATE    SERVICE       VERSION
135/tcp   filtered msrpc
136/tcp   filtered profile
137/tcp   filtered netbios-ns
138/tcp   filtered netbios-dgm
139/tcp   filtered netbios-ssn
445/tcp   open     microsoft-ds  Microsoft Windows XP microsoft-ds
1002/tcp  open     windows-icfw?
1025/tcp  open     msrpc         Microsoft Windows msrpc
16552/tcp open     unknown
Device type: general purpose
Running: Microsoft Windows NT/2K/XP
OS details: Microsoft Windows XP Professional RC1+ through final release

Host script results:
|_nbstat: NetBIOS name: TRACYD, NetBIOS user: <unknown>,  NetBIOS MAC: 00:20:35:00:29:a2:7f (IBM)
|_smbv2-enabled: Server doesn't support SMBv2 protocol
| smb-os-discovery:  
|   OS: Windows XP (Windows 2000 LAN Manager)
|_  Name: WORKGROUP\JOHND
```

Felix sonríe cuando ve esta caja Windows XP en la red. Gracias a una avalancha de vulnerabilidades MS RPC, esas máquinas son triviales de comprometer si los parches del SO no están actualizados. La segunda línea muestra que el estado predeterminado es cerrado (closed), lo que significa que el firewall no tiene la misma política de denegar por defecto para esta máquina que para sí mismo. En su lugar, intentaron bloquear específicamente los puertos de Windows que consideran peligrosos en 135-139. Este filtro es lamentablemente inadecuado, ya que MS exporta la funcionalidad MS RPC en muchos otros puertos en Windows XP. Los puertos TCP 445 y 1025 son dos ejemplos de este escaneo. Mientras que Nmap no reconoció el 16552, Felix ha visto este patrón lo suficiente como para saber que probablemente es el Servicio de Mensajería de MS. Si AO hubiera estado usando filtrado de denegar por defecto, el puerto 16552 no sería accesible en primer lugar. Mirando a través de la página de resultados, Felix ve varias otras máquinas Windows en esta red DHCP. Felix no puede esperar para probar su exploit DCOM RPC favorito contra ellas. Fue escrito por HD Moore y está disponible en http://www.metasploit.com/tools/dcom.c. Si eso falla, hay un par de vulnerabilidades MS RPC más nuevas que probará.

Felix continúa estudiando detenidamente los resultados en busca de vulnerabilidades que pueda aprovechar para comprometer la red. En la red de producción, ve que gw.avataronline.com es un router Cisco que también actúa como un firewall rudimentario para los sistemas. Caen en la trampa de bloquear solo puertos privilegiados (aquellos por debajo de 1024), lo que deja un montón de vulnerabilidades SunRPC y otros servicios accesibles en esa red. Las máquinas con nombres como clust-* tienen cada una docenas de puertos abiertos que Nmap no reconoce. Probablemente son demonios personalizados ejecutando el motor de juego de AO. www.avataronline.com es una caja Linux con un servidor Apache abierto en los puertos HTTP y HTTPS. Desafortunadamente, está vinculado con una versión explotable de la biblioteca OpenSSL. ¡Ups! Antes de que se ponga el sol, Felix ha obtenido acceso privilegiado a hosts tanto en la red corporativa como en la de producción.

Como Felix ha demostrado, Nmap es frecuentemente utilizado por auditores de seguridad y administradores de redes para ayudar a localizar vulnerabilidades en redes de clientes/corporativas. Los capítulos siguientes describen las técnicas utilizadas por Felix, así como muchas otras características de Nmap, con mucho mayor detalle.

## Salvando a la Raza Humana

**Figura 1.1. Trinity comienza su asalto**
*(Imagen: Trinity comienza su asalto)*

¡Trinity está en un buen lío! Habiendo descubierto que el mundo que damos por sentado es realmente una "Matrix" virtual dirigida por señores supremos mecánicos, Trinity decide contraatacar y liberar a la raza humana de esta esclavitud mental. Para empeorar las cosas, su colonia subterránea de humanos liberados (Zion) está bajo ataque por 250.000 poderosos centinelas alienígenas. Su única esperanza implica desactivar el sistema de energía de emergencia para 27 manzanas de la ciudad en menos de cinco minutos. El equipo anterior murió intentándolo. En los momentos más sombríos de la vida cuando toda esperanza parece perdida, ¿a qué deberías acudir? ¡A Nmap, por supuesto! Pero no todavía.

Primero debe derrotar la seguridad perimetral, que en muchas redes implica firewalls y sistemas de detección de intrusos (IDS). Ella es muy consciente de las técnicas avanzadas para eludir estos dispositivos (cubiertas más adelante en este libro). Desafortunadamente, los administradores del sistema de energía de emergencia sabían que no debían conectar un sistema tan crítico a Internet, ni siquiera indirectamente. Ninguna cantidad de enrutamiento de origen o escaneo falsificado de IP ID ayudará a Trinity a superar esta seguridad de "brecha de aire" (*air gap*). Pensando rápido, idea un plan inteligente que implica saltar con su motocicleta desde la azotea de un edificio cercano, aterrizar en el puesto de guardia de la estación de energía y luego golpear a todos los guardias de seguridad. Esta técnica avanzada no está cubierta en ningún manual de seguridad física, pero resulta altamente efectiva. Demuestra cómo los hackers inteligentes investigan e idean sus propios ataques, en lugar de utilizar siempre el enfoque de *script-kiddie* de exploits enlatados.

Trinity lucha en su camino hacia la sala de ordenadores y se sienta en una terminal. Rápidamente determina que la red está utilizando el espacio de direcciones de red privada 10.0.0.0/8. Un ping a la dirección de red genera respuestas de docenas de máquinas. Un escaneo de ping de Nmap habría proporcionado una lista más completa de máquinas disponibles, pero usar la técnica de transmisión ahorró preciosos segundos. Luego saca Nmap[5]. La terminal tiene instalada la versión 2.54BETA25. Esta versión es antigua (2001) y menos eficiente que los lanzamientos más nuevos, pero Trinity no tenía tiempo para instalar una versión mejor desde el futuro. Este trabajo no tomará mucho tiempo de todos modos. Ella ejecuta el comando `nmap -v -sS -O 10.2.1.3`. Esto ejecuta un escaneo TCP SYN y detección de SO contra 10.2.1.3 y proporciona una salida detallada. El host parece ser un desastre de seguridad: AIX 3.2 con más de una docena de puertos abiertos. Desafortunadamente, esta no es la máquina que necesita comprometer. Así que ejecuta el mismo comando contra 10.2.2.2. Esta vez el SO objetivo no es reconocido (¡debería haber actualizado Nmap!) y solo tiene el puerto 22 abierto. Este es el servicio de administración cifrada Secure Shell. Como cualquier diosa hacker vestida de látex sabe, muchos servidores SSH de esa época (2001) tienen una vulnerabilidad explotable en el detector de ataques de compensación CRC32. Trinity saca un exploit de código completamente en ensamblador y lo utiliza para cambiar la contraseña de root de la caja objetivo a Z10N0101. Trinity usa contraseñas mucho más seguras bajo circunstancias normales. Ella inicia sesión como root y emite un comando para desactivar el sistema de energía de respaldo de emergencia para 27 manzanas de la ciudad, ¡terminando justo a tiempo! Aquí hay una toma de la acción:

**Figura 1.2. Trinity escanea la Matrix**
*(Imagen: Trinity escanea la Matrix)*

Un video de vista de terminal que muestra todo el hack está disponible en https://nmap.org/movies.html, a menos que la MPAA se entere y envíe centinelas o abogados tras nosotros.

## MadHat en el País de las Maravillas

Esta historia difiere de las anteriores en que es realmente cierta. Escrita por el usuario frecuente y colaborador de Nmap Lee "MadHat" Heath, describe cómo mejoró y personalizó Nmap para el uso diario en una gran empresa. En el verdadero espíritu de código abierto, ha lanzado estos valiosos scripts en su sitio web. Las direcciones IP han sido cambiadas para proteger la identidad corporativa. El resto de esta sección está en sus propias palabras.

Después de pasar las últimas dos décadas aprendiendo computadoras y abriéndome camino desde el soporte técnico, pasando por administrador de sistemas y llegando a mi trabajo soñado de Oficial de Seguridad de la Información para una importante compañía de Internet, me encontré con un problema. Se me entregó la responsabilidad exclusiva del monitoreo de seguridad para todo nuestro espacio IP. Esto era casi 50.000 hosts en todo el mundo cuando comencé hace varios años, y se ha duplicado desde entonces.

Escanear todas estas máquinas en busca de posibles vulnerabilidades como parte de evaluaciones mensuales o trimestrales sería bastante difícil, pero la gerencia quería que se hiciera diariamente. Los atacantes no esperarán una semana o un mes para explotar una vulnerabilidad recién expuesta, así que no puedo esperar tanto tiempo para encontrarla y parchearla tampoco.

Buscando herramientas, rápidamente elegí Nmap como mi escáner de puertos. Es ampliamente considerado como el mejor escáner, y ya lo había estado usando durante años para solucionar problemas de redes y probar la seguridad. A continuación, necesitaba software para agregar la salida de Nmap e imprimir las diferencias entre ejecuciones. Consideré varias herramientas existentes, incluida Nlog de HD Moore. Desafortunadamente, ninguna de estas monitoreaba los cambios de la manera que deseaba. Tenía que saber cuándo una lista de control de acceso de enrutador o firewall estaba mal configurada o un host estaba compartiendo públicamente contenido inapropiado. También me preocupaba la escalabilidad de estas otras soluciones, así que decidí abordar el problema yo mismo.

El primer problema que surgió fue la velocidad. Nuestras redes están ubicadas en todo el mundo, sin embargo, se me proporcionó solo un host con sede en EE. UU. para hacer el escaneo. En muchos casos, los firewalls entre los sitios ralentizaron el escaneo significativamente. Escanear los 100.000 hosts tomó más de 30 horas, lo cual es inaceptable para un escaneo diario. Así que escribí un script llamado `nmap-wrapper` que ejecuta docenas de procesos Nmap en paralelo, reduciendo el tiempo de escaneo a quince horas, incluso incluyendo la detección de SO.

El siguiente problema fue lidiar con tanta información. Una base de datos SQL parecía el mejor enfoque por razones de escalabilidad y minería de datos, pero tuve que abandonar esa idea debido a las presiones de tiempo. Una versión futura puede añadir este soporte. En su lugar, utilicé un archivo plano para almacenar los resultados de cada rango de direcciones clase C para cada día. La forma más poderosa y extensible de analizar y almacenar esta información era el formato XML de Nmap, pero elegí el formato "grepeable" (opción `-oG`) porque es muy fácil de analizar desde scripts simples. Las marcas de tiempo por host también se almacenan para fines de informes. Estas han demostrado ser bastante útiles cuando los administradores intentan culpar al escáner de caídas de máquinas o servicios. No pueden afirmar creíblemente una caída del servicio a las 7:12 AM cuando tengo pruebas de que el escaneo se ejecutó a las 9:45 AM.

El escaneo produce datos copiosos, sin un método de acceso conveniente. La herramienta estándar `diff` de Unix no es lo suficientemente inteligente como para informar solo los cambios que me importan, así que escribí un script en Perl llamado `nmap-diff` para proporcionar informes de cambios diarios. Un informe de salida típico se muestra en el Ejemplo 1.4.

**Ejemplo 1.4. Salida típica de nmap-diff**
```
> nmap-diff.pl -c3
  5 IPs showed changes

  10.12.4.8 (ftp-box.foocompany.biz)
         21/tcp   open   ftp
         80/tcp   open   http
        443/tcp   open   https
       1027/tcp   open   IIS
     + 1029/tcp   open   ms-lsa
      38292/tcp   open   landesk-cba 
  OS: Microsoft Windows Millennium Edition (Me)
      Windows 2000 Professional or Advanced Server
      or Windows XP

  10.16.234.3 (media.foocompany.biz)
         80/tcp   open   http
     +  554/tcp   open   rtsp
     + 7070/tcp   open   realserver

  192.168.10.186 (testbox.foocompany.biz)
    + 8082/tcp   open   blackice-alerts
  OS: Linux Kernel 2.4.0 - 2.5.20

  172.24.12.58 (mtafoocompany.biz)
    +   25/tcp   open   smtp 
  OS: FreeBSD 4.3 - 4.4PRERELEASE

  172.23.76.22 (media2.foocorp.biz)
         80/tcp   open   http
       1027/tcp   open   IIS
     + 1040/tcp   open   netsaint
       1755/tcp   open   wms 
       3372/tcp   open   msdtc
       6666/tcp   open   irc-serv
       7007/tcp   open   afs3-bos
  OS: Microsoft Windows Millennium Edition (Me)
      Windows 2000 Professional or Advanced Server
      or Windows XP
```

La gerencia y el personal quedaron impresionados cuando demostré este nuevo sistema en un simposio interno de seguridad de la empresa. Pero en lugar de permitirme dormirme en los laureles, comenzaron a pedir nuevas características. Querían recuentos de servidores de correo y web, estimaciones de crecimiento y más. Todos estos datos estaban disponibles en los escaneos, pero eran difíciles de acceder. Así que creé otro script en Perl, `nmap-report`, que hizo que consultar los datos fuera mucho más fácil. Toma especificaciones como puertos abiertos o sistemas operativos y encuentra todos los sistemas que coincidieron en un día determinado.

Un problema con este enfoque para el monitoreo de seguridad es que los empleados no siempre colocan servicios en sus puertos oficiales registrados por IANA. Por ejemplo, podrían poner un servidor web en el puerto 22 (SSH) o viceversa. Justo cuando estaba debatiendo cómo abordar este problema, Nmap salió con un sistema avanzado de detección de servicios y versiones (ver Capítulo 7, Detección de Versiones de Servicios y Aplicaciones). `nmap-report` ahora tiene una característica de reescaneo que usa el escaneo de versiones para informar los verdaderos servicios en lugar de adivinar basándose en el número de puerto. Espero integrar aún más la detección de versiones en futuras versiones. El Ejemplo 1.5 muestra `nmap-report` listando servidores FTP.

**Ejemplo 1.5. Ejecución de nmap-report**
```
> nmap-report -p21 -rV
[...]
172.21.199.76 (ftp1.foocorp.biz)
   21/tcp   open   ssl|ftp Serv-U ftpd 4.0

192.168.12.56 (ftp2.foocorp.biz)
   21/tcp   open   ftp     NcFTPd

 192.168.13.130 (dropbox.foocorp.biz)
   21/tcp   open   ftp     WU-FTPD 6.00LS
```

Aunque lejos de ser perfectos, estos scripts han demostrado ser bastante valiosos para monitorear grandes redes en busca de cambios que afecten a la seguridad. Dado que Nmap en sí es de código abierto, solo parecía justo lanzar mis scripts al público también. Los he puesto a disposición gratuitamente en http://www.unspecific.com/nmap.

---

[1] Estas direcciones IP están realmente registradas en el Campo de Pruebas de Yuma del Ejército de los Estados Unidos, que se utiliza para probar una amplia variedad de artillería, misiles, tanques y otras armas mortales. La moraleja es tener mucho cuidado con a quién escaneas, para no golpear accidentalmente una red altamente sensible. Los resultados del escaneo en esta historia no son realmente de este rango de IP.

[2] La notación de Enrutamiento Inter-Dominios sin Clases (CIDR) es un método para describir redes con más granularidad que la notación de clase A (CIDR /8), clase B (CIDR /16) o clase C (CIDR /24). Ver https://es.wikipedia.org/wiki/Classless_Inter-Domain_Routing.

[3] Es posible que el servidor de nombres objetivo registre un grupo sospechoso de consultas de DNS inverso desde el servidor de nombres de Felix, pero la mayoría de las organizaciones ni siquiera mantienen tales registros, y mucho menos los analizan.

[4] stdout es la notación "C" para representar el mecanismo de salida estándar para un sistema, como al xterm de Unix o la ventana de comandos de Windows en la que se inició Nmap.

[5] Una atacante sexy vestida de cuero del equipo anterior en realidad comenzó la sesión. No está claro en qué momento murió y dejó las tareas restantes a Trinity.
