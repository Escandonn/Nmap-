# Especificación de Hosts y Redes Objetivo

Todo en la línea de comandos de Nmap que no es una opción (o argumento de opción) se trata como una especificación de host objetivo. El caso más simple es especificar una dirección IP o nombre de host objetivo para escanear.

A veces deseas escanear una red completa de hosts adyacentes. Para esto, Nmap admite el direccionamiento de estilo CIDR. Puedes agregar `/<numbits>` a una dirección IPv4 o nombre de host y Nmap escaneará cada dirección IP para la cual los primeros `<numbits>` sean los mismos que para la IP o nombre de host de referencia dado. Por ejemplo, `192.168.10.0/24` escanearía los 256 hosts entre 192.168.10.0 (binario: `11000000 10101000 00001010 00000000`) y 192.168.10.255 (binario: `11000000 10101000 00001010 11111111`), ambos inclusive. `192.168.10.40/24` escanearía exactamente los mismos objetivos. Dado que el host `scanme.nmap.org` está en la dirección IP `64.13.134.52`, la especificación `scanme.nmap.org/16` escanearía las 65.536 direcciones IP entre 64.13.0.0 y 64.13.255.255. El valor más pequeño permitido es `/0`, que apunta a todo Internet. El valor más grande es `/32`, que escanea solo el host o dirección IP nombrada porque todos los bits de dirección son fijos.

La notación CIDR es corta pero no siempre lo suficientemente flexible. Por ejemplo, es posible que desees escanear `192.168.0.0/16` pero omitir cualquier IP que termine en `.0` o `.255` porque pueden usarse como direcciones de red de subred y de transmisión (*broadcast*). Nmap admite esto mediante el direccionamiento de rango de octetos. En lugar de especificar una dirección IP normal, puedes especificar una lista separada por comas de números o rangos para cada octeto. Por ejemplo, `192.168.0-255.1-254` omitirá todas las direcciones en el rango que terminan en `.0` o `.255`, y `192.168.3-5,7.1` escaneará las cuatro direcciones `192.168.3.1`, `192.168.4.1`, `192.168.5.1` y `192.168.7.1`. Cualquiera de los lados de un rango puede omitirse; los valores predeterminados son 0 a la izquierda y 255 a la derecha. Usar `-` por sí solo es lo mismo que `0-255`, pero recuerda usar `0-` en el primer octeto para que la especificación del objetivo no parezca una opción de línea de comandos. Los rangos no necesitan limitarse a los octetos finales: el especificador `0-255.0-255.13.37` realizará un escaneo de Internet completo para todas las direcciones IP que terminen en `13.37`. Este tipo de muestreo amplio puede ser útil para encuestas e investigaciones de Internet.

Las direcciones IPv6 solo se pueden especificar mediante su dirección IPv6 completa o nombre de host. Los rangos CIDR y de octetos no son compatibles con IPv6 porque rara vez son útiles.

Nmap acepta múltiples especificaciones de host en la línea de comandos, y no necesitan ser del mismo tipo. El comando `nmap scanme.nmap.org 192.168.0.0/8 10.0.0,1,3-7.-` hace lo que esperarías.

### Entrada desde Lista (-iL)

Pasar una lista enorme de hosts a menudo es incómodo en la línea de comandos, sin embargo, es una necesidad común. Por ejemplo, tu servidor DHCP podría exportar una lista de 10.000 arrendamientos actuales que deseas escanear. O tal vez desees escanear todas las direcciones IP excepto esas para localizar hosts que usan direcciones IP estáticas no autorizadas. Simplemente genera la lista de hosts para escanear y pasa ese nombre de archivo a Nmap como argumento para la opción `-iL`. Las entradas pueden estar en cualquiera de los formatos aceptados por Nmap en la línea de comandos (dirección IP, nombre de host, CIDR, IPv6 o rangos de octetos). Cada entrada debe estar separada por uno o más espacios, tabulaciones o nuevas líneas. Puedes especificar un guion (`-`) como nombre de archivo si deseas que Nmap lea los hosts desde la entrada estándar en lugar de un archivo real.

### Elegir Objetivos al Azar (-iR <numtargets>)

