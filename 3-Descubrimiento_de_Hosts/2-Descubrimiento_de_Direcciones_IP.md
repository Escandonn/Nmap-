# Descubrimiento de las direcciones IP de una organización

Nmap automatiza muchos aspectos del escaneo de redes, pero aún debes indicarle qué redes escanear. Supongo que podrías especificar `-iR` y esperar que Nmap alcance a tu empresa objetivo al azar, o podrías intentar el método de fuerza bruta especificando `0.0.0.0/0` para escanear toda la Internet. Pero cualquiera de esas opciones podría tomar meses o años, y posiblemente meterte en problemas. Por lo tanto, es importante investigar cuidadosamente los bloques de red (netblocks) de destino antes de escanearlos. Incluso si estás realizando una prueba de penetración legítima y el cliente te dio una lista de sus bloques de red, es importante verificarlos dos veces. Los clientes a veces tienen registros desactualizados o simplemente los anotan mal. Una carta de autorización firmada por tu cliente no ayudará si accidentalmente irrumpes en la empresa equivocada.

En muchos casos, comienzas solo con el nombre de dominio de una empresa. Esta sección demuestra algunas de las formas más comunes y efectivas de convertir eso en una lista de bloques de red de los cuales la empresa objetivo es propietaria, operadora o está afiliada. Se demuestran utilidades típicas de la línea de comandos de Linux, pero herramientas similares están disponibles para otras plataformas.

En la conferencia ShmooCon en 2006, un compañero se me acercó y se quejó de que la documentación de Nmap especificaba muchas formas de ejemplo para escanear `target.com`. Señaló que el ICANN había reservado el nombre de dominio `example.com` para este propósito, y me presionó para revisar la página del manual en consecuencia. Aunque técnicamente tenía razón, era algo extraño con lo que obsesionarse. Su motivación quedó clara cuando me entregó su tarjeta de presentación:

**Figura 3.1. Una tarjeta de presentación lo explica todo**

Aparentemente, muchos usuarios de Nmap copiaban ejemplos directamente de la página del manual y los ejecutaban sin cambiar el especificador de objetivo. Así que `target.com` se inundó de escaneos y las correspondientes alertas de IDS. En honor a ese incidente, el objetivo de esta sección es determinar los rangos de IP asignados y utilizados por Target Corporation.

## Trucos de DNS
El propósito principal del DNS es resolver nombres de dominio en direcciones IP, por lo que es un lugar lógico para comenzar. En el Ejemplo 3.1, utilizo el comando `host` de Linux para consultar algunos tipos de registros DNS comunes.

**Ejemplo 3.1. Uso del comando host para consultar tipos de registros DNS comunes**
```bash
> host -t ns target.com
target.com name server ns4.target.com.
target.com name server ns3.target.com.
target.com name server ns1-auth.sprintlink.net.
target.com name server ns2-auth.sprintlink.net.
target.com name server ns3-auth.sprintlink.net.
> host -t a target.com
target.com has address 161.225.130.163
target.com has address 161.225.136.0
> host -t aaaa target.com
target.com has no AAAA record
> host -t mx target.com
target.com mail is handled by 50 smtp02.target.com.
target.com mail is handled by 5 smtp01.target.com.
> host -t soa target.com
target.com has SOA record extdns02.target.com. hostmaster.target.com.
```

A continuación, resuelvo las direcciones IP de los nombres de host anteriores (usando `host` nuevamente) y pruebo algunos nombres de subdominio comunes como `www.target.com` y `ftp.target.com`. Comenzando con nombres como `ns3.target.com` y `smtp01.target.com`, intento cambiar los dígitos para encontrar nuevas máquinas. Todo esto me deja con los siguientes nombres y direcciones de `target.com`:

**Tabla 3.1. Primera pasada para listar las IP de target.com**

| Nombre de host | Direcciones IP |
| :--- | :--- |
| ns3.target.com | 161.225.130.130 |
| ns4.target.com | 161.225.136.136 |
| ns5.target.com | 161.225.130.150 |
| target.com | 161.225.136.0, 161.225.130.163 |
| smtp01.target.com | 161.225.140.120 |
| smtp02.target.com | 198.70.53.234, 198.70.53.235 |
| extdns02.target.com | 172.17.14.69 |
| www.target.com | 207.171.166.49 |

