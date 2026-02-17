# Controles de Descubrimiento de Hosts

Por defecto, Nmap incluirá una etapa de escaneo de ping antes de realizar sondas más intrusivas, como escaneos de puertos, detección de SO, el Motor de Scripting de Nmap (NSE) o detección de versiones. Nmap normalmente solo realiza escaneos intrusivos en máquinas que demuestran estar disponibles durante la etapa de escaneo de ping. Esto ahorra una cantidad sustancial de tiempo y ancho de banda en comparación con la realización de escaneos completos contra cada una de las direcciones IP. Sin embargo, este enfoque no es ideal para todas las circunstancias. Hay veces en las que se desea escanear cada IP (`-Pn`), y otras en las que se desea realizar el descubrimiento de hosts sin un escaneo de puertos (`-sn`). Incluso hay ocasiones en las que se desea listar los hosts de destino y salir antes de enviar siquiera las sondas de ping (`-sL`). Nmap ofrece varias opciones de alto nivel para controlar este comportamiento.

### Escaneo de Lista (`-sL`)

El escaneo de lista es una forma degenerada de descubrimiento de hosts que simplemente enumera cada host en la(s) red(es) especificada(s), sin enviar ningún paquete a los hosts de destino. Por defecto, Nmap sigue realizando la resolución DNS inversa en los hosts para conocer sus nombres. Nmap también informa el número total de direcciones IP al final. El escaneo de lista es una buena comprobación de cordura para asegurar que se tienen las direcciones IP adecuadas para los objetivos. Si los hosts muestran nombres de dominio que no reconoce, vale la pena investigar más a fondo para evitar escanear la red de la empresa equivocada.

Existen muchas razones por las que los rangos de IP de destino pueden ser incorrectos. Incluso los administradores de red pueden escribir mal sus propios bloques de red, y los especialistas en pruebas de penetración (pen-testers) tienen aún más de qué preocuparse. En algunos casos, a los consultores de seguridad se les dan direcciones erróneas. En otros, intentan encontrar rangos de IP adecuados a través de recursos como bases de datos whois y tablas de enrutamiento. Las bases de datos pueden estar desactualizadas, o la empresa podría estar prestando espacio de IP a otras organizaciones. Si se deben escanear empresas matrices, filiales, proveedores de servicios y subsidiarias es un tema importante que debe acordarse con el cliente de antemano. Un escaneo de lista preliminar ayuda a confirmar exactamente qué objetivos se están escaneando.

Otra razón para realizar un escaneo de lista anticipado es el sigilo. En algunos casos, no se desea comenzar con un asalto a gran escala en la red de destino que probablemente active alertas de IDS y atraiga atención no deseada. Un escaneo de lista no es intrusivo y proporciona información que puede ser útil para elegir qué máquinas individuales atacar. Es posible, aunque muy poco probable, que el objetivo note todas las solicitudes de DNS inverso. Cuando eso sea un problema, se puede rebotar a través de servidores DNS recursivos anónimos utilizando la opción `--dns-servers`, como se describe en la sección llamada "Proxying de DNS".

Un escaneo de lista se especifica con la opción de línea de comandos `-sL`. Dado que la idea es simplemente imprimir una lista de hosts de destino, las opciones para funcionalidades de mayor nivel, como el escaneo de puertos, la detección de SO o el escaneo de ping, no pueden combinarse con `-sL`. Si desea desactivar el escaneo de ping mientras sigue realizando dichas funciones de mayor nivel, consulte la opción `-Pn`. El Ejemplo 3.6 muestra el uso del escaneo de lista para enumerar el rango de red CIDR /28 (16 direcciones IP) que rodea al servidor web principal de la Universidad de Stanford.

**Ejemplo 3.6. Enumeración de hosts que rodean www.stanford.edu con escaneo de lista**
```bash
felix~> nmap -sL www.stanford.edu/28

Starting Nmap ( https://nmap.org )
Host www9.Stanford.EDU (171.67.16.80) not scanned
Host www10.Stanford.EDU (171.67.16.81) not scanned
Host scriptorium.Stanford.EDU (171.67.16.82) not scanned
Host coursework-a.Stanford.EDU (171.67.16.83) not scanned
Host coursework-e.Stanford.EDU (171.67.16.84) not scanned
Host www3.Stanford.EDU (171.67.16.85) not scanned
Host leland-dev.Stanford.EDU (171.67.16.86) not scanned
Host coursework-preprod.Stanford.EDU (171.67.16.87) not scanned
Host stanfordwho-dev.Stanford.EDU (171.67.16.88) not scanned
Host workgroup-dev.Stanford.EDU (171.67.16.89) not scanned
Host courseworkbeta.Stanford.EDU (171.67.16.90) not scanned
Host www4.Stanford.EDU (171.67.16.91) not scanned
Host coursework-i.Stanford.EDU (171.67.16.92) not scanned
Host leland2.Stanford.EDU (171.67.16.93) not scanned
Host coursework-j.Stanford.EDU (171.67.16.94) not scanned
Host 171.67.16.95 not scanned
Nmap done: 16 IP addresses (0 hosts up) scanned in 0.38 seconds
```

