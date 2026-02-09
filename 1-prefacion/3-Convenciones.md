# Convenciones

La salida de Nmap se utiliza a lo largo de este libro para demostrar principios y características. La salida a menudo se edita para eliminar líneas que son irrelevantes para el punto que se está tratando. Las fechas/horas y los números de versión impresos por Nmap generalmente también se eliminan, ya que algunos lectores los encuentran molestos. La información confidencial, como nombres de host, direcciones IP y direcciones MAC, puede modificarse o eliminarse. Otra información puede cortarse o las líneas ajustarse para que quepan en una página impresa. Se realiza una edición similar para la salida de otras aplicaciones. El Ejemplo 1 ofrece un vistazo a las capacidades de Nmap y al mismo tiempo demuestra el formato de salida.

**Ejemplo 1. Un escaneo típico de Nmap**

```bash
# nmap -A -T4 scanme.nmap.org

Starting Nmap ( https://nmap.org )
Nmap scan report for scanme.nmap.org (64.13.134.52)
Host is up (0.034s latency).
Not shown: 994 filtered ports
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 1024 60:ac:4d:51:b1:cd:85:09:12:16:92:76:1d:5d:27:6e (DSA)
|_2048 2c:22:75:60:4b:c3:3b:18:a2:97:2c:96:7e:28:dc:dd (RSA)
53/tcp    open   domain
70/tcp    closed gopher
80/tcp    open   http    Apache httpd 2.2.3 ((CentOS))
| http-methods: Potentially risky methods: TRACE
|_See https://nmap.org/nsedoc/scripts/http-methods.html
|_html-title: Go ahead and ScanMe!
113/tcp   closed auth
31337/tcp closed Elite
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.18 (CentOS 5.4)
Network Distance: 10 hops

TRACEROUTE (using port 113/tcp)
HOP RTT      ADDRESS
[Cortados los primeros ocho saltos por brevedad]
9   20.29 ms xe6-2.core1.svk.layer42.net (69.36.239.221)
10  19.58 ms scanme.nmap.org (64.13.134.52)

Nmap done: 1 IP address (1 host up) scanned in 25.97 seconds
```

Se proporciona un formato especial para ciertos tokens, como nombres de archivo y comandos de aplicaciones. La Tabla 1 demuestra las convenciones de formato más comunes.

**Tabla 1. Convenciones de estilo de formato**

| Tipo de token | Ejemplo |
| :--- | :--- |
| Cadena literal | Me emociono mucho más por los puertos en estado abierto (`open`) que por aquellos reportados como cerrados (`closed`) o filtrados (`filtered`). |
| Opciones de línea de comandos | Una de las opciones de Nmap más geniales, pero menos entendidas, es `--packet-trace`. |
| Nombres de archivo | Siga la opción `-iL` con el nombre del archivo de entrada como `C:\net\dhcp-leases.txt` o `/home/h4x/hosts-to-pwn.lst`. |
| Énfasis | *Usar Nmap desde la computadora de su trabajo o escuela para atacar bancos y objetivos militares es una mala idea.* |
| Comandos de aplicación | Trinity escaneó la Matrix con el comando `nmap -v -sS -O 10.2.2.2`. |
| Variables reemplazables | Sea `<fuente>` la máquina que ejecuta Nmap y `<objetivo>` sea `microsoft.com`. |