Si bien se puede generar una lista sustancial de nombres de host de esta manera, la "veta principal" de nombres de host proviene de una transferencia de zona. La mayoría de los servidores DNS ahora rechazan las solicitudes de transferencia de zona, pero vale la pena intentarlo porque muchos todavía lo permiten. Asegúrate de probar cada servidor DNS que hayas encontrado a través de los registros NS del dominio y el escaneo de puertos de los rangos de IP corporativos. Hasta ahora hemos encontrado siete servidores de nombres de Target: `ns3.target.com`, `ns4.target.com`, `ns5.target.com`, `ns1-auth.sprintlink.net`, `ns2-auth.sprintlink.net`, `ns3-auth.sprintlink.net` y `extdns02.target.com`. Desafortunadamente, todos esos servidores rechazaron la transferencia o no admitieron las conexiones DNS TCP requeridas para una transferencia de zona. El Ejemplo 3.2 muestra un intento fallido de transferencia de zona de `target.com` utilizando la herramienta común `dig` (domain information groper), seguido de uno exitoso contra una organización no relacionada (`cpsr.org`).

**Ejemplo 3.2. Fracaso y éxito de la transferencia de zona**
```bash
> dig @ns2-auth.sprintlink.net -t AXFR target.com
; <<>> DiG 9.5.0b3 <<>> @ns2-auth.sprintlink.net -t AXFR target.com

; Transfer failed.

> dig @ns2.eppi.com -t AXFR cpsr.org
; <<>> DiG 9.5.0b1 <<>> @ns2.eppi.com -t AXFR cpsr.org

cpsr.org             10800   IN      SOA   ns1.findpage.com. root.cpsr.org.
cpsr.org.            10800   IN      NS    ns.stimpy.net.
cpsr.org.            10800   IN      NS    ns1.findpage.com.
cpsr.org.            10800   IN      NS    ns2.eppi.com.
cpsr.org.            10800   IN      A     208.96.55.202
cpsr.org.            10800   IN      MX    0 smtp.electricembers.net.
diac.cpsr.org.       10800   IN      A     64.147.163.10
groups.cpsr.org.     10800   IN      NS    ns1.electricembers.net.
localhost.cpsr.org.  10800   IN      A     127.0.0.1
mail.cpsr.org.       10800   IN      A     209.209.81.73
peru.cpsr.org.       10800   IN      A     208.96.55.202
www.peru.cpsr.org.   10800   IN      A     208.96.55.202
[...]
```

Un error común al recopilar resultados de DNS directo como estos es asumir que todos los sistemas encontrados bajo un nombre de dominio deben ser parte de la red de esa organización y seguros para escanear. De hecho, nada impide que una organización agregue registros que apunten a cualquier lugar de Internet. Esto se hace comúnmente para subcontratar servicios a terceros manteniendo el nombre de dominio de origen para fines de marca. Por ejemplo, `www.target.com` se resuelve en `207.171.166.49`. ¿Es esto parte de la red de Target o es administrado por un tercero que quizás no queramos escanear? Tres pruebas rápidas y fáciles son la resolución inversa de DNS, traceroute y whois contra el registro de direcciones IP correspondiente. Los dos primeros pasos pueden ser realizados por Nmap, mientras que el comando `whois` de Linux funciona bien para el tercero. Estas pruebas contra `target.com` se muestran en el Ejemplo 3.3 y el Ejemplo 3.4.

**Ejemplo 3.3. Resolución inversa de DNS y escaneo de traceroute con Nmap contra www.target.com**
```bash
# nmap -Pn -T4 --traceroute www.target.com

Starting Nmap ( https://nmap.org )
Nmap scan report for 166-49.amazon.com (207.171.166.49)
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

TRACEROUTE (using port 80/tcp)
HOP RTT    ADDRESS
[cut]
9   84.94  ae-2.ebr4.NewYork1.Level3.net (4.69.135.186)
10  87.91  ae-3.ebr4.Washington1.Level3.net (4.69.132.93)
11  94.80  ae-94-94.csw4.Washington1.Level3.net (4.69.134.190)
12  86.40  ae-21-69.car1.Washington3.Level3.net (4.68.17.7)
13  185.10 AMAZONCOM.car1.Washington3.Level3.net (4.71.204.18)
14  84.70  72.21.209.38
15  85.73  72.21.193.37
16  85.68  166-49.amazon.com (207.171.166.49)

Nmap done: 1 IP address (1 host up) scanned in 20.57 seconds
```

