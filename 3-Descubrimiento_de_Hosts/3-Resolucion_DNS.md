# Resolución DNS

El enfoque clave del descubrimiento de hosts de Nmap es determinar qué hosts están activos y responden en la red. Eso reduce el campo de objetivos, ya que no se puede hackear un host que no existe. Pero no dejes que el descubrimiento termine ahí. No saldrías con alguien solo porque respira, y seleccionar equipos en la red para penetrar también merece un cuidado especial. Una gran fuente de información (sobre hosts en red, no sobre posibles citas) es el DNS, el sistema de nombres de dominio. Incluso las organizaciones conscientes de la seguridad a menudo asignan nombres que revelan la función de sus sistemas. No es raro ver puntos de acceso inalámbricos llamados `wap` o `wireless`, firewalls llamados `fw`, `firewall` o `fw-1`, y servidores web de desarrollo con contenido aún no publicado llamados `dev`, `staging`, `www-int` o `beta`. También se suelen revelar nombres de ubicaciones o departamentos, como en la empresa cuyo firewall de la oficina de Chicago se llama `fw.chi`.

Por defecto, Nmap realiza una resolución DNS inversa para cada IP que responde a las sondas de descubrimiento de hosts (es decir, las que están en línea). Si se omite el descubrimiento de hosts con `-Pn`, la resolución se realiza para todas las IP. En lugar de utilizar las lentas librerías de resolución DNS estándar, Nmap utiliza un resolutor stub personalizado que realiza docenas de solicitudes en paralelo.

Si bien los valores predeterminados generalmente funcionan bien, Nmap ofrece cuatro opciones para controlar la resolución DNS. Pueden afectar sustancialmente la velocidad del escaneo y la cantidad de información recopilada.

### Opciones de Resolución DNS

- **`-n` (Sin resolución DNS)**
  Indica a Nmap que nunca realice la resolución DNS inversa en las direcciones IP activas que encuentre. Dado que el DNS puede ser lento incluso con el resolutor stub paralelo integrado de Nmap, esta opción reduce los tiempos de escaneo.

- **`-R` (Resolución DNS para todos los objetivos)**
  Indica a Nmap que siempre realice la resolución DNS inversa en las direcciones IP de destino. Normalmente, el DNS inverso solo se realiza contra hosts que responden (en línea).

- **`--system-dns` (Usar el resolutor DNS del sistema)**
  Por defecto, Nmap resuelve las direcciones IP enviando consultas directamente a los servidores de nombres configurados en su host y luego escuchando las respuestas. Se realizan muchas solicitudes (a menudo docenas) en paralelo para mejorar el rendimiento. Especifique esta opción para usar el resolutor de su sistema en su lugar (una IP a la vez a través de la llamada `getnameinfo`). Esto es lento y rara vez útil a menos que encuentre un error en el resolutor paralelo de Nmap (por favor, háganoslo saber si lo hace). El resolutor del sistema se utiliza siempre para escaneos IPv6.

- **`--dns-servers <servidor1>[,<servidor2>[,...]]` (Servidores a utilizar para consultas DNS inversas)**
  Por defecto, Nmap determina sus servidores DNS (para la resolución rDNS) a partir de su archivo `resolv.conf` (Unix) o del Registro (Win32). Alternativamente, puede usar esta opción para especificar servidores alternativos. Esta opción no se respeta si está usando `--system-dns` o un escaneo IPv6. El uso de varios servidores DNS suele ser más rápido, especialmente si elige servidores autoritativos para su espacio de IP de destino. Esta opción también puede mejorar el sigilo, ya que sus solicitudes pueden rebotar en casi cualquier servidor DNS recursivo en Internet.

Esta opción también resulta útil cuando se escanean redes privadas. A veces, solo unos pocos servidores de nombres proporcionan información rDNS adecuada, y es posible que ni siquiera sepa dónde están. Puede escanear la red en busca del puerto 53 (quizás con detección de versiones), luego intentar escaneos de lista de Nmap (`-sL`) especificando cada servidor de nombres uno a por uno con `--dns-servers` hasta que encuentre uno que funcione.