### Desactivar Escaneo de Puertos (`-sn`)

Esta opción le indica a Nmap que no ejecute un escaneo de puertos después del descubrimiento de hosts. Cuando se usa sola, hace que Nmap realice el descubrimiento de hosts y luego imprima los hosts disponibles que respondieron al escaneo. Esto a menudo se denomina "escaneo de ping". Aunque no se realiza ningún escaneo de puertos, se pueden solicitar scripts de host del Motor de Scripting de Nmap (`--script`) y sondas de traceroute (`--traceroute`). Un escaneo de solo ping es un paso más intrusivo que un escaneo de lista, y a menudo se puede utilizar para los mismos propósitos. Realiza un reconocimiento ligero de una red de destino rápidamente y sin atraer mucha atención. Saber cuántos hosts están activos es más valioso para los atacantes que la lista de cada IP y nombre de host proporcionada por el escaneo de lista.

Los administradores de sistemas a menudo también encuentran valiosa esta opción. Puede usarse fácilmente para contar máquinas disponibles en una red o monitorear la disponibilidad de servidores. Esto a menudo se llama "ping sweep" (barrido de ping), y es más confiable que hacer ping a la dirección de difusión (broadcast) porque muchos hosts no responden a las consultas de difusión.

El Ejemplo 3.7 muestra un escaneo de ping rápido contra el CIDR /24 (256 IP) que rodea uno de mis sitios web favoritos, Linux Weekly News.

**Ejemplo 3.7. Descubrimiento de hosts que rodean www.lwn.net con un escaneo de ping**
```bash
# nmap -sn -T4 www.lwn.net/24

Starting Nmap ( https://nmap.org )
Host 66.216.68.0 seems to be a subnet broadcast address (returned 1 extra ping)
Host 66.216.68.1 appears to be up.
Host 66.216.68.2 appears to be up.
Host 66.216.68.3 appears to be up.
Host server1.camnetsec.com (66.216.68.10) appears to be up.
Host akqa.com (66.216.68.15) appears to be up.
Host asria.org (66.216.68.18) appears to be up.
Host webcubic.net (66.216.68.19) appears to be up.
Host dizzy.yellowdog.com (66.216.68.22) appears to be up.
Host www.outdoorwire.com (66.216.68.23) appears to be up.
Host www.inspectorhosting.com (66.216.68.24) appears to be up.
Host jwebmedia.com (66.216.68.25) appears to be up.
[...]
Host rs.lwn.net (66.216.68.48) appears to be up.
Host 66.216.68.52 appears to be up.
Host cuttlefish.laughingsquid.net (66.216.68.53) appears to be up.
[...]
Nmap done: 256 IP addresses (105 hosts up) scanned in 12.69 seconds
```

Este ejemplo solo tomó 13 segundos, pero proporciona información valiosa. En ese rango de direcciones de tamaño clase C, 105 hosts están en línea. A partir de los nombres de dominio no relacionados, todos empaquetados en un espacio de IP tan pequeño, queda claro que LWN utiliza un proveedor de coubicación o de servidores dedicados. Si las máquinas de LWN resultan ser altamente seguras, un atacante podría ir tras una de esas máquinas vecinas y luego realizar un ataque ethernet local con herramientas como Ettercap o Dsniff. Un uso ético de esta información sería el de un administrador de red que esté considerando mover máquinas a este proveedor. Podría enviar correos electrónicos a algunas de las organizaciones enumeradas y pedir su opinión sobre el servicio antes de firmar un contrato a largo plazo o realizar el costoso y disruptivo traslado del centro de datos.