**Ejemplo 3.4. Uso de whois para encontrar al propietario de la dirección IP de www.target.com**
```bash
> whois 207.171.166.49
[Querying whois.arin.net]
[whois.arin.net]

OrgName:    Amazon.com, Inc. 
OrgID:      AMAZON-4
Address:    605 5th Ave S
City:       SEATTLE
StateProv:  WA
PostalCode: 98104
Country:    US
[...]
```

En el Ejemplo 3.3, el DNS inverso (en dos lugares) e interesantes resultados de traceroute están resaltados. El nombre de dominio `Amazon.com` hace muy probable que el sitio web sea gestionado por Amazon en lugar de por la propia Target. Luego, los resultados de whois que muestran a "Amazon.com, Inc." como propietaria del espacio de IP eliminan toda duda. El sitio web tiene la marca de Target, pero muestra "Powered by Amazon.com" en la parte inferior. Si Target nos contratara para probar su seguridad, necesitaríamos el permiso por separado de Amazon para tocar este espacio de direcciones.

Las bases de datos web también se pueden utilizar para encontrar nombres de host bajo un dominio determinado. Por ejemplo, Netcraft tiene una función de búsqueda DNS en el sitio web [http://searchdns.netcraft.com/?host](http://searchdns.netcraft.com/?host). Escribir `.target.com` en el formulario trae 36 resultados, como se muestra en la Figura 3.2. Su práctica tabla también muestra al propietario del bloque de red, lo que detecta casos como el de Amazon ejecutando `www.target.com`. Ya conocíamos algunos de los hosts descubiertos, pero habría sido poco probable adivinar nombres como `sendasmoochie.target.com`.

**Figura 3.2. Netcraft encuentra 36 servidores web de Target**

Google también se puede utilizar para este propósito con consultas como `site:target.com`.

## Consultas Whois contra registros de IP
Después de descubrir un conjunto de IP iniciales "semilla", estas deben investigarse para asegurar que pertenecen a la empresa que esperas y para determinar de qué bloques de red forman parte. Una empresa pequeña podría tener una asignación minúscula de 1 a 16 direcciones IP, mientras que las corporaciones más grandes a menudo tienen miles. Esta información se mantiene en bases de datos regionales, como ARIN (American Registry for Internet Numbers) para América del Norte y RIPE para Europa y Oriente Medio. Las herramientas whois modernas toman una dirección IP y consultan automáticamente el registro correspondiente.

Las empresas de tamaño pequeño y mediano normalmente no tienen espacio de IP asignado por empresas como ARIN. En su lugar, se les delegan bloques de red de sus ISP. A veces obtienes esta información del ISP a través de consultas de IP. Esto generalmente te deja con un gran bloque de red y no sabes qué porción del mismo está asignada a tu objetivo. Afortunadamente, muchos ISP ahora subdelegan rangos de clientes utilizando Shared Whois (SWIP) o Referral Whois (RWhois). Si el ISP ha hecho esto, conoces el tamaño exacto del bloque de red del cliente.

Una de las direcciones IP descubiertas anteriormente para `target.com` fue `161.225.130.163`. El Ejemplo 3.5 demuestra una consulta whois (dirigida automáticamente contra ARIN) para determinar el propietario y la información de asignación de IP para esta IP.

**Ejemplo 3.5. Uso de whois para encontrar el bloque de red que contiene 161.225.130.163**
```bash
> whois 161.225.130.163
[Querying whois.arin.net]
[whois.arin.net]

OrgName:    Target Corporation 
OrgID:      TARGET-14
Address:    1000 Nicollet TPS 3165
City:       Minneapolis
StateProv:  MN
PostalCode: 55403
Country:    US

NetRange:   161.225.0.0 - 161.225.255.255 
CIDR:       161.225.0.0/16 
NetName:    TARGETNET
NetHandle:  NET-161-225-0-0-1
Parent:     NET-161-0-0-0-0
NetType:    Direct Assignment
NameServer: NS3.TARGET.COM
NameServer: NS4.TARGET.COM
Comment:    
RegDate:    1993-03-04
Updated:    2005-11-02

OrgTechHandle: DOMAI45-ARIN
OrgTechName:   Domainnames admin 
OrgTechPhone:  +1-612-696-2525
OrgTechEmail:  Domainnames.admin@target.com
```

No es de extrañar que Target posea un enorme bloque de red de Clase B, que cubre las 65,536 direcciones IP desde `161.225.0.0` hasta `161.225.255.255`. Dado que el OrgName es Target, este no es un caso en el que estemos viendo resultados de su ISP.

El siguiente paso es buscar de manera similar todas las IP descubiertas anteriormente que no caigan dentro de este rango. Luego puedes comenzar con consultas más avanzadas. El comando `whois -h whois.arin.net \?` proporciona la sintaxis de consulta de ARIN. Sería estupendo si pudieras buscar todos los bloques de red que coincidan con una dirección, OrgID u OrgTechEmail determinada, pero los registros de IP generalmente no permiten eso. Sin embargo, se permiten muchas otras consultas útiles. Por ejemplo, `whois -h whois.arin.net @target.com` muestra todos los contactos de ARIN con direcciones de correo electrónico en `target.com`. La consulta `whois -h whois.arin.net "n target*"` muestra todos los identificadores (handles) de bloques de red que comienzan con target. No distingue entre mayúsculas y minúsculas. De manera similar, `whois -h whois.arin.net "o target*"` muestra todos los nombres organizacionales que comienzan con target. Puedes buscar la dirección, el número de teléfono y el correo electrónico de contacto asociados con cada entrada para determinar si forman parte de la empresa que deseas escanear. A menudo son terceros que resultan tener un nombre similar.

## Información de enrutamiento de Internet
El protocolo de enrutamiento principal de Internet es el Border Gateway Protocol (BGP). Al escanear organizaciones medianas y grandes, las tablas de enrutamiento BGP pueden ayudarte a encontrar sus subredes IP en todo el mundo. Por ejemplo, supongamos que deseas escanear direcciones IP pertenecientes a Microsoft Corporation. Una búsqueda DNS para `microsoft.com` proporciona la dirección IP `207.46.196.115`. Una consulta whois, como se discutió en la sección anterior, muestra que todo el bloque `207.46.0.0/16` pertenece a Microsoft en su dirección correspondiente "One Microsoft Way" en Redmond. Eso proporciona 65,536 direcciones IP para escanear, pero las tablas BGP exponen muchas más.

A entidades como Microsoft se les asignan números de sistema autónomo (AS) con fines de enrutamiento. Una herramienta práctica para determinar el número de AS anunciado para una dirección IP determinada está disponible en [http://asn.cymru.com/](http://asn.cymru.com/). Escribir `207.46.0.0` en este formulario proporciona el número de AS 8075 de Microsoft. A continuación, quiero encontrar todos los prefijos IP que se enrutan a este AS. Una herramienta práctica para hacerlo está disponible en [http://www.robtex.com/as/](http://www.robtex.com/as/). Escribir `AS8075` y presionar Go en esa página conduce a una pantalla de resumen que muestra 42 prefijos encontrados. Esos prefijos representan 339,456 direcciones IP y pueden enumerarse haciendo clic en la pestaña BGP.

Si bien obtener información de BGP de formularios web ya preparados como estos es conveniente, obtener datos de enrutamiento de enrutadores reales es más divertido y puede permitir consultas personalizadas más potentes. Varias organizaciones ofrecen este servicio. Para un ejemplo, conéctate por telnet a `route-views.routeviews.org` o visita [http://routeviews.org](http://routeviews.org). Por supuesto, estos servicios proporcionan acceso de solo lectura a los datos. Si necesitas manipular tablas de enrutamiento globales como parte de un plan diabólico para apoderarte de Internet, eso está fuera del alcance de este libro.