Para encuestas de Internet completas y otras investigaciones, es posible que desees elegir objetivos al azar. Esto se hace con la opción `-iR`, que toma como argumento el número de IPs a generar. Nmap omite automáticamente ciertas IPs indeseables, como aquellas en rangos de direcciones privadas, multidifusión o no asignadas. Se puede especificar el argumento 0 para un escaneo interminable. Ten en cuenta que algunos administradores de red se molestan ante escaneos no autorizados de sus redes. Lee atentamente la sección llamada "Asuntos Legales" antes de usar `-iR`.

Si te encuentras realmente aburrido una tarde lluviosa, prueba el comando `nmap -sS -PS80 -iR 0 -p 80` para localizar servidores web aleatorios para navegar.

### Excluir Objetivos (--exclude, --excludefile <filename>)

Es común tener máquinas que no deseas escanear bajo ninguna circunstancia. Las máquinas pueden ser tan críticas que no correrás ningún riesgo de una reacción adversa. Podrían culparte por una interrupción coincidente incluso si el escaneo de Nmap no tuvo nada que ver con eso. O tal vez tienes hardware heredado que se sabe que se bloquea cuando se escanea, pero aún no has podido arreglarlo o reemplazarlo. O tal vez ciertos rangos de IP representan a compañías subsidiarias, clientes o socios que no estás autorizado a escanear. Los consultores a menudo no quieren que su propia máquina se incluya en un escaneo de las redes de sus clientes. Cualquiera sea la razón, puedes excluir hosts o redes completas con la opción `--exclude`. Simplemente pasa a la opción una lista separada por comas de objetivos y bloques de red excluidos utilizando la sintaxis normal de Nmap. Alternativamente, puedes crear un archivo de hosts/redes excluidos y pasarlo a Nmap con la opción `--excludefile`. La opción `--exclude` no se mezcla con rangos de IP que usan comas (`192.168.0.10,20,30`) porque `--exclude` en sí mismo usa comas. Usa `--excludefile` en estos casos.

## Ejemplos Prácticos

Si bien algunas herramientas tienen interfaces simples que solo permiten una lista de hosts o tal vez te permiten especificar las direcciones IP inicial y final para un rango, Nmap es mucho más potente y flexible. Pero Nmap también puede ser más difícil de aprender, y escanear las direcciones IP incorrectas es ocasionalmente desastroso. Afortunadamente, Nmap ofrece una ejecución de prueba utilizando el escaneo de lista (opción `-sL`). Simplemente ejecuta `nmap -sL -n <targets>` para ver qué IPs se escanearían antes de hacerlo realmente.

Los ejemplos pueden ser la forma más efectiva de enseñar la sintaxis de especificación de host de Nmap. Esta sección proporciona algunos, comenzando con los más simples.

`nmap scanme.nmap.org`, `nmap scanme.nmap.org/32`, `nmap 64.13.134.52`
Estos tres comandos hacen lo mismo, asumiendo que `scanme.nmap.org` se resuelve en `64.13.134.52`. Escanean esa única IP y luego salen.

`nmap scanme.nmap.org/24`, `nmap 64.13.134.52/24`, `nmap 64.13.134.-`, `nmap 64.13.134.0-255`
Estos cuatro comandos piden a Nmap que escanee las 256 direcciones IP desde 64.13.134.0 hasta 64.13.134.255. En otras palabras, piden escanear el espacio de direcciones de tamaño clase C que rodea a `scanme.nmap.org`.

`nmap 64.13.134.52/24 --exclude scanme.nmap.org,insecure.org`
Le dice a Nmap que escanee la clase C alrededor de `64.13.134.52`, pero que omita `scanme.nmap.org` e `insecure.org` si se encuentran dentro de ese rango de direcciones.

`nmap 10.0.0.0/8 --exclude 10.6.0.0/16,ultra-sensitive-host.company.com`
Le dice a Nmap que escanee todo el rango privado 10, excepto que debe omitir cualquier cosa que comience con 10.6, así como `ultra-sensitive-host.company.com`.

`egrep '^lease' /var/lib/dhcp/dhcpd.leases | awk '{print $2}' | nmap -iL -`
Obtiene la lista de direcciones IP DHCP asignadas y las alimenta directamente a Nmap para escanear. Ten en cuenta que se pasa un guion a `-iL` para leer desde la entrada estándar.

`nmap -6 2001:800:40:2a03::3`
Escanea el host IPv6 en la dirección `2001:800:40:2a03::3`.
