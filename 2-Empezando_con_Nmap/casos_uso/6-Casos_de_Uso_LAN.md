# 5 Casos de Uso Comunes de Nmap en una Red LAN

A continuación, se presentan cinco escenarios prácticos para utilizar Nmap en una red local (LAN), junto con los comandos correspondientes y una explicación del contexto.

## 1. Descubrimiento de Dispositivos Activos (Host Discovery)
**Contexto:**
Acabas de conectarte a una red y necesitas saber qué dispositivos están encendidos y conectados (computadoras, impresoras, routers, dispositivos IoT). No te interesan los puertos abiertos todavía, solo quieres un inventario rápido de la red.

**Comando:**
```bash
nmap -sn 192.168.1.0/24
```
*(Reemplaza `192.168.1.0/24` con el rango de tu red)*

**Explicación:**
*   `-sn`: Realiza un "Ping Scan". Desactiva el escaneo de puertos y solo verifica si los hosts están activos mediante solicitudes ARP (en LAN) o ICMP/TCP (en redes externas).
*   Es rápido y sigiloso para hacer un mapa inicial de la red.

## 2. Inventario de Servicios y Versiones
**Contexto:**
Has identificado un servidor específico (por ejemplo, `192.168.1.50`) y quieres saber qué software está ejecutando (servidor web, base de datos, SSH) y sus versiones exactas para buscar vulnerabilidades específicas asociadas a esas versiones.

**Comando:**
```bash
nmap -sV 192.168.1.50
```

**Explicación:**
*   `-sV`: Activa la detección de versiones. Nmap interroga a los puertos abiertos para obtener información sobre el servicio y la versión del software (ej. "Apache httpd 2.4.41" en lugar de solo "http").
*   Esencial para determinar si un servicio está desactualizado.

## 3. Identificación del Sistema Operativo
**Contexto:**
Necesitas saber si un dispositivo desconocido en la red es una máquina Windows, un servidor Linux, una impresora o un dispositivo móvil. Esto te ayuda a entender mejor la superficie de ataque o a administrar la red.

**Comando:**
```bash
nmap -O 192.168.1.50
```

**Explicación:**
*   `-O`: Activa la detección de sistema operativo (OS Fingerprinting). Nmap analiza las respuestas TCP/IP del objetivo para comparar su "huella" con una base de datos conocida.
*   Útil para inventario de activos y para seleccionar exploits específicos de un SO.

## 4. Auditoría de Seguridad con Scripts (NSE)
**Contexto:**
Quieres verificar si los equipos Windows de tu red son vulnerables a amenazas conocidas como *EternalBlue* (MS17-010) u otras vulnerabilidades críticas del protocolo SMB, sin necesidad de herramientas externas complejas.

**Comando:**
```bash
nmap --script smb-vuln* -p 139,445 192.168.1.0/24
```

**Explicación:**
*   `--script smb-vuln*`: Ejecuta todos los scripts del Nmap Scripting Engine (NSE) que comienzan con "smb-vuln", diseñados para detectar fallos en SMB.
*   `-p 139,445`: Específica los puertos SMB para acelerar el escaneo.
*   Permite detectar rápidamente configuraciones inseguras o parches faltantes.

## 5. Escaneo Agresivo y Rápido (Auditoría Completa)
**Contexto:**
Necesitas realizar una auditoría completa y rápida de un host sospechoso o crítico. Quieres obtener la mayor cantidad de información posible (versiones, SO, traceroute y scripts por defecto) en un solo paso.

**Comando:**
```bash
nmap -A -T4 192.168.1.50
```

**Explicación:**
*   `-A`: Activa las opciones "Agresivas": Detección de OS (`-O`), detección de versiones (`-sV`), escaneo de scripts predeterminados (`-sC`) y traceroute (`--traceroute`).
*   `-T4`: Establece la plantilla de tiempo a "Agresiva" (más rápido). Recomendado para redes LAN rápidas y estables.
*   Proporciona una visión profunda del objetivo en una sola ejecución.
