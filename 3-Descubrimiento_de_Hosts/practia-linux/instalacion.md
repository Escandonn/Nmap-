Verificar Python en Linux

Instalar Nmap correctamente

Probar Nmap desde terminal

Instalar la librería python-nmap


🧱 1️⃣ Crear carpeta del proyecto
mkdir nmap_scanner
cd nmap_scanner


🧪 2️⃣ Crear y activar entorno virtual
python3 -m venv venv
source venv/bin/activate


📦 3️⃣ Instalar librería
pip install python-nmap



import nmap

print("======================================")
print("      HERRAMIENTA NMAP EN PYTHON")
print("======================================\n")

scanner = nmap.PortScanner()

# -----------------------------
# PASO 1: Tipo de objetivo
# -----------------------------
while True:
    print("¿Qué deseas escanear?")
    print("1 → Un host específico (ej: 192.168.1.10)")
    print("2 → Una red completa (ej: 192.168.1.0/24)")
    opcion = input("Selecciona 1 o 2: ")

    if opcion == "1":
        objetivo = input("Ingresa la IP del host: ")
        break
    elif opcion == "2":
        objetivo = input("Ingresa la red (formato 192.168.1.0/24): ")
        break
    else:
        print("Opción inválida. Intenta nuevamente.\n")

# -----------------------------
# PASO 2: Puertos
# -----------------------------
while True:
    puertos = input("\nIngresa el rango de puertos a escanear (ej: 1-1024): ")
    if "-" in puertos:
        break
    else:
        print("Formato incorrecto. Debe ser como 1-1024")

# -----------------------------
# PASO 3: Tipo de escaneo
# -----------------------------
while True:
    print("\n¿Qué tipo de escaneo deseas?")
    print("1 → TCP SYN (rápido y sigiloso)")
    print("2 → UDP (servicios UDP)")
    tipo_scan = input("Selecciona 1 o 2: ")

    if tipo_scan == "1":
        argumentos = "-sS -sV"
        descripcion = "TCP SYN Scan"
        break
    elif tipo_scan == "2":
        argumentos = "-sU -sV"
        descripcion = "UDP Scan"
        break
    else:
        print("Opción inválida.")

# -----------------------------
# RESUMEN
# -----------------------------
print("\n======================================")
print("RESUMEN DEL ESCANEO")
print(f"Objetivo: {objetivo}")
print(f"Puertos: {puertos}")
print(f"Tipo: {descripcion}")
print("======================================\n")

input("Presiona ENTER para iniciar el escaneo...")

# -----------------------------
# EJECUTAR ESCANEO
# -----------------------------
print("\nEscaneando, espera...\n")
scanner.scan(objetivo, puertos, arguments=argumentos)

# -----------------------------
# MOSTRAR RESULTADOS
# -----------------------------
for host in scanner.all_hosts():
    print(f"\n🖥️ Host: {host}")
    print(f"Estado: {scanner[host].state()}")

    for proto in scanner[host].all_protocols():
        print(f"\n  Protocolo: {proto}")

        for port in scanner[host][proto]:
            estado = scanner[host][proto][port]['state']
            servicio = scanner[host][proto][port].get('name', 'desconocido')

            print(f"   Puerto: {port}\tEstado: {estado}\tServicio: {servicio}")

print("\nEscaneo finalizado.")
