# 🏴‍☠️ Análisis del Header IPv4: Perspectiva del Atacante

Este documento desglosa la cabecera IPv4 enfocándose exclusivamente en los campos y datos que son relevantes para un atacante, pentester o investigador de seguridad. Aquí no nos importa la teoría de redes estándar; nos importa cómo usar estos campos para reconocimiento, evasión y ataque.

---

## 🎯 Campos Críticos para la Explotación

### 1. Source Address (Dirección de Origen) & Destination Address (Dirección de Destino)
*   **Posición:** Offsets 12 y 16.
*   **Interés del Hacker:**
    *   **Spoofing:** Falsificar la IP de origen para ocultar la identidad o realizar ataques de reflexión (DDoS).
    *   **Man-in-the-Middle (MitM):** En redes locales, ARP spoofing redirige el tráfico víctima -> atacante -> router.
    *   **Geo-localización:** Identificar la ubicación física del objetivo.

### 2. Time To Live (TTL)
*   **Función Normal:** Evitar bucles infinitos.
*   **Uso Ofensivo:**
    *   **OS Fingerprinting:** Diferentes sistemas operativos usan valores TTL iniciales distintos (ej. Windows suele ser 128, Linux 64). Esto permite identificar el SO del objetivo sin escanear puertos.
    *   **Traceroute/Firewalking:** Manipular el TTL para mapear la topología de la red interna y descubrir reglas de firewall.

### 3. Identification (ID)
*   **Función Normal:** Reensamblaje de fragmentos.
*   **Uso Ofensivo:**
    *   **Idle Scan (Zombie Scan):** Si el ID es secuencial, se puede usar un host "zombie" para escanear a una víctima sin revelar la IP del atacante.
    *   **OS Fingerprinting:** Analizar si el ID es secuencial, aleatorio o cero.

### 4. IP Flags (Banderas) & Fragment Offset
*   **Función Normal:** Control de fragmentación.
*   **Uso Ofensivo:**
    *   **Evasión de IDS/IPS:** Fragmentar paquetes maliciosos en trozos pequeños para que pasen desapercibidos por los sistemas de detección de intrusos y se reensamblen en el destino.
    *   **Teardrop Attack:** Enviar fragmentos superpuestos o mal formados para crashear sistemas vulnerables.
    *   **DF (Don't Fragment):** Usado para descubrir la MTU del camino (PMTUD).

### 5. Protocol (Protocolo)
*   **Función Normal:** Identifica el protocolo de la capa superior (TCP, UDP, ICMP).
*   **Uso Ofensivo:**
    *   **Identificación de Servicios:** Saber qué esperar (TCP=Web/Mail, UDP=DNS/VoIP, ICMP=Ping).
    *   **Protocol Tunneling:** Esconder datos en protocolos permitidos (ej. túneles DNS o ICMP) para exfiltrar información.

---

## 🕵️‍♂️ Análisis de Tráfico Real (Sniffing)

Cuando interceptamos tráfico (usando Wireshark, tcpdump, o Bettercap), esto es lo que buscamos en aplicaciones comunes:

### Gmail, Facebook, WhatsApp (Tráfico Encriptado)
*   **Lo que VES:**
    *   **IPs:** Sabes con quién se comunica la víctima (ej. servidores de Meta o Google).
    *   **Volumen y Timing:** Puedes inferir actividad (ej. "el usuario está enviando un archivo grande").
*   **Lo que NO VES (directamente):**
    *   **Payload:** El contenido (mensajes, contraseñas) está cifrado con TLS/SSL (Capa de Transporte/Aplicación).
*   **Vector de Ataque:**
    *   **SSL Stripping (obsoleto en HSTS):** Intentar degradar la conexión a HTTP.
    *   **Fake CA:** Si puedes instalar un certificado raíz en la máquina víctima, puedes desencriptar el tráfico (Intercepción HTTPS).

### Tráfico No Cifrado (HTTP, FTP, Telnet)
*   **Tesoro:** Credenciales, cookies de sesión y archivos visibles en texto plano directamente en el payload después de la cabecera IP y TCP.

---

## 🛠️ Herramientas de Manipulación

Para interactuar con estos campos a bajo nivel, un hacker utiliza:
*   **Scapy (Python):** Para forjar paquetes con valores arbitrarios en cualquier campo (ej. `IP(src="1.2.3.4", dst="target", ttl=1)/TCP()`).
*   **Nmap:** Para escaneos que manipulan flags y fragmentación (`-f`, `-D`, `--source-port`).
*   **Hping3:** Generador de paquetes para pruebas de firewall y DoS.

> **Nota Final:** El header IPv4 es el mapa de entrega. Controlarlo significa controlar cómo, dónde y si el ataque llega a su destino.
