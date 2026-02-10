# Práctica: Mapeo de Red LAN con Enfoque Orientado a Objetos (POO)

## Objetivo
El objetivo de esta práctica es realizar un descubrimiento de hosts en tu red de área local (LAN) utilizando `nmap`, pero estructurando la lógica, el procesamiento y los resultados siguiendo los principios de la Programación Orientada a Objetos (POO).

En lugar de simplemente ejecutar un comando y leer texto plano, modelarás los elementos de la red (Hosts, Puertos, Servicios) como **Objetos** dentro de un script (se recomienda Python).

## Escenario
Eres un administrador de sistemas que necesita inventariar su red LAN doméstica o de laboratorio. Necesitas identificar qué dispositivos están conectados, qué sistema operativo corren (especialmente Linux) y qué servicios exponen.

Tu red objetivo es (ejemplo): `192.168.1.0/24` (Ajusta esto a tu segmento de red real, ej. `192.168.0.0/24`).

---

## Requisitos del Ejercicio

### 1. Modelado de Clases (El "Modelo")
Debes diseñar las siguientes clases para representar la realidad de tu red:

#### Clase `Puerto`
Representa un puerto abierto en un host.
- **Atributos**:
  - `numero`: Entero (ej. 80).
  - `protocolo`: String (ej. "tcp").
  - `servicio`: String (ej. "http").
  - `estado`: String (ej. "open").
- **Métodos**:
  - `__str__()`: Devuelve una representación legible (ej. "80/tcp (http)").

#### Clase `Host`
Representa un dispositivo en la red.
- **Atributos**:
  - `ip`: String (Dirección IP).
  - `mac`: String (Dirección MAC, si está disponible).
  - `hostname`: String (Nombre DNS o NetBIOS).
  - `os_match`: String (Mejor coincidencia del Sistema Operativo).
  - `puertos`: Lista de objetos `Puerto`.
- **Métodos**:
  - `agregar_puerto(puerto)`: Añade un objeto `Puerto` a la lista.
  - `es_linux()`: Devuelve `True` si `os_match` contiene "Linux".
  - `tiene_servicio(nombre_servicio)`: Devuelve `True` si alguno de sus puertos corre el servicio indicado (ej. "ssh").
  - `resumen()`: Imprime IP, Hostname y cantidad de puertos abiertos.

#### Clase `EscaneoRed` (El "Controlador")
Encapsula la herramienta Nmap.
- **Atributos**:
  - `objetivo`: String (CIDR de la red, ej. "192.168.1.0/24").
- **Métodos**:
  - `ejecutar()`: Lanza el proceso de Nmap (usando `subprocess` o librerías como `python-nmap`). Se sugiere usar los flags `-sS` (Stealth), `-sV` (Versiones) y `-O` (Sistema Operativo).
  - `obtener_resultados()`: Parsea la salida (XML o texto) y devuelve una lista de objetos `Host` instanciados.

---

### 2. Lógica del Script
Tu script principal (`main`) debe seguir este flujo:
1. Instanciar `EscaneoRed` con tu rango de IP local.
2. Ejecutar el escaneo.
3. Iterar sobre la lista de objetos `Host` devueltos.
4. **Polimorfismo/Filtrado**:
   - Si el host es Linux (`host.es_linux()`), imprimir sus detalles con un formato especial (ej. color verde).
   - Si el host tiene el puerto 80/443 abierto, marcarlo como "Servidor Web".

---

## Ejemplo de Estructura (Python)

```python
import nmap  # Requiere 'pip install python-nmap' opciona, o usar subprocess

class Puerto:
    def __init__(self, numero, servicio):
        self.numero = numero
        self.servicio = servicio
    # ... implementación ...

class Host:
    def __init__(self, ip):
        self.ip = ip
        self.puertos = []
    
    def agregar_puerto(self, port_obj):
        self.puertos.append(port_obj)
    
    # ... más métodos ...

# Bloque Principal
if __name__ == "__main__":
    target = "192.168.1.0/24"  # ¡Configura esto con TU red!
    print(f"Iniciando escaneo de objetos en {target}...")
    
    # ... lógica de escaneo e instanciación ...
```

## Entregable Esperado
1. Un archivo `lan_oop_scan.py` con las clases implementadas.
2. Una ejecución en tu terminal mostrando los objetos detectados en tu LAN.

---
**Nota**: Recuerda que para la detección de SO (`-O`), `nmap` requiere privilegios de administrador (`sudo` o ejecutar como Administrador).
