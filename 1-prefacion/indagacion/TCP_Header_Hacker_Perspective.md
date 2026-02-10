# 💀 Análisis del Header TCP: Perspectiva del Atacante

Mientras que el Header IPv4 es el sistema de entrega ("el sobre"), el **Header TCP (Transmission Control Protocol)** es el mecanismo de control de sesión ("la carta certificada con acuse de recibo"). Desde una perspectiva de ciberseguridad y Programación Orientada a Objetos (POO), este componente define la lógica de estado de la conexión.

A diferencia del ruteo simple, aquí es donde un atacante busca manipular la "máquina de estados" de la comunicación para secuestrar sesiones, denegar servicios o evadir detecciones.

---

## 🛠️ La Clase `TCPSegment`: Atributos de Control

En términos de POO, esta cabecera define las propiedades y comportamientos del objeto de flujo de datos. Un atacante no respeta los métodos públicos de esta clase; manipula directamente sus atributos privados.

### 1. Source & Destination Port (Puertos de Origen y Destino)
*   **Concepto:** Equivalentes a los "IDs de proceso" o endpoints de servicio.
*   **Perspectiva del Hacker:**
    *   **Destination Port:** Identifica el servicio víctima (ej. 443 para HTTPS, 22 para SSH).
    *   **Source Port:** Típicamente aleatorio en clientes legítimos. Un atacante puede fijarlo (ej. a 53, DNS) para intentar evadir reglas de firewall mal configuradas que confían en el tráfico proveniente de ciertos puertos.

### 2. Sequence Number (Número de Secuencia - SEQ)
*   **Concepto:** El "índice" del objeto en el stream de datos. Garantiza el orden.
*   **Vector de Ataque:** **TCP Sequence Prediction Attack**.
    *   Si un atacante puede predecir el siguiente número de secuencia de una sesión activa, puede inyectar paquetes maliciosos (resetear conexión o inyectar comandos) haciéndose pasar por una de las partes legítimas.

### 3. Acknowledgment Number (Número de Reconocimiento - ACK)
*   **Concepto:** El método `confirmarRecepcion()`. Indica al emisor qué bytes han llegado íntegros.
*   **Vector de Ataque:** Manipulación de flujo y escaneos ACK para determinar reglas de firewall (si el firewall bloquea o permite paquetes establecidos).

### 4. TCP Flags (Banderas de Control)
*   **Concepto:** Propiedades booleanas que gestionan el ciclo de vida de la conexión.
*   **Vector de Ataque:**
    *   **SYN (0x02):** Inundación (**SYN Flood**) para agotar la memoria del servidor creando miles de conexiones "a medio abrir".
    *   **RST (0x04):** Usado para "matar" conexiones activas (ej. TCP Reset Attack contra sesiones BGP o descargas).
    *   **FIN (0x01), URG (0x20), PSH (0x08):** Combinaciones ilegales o inusuales (Xmas Tree Scan, Null Scan) para fingerprinting de SO o evasión de IDS.

### 5. Window Size (Tamaño de Ventana)
*   **Concepto:** Control de flujo. Define cuántos bytes puede procesar el receptor antes de saturarse.
*   **Vector de Ataque:** **Sockstress / Zero Window Attack**.
    *   El atacante establece una conexión y luego anuncia una ventana de tamaño `0`. El servidor víctima mantiene la conexión viva y sondea periódicamente, consumiendo recursos sin poder enviar datos, lo que lleva a una Denegación de Servicio (DoS).

---

## 💻 Ejemplo: Instanciación de Ataque (Pseudocódigo)

Un hacker utiliza librerías de bajo nivel (como Scapy en Python) para crear instancias de paquetes que violan las reglas del protocolo estándar.

```python
# Manipulación directa de atributos del objeto TCP
paquete_malicioso = TCP()

# 1. Selección de Objetivo
paquete_malicioso.dport = 443           # Servicio HTTPS
paquete_malicioso.flags = "S"           # Flag SYN (Inicio de conexión)

# 2. Manipulación de Estado (Sequence Prediction)
# Intentar adivinar el número de secuencia para secuestrar sesión
paquete_malicioso.seq = 12345678       

# 3. Inyección en la Red
# Se envía encapsulado en un objeto IP falsificado
send(IP(src="192.168.1.100", dst="Target_IP")/paquete_malicioso)
```

---

## 🔍 Resumen Técnico

| Componente | Función Legítima | Objetivo del Atacante |
| :--- | :--- | :--- |
| **Puertos** | Multiplexación de servicios | Identificación de superficie de ataque. |
| **Flags (SYN/FIN)** | Gestión de conexión | Evasión de firewalls, DoS y Fingerprinting. |
| **SEQ/ACK** | Fiabilidad y orden | Secuestro de sesión (Hijacking) e Inyección. |
| **Window Size** | Control de flujo | Denegación de Servicio (DoS). |

> **Conclusión:** Mientras la cabecera IP permite que el ataque llegue a la puerta, la cabecera TCP es la herramienta para **forzar la cerradura**, manipular la conversación o sabotear el servicio.