La opción `-sn` envía una solicitud de eco ICMP, un paquete TCP SYN al puerto 443, un paquete TCP ACK al puerto 80 y una solicitud de marca de tiempo ICMP por defecto. Dado que los usuarios de Unix sin privilegios (o los usuarios de Windows sin Npcap instalado) no pueden enviar estos paquetes sin procesar, en esos casos solo se envían paquetes SYN. El paquete SYN se envía mediante una llamada al sistema `connect` de TCP a los puertos 80 y 443 del host de destino. Cuando un usuario privilegiado intenta escanear objetivos en una red ethernet local, se utilizan solicitudes ARP (`-PR`) a menos que se especifique la opción `--send-ip`.

La opción `-sn` puede combinarse con cualquiera de las técnicas discutidas en la sección llamada "Técnicas de Descubrimiento de Hosts" para una mayor flexibilidad. Si se utiliza cualquiera de esas opciones de tipo de sonda y número de puerto, las sondas predeterminadas se sobrescriben. Cuando hay firewalls estrictos entre el host de origen que ejecuta Nmap y la red de destino, se recomienda utilizar esas técnicas avanzadas. De lo contrario, se podrían omitir hosts cuando el firewall descarta las sondas o sus respuestas.

### Desactivar Ping (`-Pn`)

Otra opción es omitir la etapa de descubrimiento de Nmap por completo. Normalmente, Nmap utiliza esta etapa para determinar las máquinas activas para un escaneo más pesado. Por defecto, Nmap solo realiza sondas pesadas, como escaneos de puertos, detección de versiones o detección de SO, contra hosts que se encuentran activos. Desactivar el descubrimiento de hosts con la opción `-Pn` hace que Nmap intente las funciones de escaneo solicitadas contra cada dirección IP de destino especificada. Por lo tanto, si se especifica un espacio de direcciones de destino de tamaño clase B (`/16`) en la línea de comandos, se escanean las 65,536 direcciones IP. El descubrimiento de hosts adecuado se omite al igual que con un escaneo de lista, pero en lugar de detenerse e imprimir la lista de objetivos, Nmap continúa realizando las funciones solicitadas como si cada IP de destino estuviera activa.

Existen muchas razones para desactivar las pruebas de ping de Nmap. Una de las más comunes son las evaluaciones de vulnerabilidad intrusivas. Se pueden especificar docenas de sondas de ping diferentes en un intento de obtener una respuesta de todos los hosts disponibles, pero aún es posible que una máquina activa pero muy protegida por un firewall no responda a ninguna de esas sondas. Por lo tanto, para evitar perderse nada, los auditores realizan con frecuencia escaneos intensos, por ejemplo para los 65,536 puertos TCP, contra cada IP en la red de destino. Puede parecer un desperdicio enviar cientos de miles de paquetes a direcciones IP que probablemente no tengan ningún host escuchando, y puede ralentizar los tiempos de escaneo en un orden de magnitud o más. Nmap debe enviar retransmisiones a cada puerto en caso de que la sonda original se haya perdido en tránsito, y debe pasar un tiempo sustancial esperando respuestas porque no tiene una estimación del tiempo de ida y vuelta (RTT) para estas direcciones IP que no responden. Pero los especialistas en pruebas de penetración serios están dispuestos a pagar este precio para evitar incluso un ligero riesgo de perder máquinas activas. Siempre pueden realizar un escaneo rápido también, dejando el escaneo masivo con `-Pn` ejecutándose en segundo plano mientras trabajan. El Capítulo 6, Optimización del Rendimiento de Nmap, proporciona más consejos para el ajuste del rendimiento.

Otra razón frecuente que se da para usar `-Pn` es que el evaluador tiene una lista de máquinas que ya se sabe que están activas. Por lo tanto, el usuario no ve sentido en perder el tiempo con la etapa de descubrimiento de hosts. El usuario crea su propia lista de hosts activos y luego la pasa a Nmap usando la opción `-iL` (tomar entrada de lista). Esta estrategia rara vez es beneficiosa desde la perspectiva del ahorro de tiempo. Debido a los problemas de retransmisión y estimación de RTT discutidos en el párrafo anterior, incluso una dirección IP que no responde en una lista grande a menudo tomará más tiempo para escanear de lo que habría tomado toda una etapa de escaneo de ping. Además, la etapa de ping permite a Nmap recopilar muestras de RTT que pueden acelerar el siguiente escaneo de puertos, especialmente si el host de destino tiene reglas de firewall estrictas. Si bien especificar `-Pn` rara vez es útil para ahorrar tiempo, es importante si algunas de las máquinas de su lista bloquean todas las técnicas de descubrimiento que de otro modo se especificarían. Los usuarios deben encontrar un equilibrio entre la velocidad del escaneo y la posibilidad de pasar por alto máquinas fuertemente camufladas.
