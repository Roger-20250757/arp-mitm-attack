# Ataque Man in the Middle (MitM) mediante ARP Spoofing

**Autor:** Roger Rodriguez  
**Matrícula:** 20250757  
**Fecha:** Junio 2026  
**Link** https://youtu.be/Gi_knIUwEgI

---

## Objetivo del laboratorio

Demostrar cómo un atacante puede interceptar el tráfico entre dos dispositivos
de la red mediante ARP Spoofing (envenenamiento de caché ARP), posicionándose
como intermediario entre las víctimas sin que estas lo detecten.

---

## Objetivo del script

El script `arp_mitm.py` realiza las siguientes acciones:

1. Obtiene las MACs reales de PC1, PC2 y el Gateway
2. Envía respuestas ARP falsas a cada víctima asociando las IPs ajenas con la MAC del atacante
3. Habilita IP Forwarding para que el tráfico siga fluyendo sin interrupciones
4. Al detenerse restaura las tablas ARP originales

---

## Parámetros usados

| Parámetro | Valor | Descripción |
|---|---|---|
| `INTERFAZ` | `eth0` | Interfaz de red del atacante |
| `IP_VICTIMA1` | `20.25.7.10` | IP de PC1 |
| `IP_VICTIMA2` | `20.25.7.20` | IP de PC2 |
| `IP_GATEWAY` | `20.25.7.1` | IP del Router-GW |
| `DELAY` | `2` segundos | Intervalo entre envíos ARP |

---

## Requisitos para utilizar la herramienta

### Software
- Kali Linux
- Python 3.x
- Librería Scapy

### Instalación de dependencias
```bash
sudo apt update && sudo apt install python3-scapy -y
```

### Permisos
```bash
sudo python3 arp_mitm.py
```

---

## Documentación del funcionamiento del script

### ¿Cómo funciona ARP Spoofing?

ARP (Address Resolution Protocol) es un protocolo sin autenticación. Cualquier
dispositivo puede enviar una respuesta ARP falsa y los demás la aceptarán
actualizando su caché ARP sin verificar su legitimidad.

### Flujo del ataque

```
ANTES del ataque:
PC1 cree que 20.25.7.1 tiene MAC: aa:bb:cc:00:10:00 (Router real)

DURANTE el ataque:
PC1 cree que 20.25.7.1 tiene MAC: 00:50:00:00:05:00 (Kali)
Gateway cree que 20.25.7.10 tiene MAC: 00:50:00:00:05:00 (Kali)

RESULTADO:
PC1 --> Kali --> Gateway  (Kali intercepta todo el tráfico)
```

### Diagrama del ataque

```
   PC1                  ATTACKER (Kali)           Gateway
20.25.7.10              20.25.7.100              20.25.7.1
    |                        |                       |
    |<-- ARP falso: GW=Kali--|                       |
    |                        |<-- ARP falso: PC1=Kali|
    |                        |                       |
    |-------- trafico ------>|-------- trafico ----->|
    |                        |                       |
    |     Kali intercepta todo el trafico            |
```

### Pasos del script

1. **Descubrimiento:** Envía ARP requests para obtener MACs reales
2. **Envenenamiento:** Envía ARP replies falsos cada 2 segundos
3. **Forwarding:** Activa `ip_forward` para reenviar el tráfico
4. **Restauración:** Al hacer Ctrl+C restaura las tablas ARP originales

---

## Documentación de la red

### Topología

```
                    Router-GW
                   20.25.7.1/24
                        |
                    SW-CORE
                   /         \
            SW-ACCESS-1    SW-ACCESS-2
            /       \            \
        ATTACKER    PC1          PC2
      20.25.7.100  20.25.7.10  20.25.7.20
```

### Interfaces y direccionamiento

| Dispositivo | Interfaz | IP | VLAN |
|---|---|---|---|
| Router-GW | e0/0 | 20.25.7.1/24 | 1 |
| ATTACKER | eth0 | 20.25.7.100/24 | 1 |
| PC1 | eth0 | 20.25.7.10/24 | 1 |
| PC2 | eth0 | 20.25.7.20/24 | 1 |

---

## Ejecución del ataque

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/arp-mitm-attack

# Entrar al directorio
cd arp-mitm-attack

# Ejecutar el script
sudo python3 arp_mitm.py
```

### Resultado esperado
```
==================================================
 ARP MitM Attack
 Autor    : Roger Rodriguez
 Matricula: 20250757
 Victima 1: 20.25.7.10
 Victima 2: 20.25.7.20
 Gateway  : 20.25.7.1
==================================================
[*] Obteniendo MACs...
[+] MAC PC1    : 00:50:79:66:68:06
[+] MAC PC2    : 00:50:79:66:68:07
[+] MAC Gateway: aa:bb:cc:00:10:00
[*] IP Forwarding habilitado
[*] Iniciando envenenamiento ARP... Ctrl+C para detener
[+] Paquetes ARP enviados: 57
```

### Verificar impacto en PC1
```
VPCS> show arp
00:50:00:00:05:00  20.25.7.100 expires in 113 seconds
00:50:00:00:05:00  20.25.7.1   expires in 119 seconds
```
La MAC del Gateway ahora apunta al Kali — ataque exitoso.

---

## Contra-medida

### Descripción
Habilitar **Dynamic ARP Inspection (DAI)** en el switch para validar
todos los paquetes ARP contra la tabla de DHCP Snooping.

### Implementación en SW-CORE
```
enable
configure terminal
ip arp inspection vlan 1
interface e0/0
 ip arp inspection trust
end
write memory
```

### Verificación
```
SW-CORE# show ip arp inspection
```

### ¿Por qué funciona?
DAI intercepta todos los paquetes ARP en puertos no confiables y los valida
contra la tabla de DHCP Snooping. Si la MAC/IP no coincide, el paquete
es descartado automáticamente.

---

## Capturas de pantalla

| Archivo | Descripción |
|---|---|
| <img width="905" height="527" alt="image" src="https://github.com/user-attachments/assets/17d1eead-da9f-439e-a4fa-33d2c18f4b1e" /> | Topología en EVE-NG |
| <img width="838" height="155" alt="image" src="https://github.com/user-attachments/assets/feb02ff7-37d4-4e33-aa3a-35cc284c2ba9" /> | Tabla ARP de PC1 antes del ataque |
|<img width="935" height="518" alt="image" src="https://github.com/user-attachments/assets/ef7c06ff-715d-4b94-a1d2-c85388714f5b" />  | Script corriendo con paquetes enviados |
| <img width="665" height="116" alt="image" src="https://github.com/user-attachments/assets/cbb5ebd0-cd30-4eba-b5b0-3eb1a6b9a96b" /> | Tabla ARP de PC1 con MAC falsa |
| <img width="793" height="483" alt="image" src="https://github.com/user-attachments/assets/ac3ab270-ff0b-4062-aaa8-feca26eb14ff" /> | ARP Inspection activo en SW-CORE |

---

## Referencias

- [ARP Spoofing - OWASP](https://owasp.org/www-community/attacks/ARP_spoofing)
- [Cisco Dynamic ARP Inspection](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/12-2SX/configuration/guide/book/dynarp.html)
- [Scapy Documentation](https://scapy.readthedocs.io/)
