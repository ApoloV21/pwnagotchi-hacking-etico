# Pwnagotchi — Sistema Autónomo de Auditoría WiFi con Deep Reinforcement Learning

> **Proyecto académico** — Hacking Ético  
> Escuela Superior Politécnica de Chimborazo, Sede Morona Santiago  
> Facultad de Informática y Electrónica  
> Docente: Ing. Katherine Adriana Merino Villa  
> Período: Marzo–Julio 2026

---

## Descripción

Implementación de un dispositivo autónomo de auditoría de redes WiFi basado en el algoritmo de aprendizaje por refuerzo profundo A2C (*Advantage Actor-Critic*), ejecutado sobre una Raspberry Pi Zero 2 W. El sistema utiliza Bettercap como motor de captura y Pwnagotchi v2.9.5.4 como capa de inteligencia artificial, con el objetivo de capturar handshakes WPA/WPA2 de forma autónoma en entornos controlados y autorizados.

---

## Hardware utilizado

| Componente | Especificación |
|---|---|
| Raspberry Pi Zero 2 W | ARM Cortex-A53 64-bit, 512MB RAM |
| Tarjeta MicroSD | 16GB Clase 10 |
| Cable micro-USB con datos | Micro-USB a USB-A |
| Power bank | 5V, 1A mínimo |

**Nota:** este proyecto no utiliza pantalla e-ink. La interacción se realiza exclusivamente mediante SSH sobre USB (modo gadget) y el dashboard web en `http://10.0.0.2:8080`.

---

## Software

| Componente | Versión |
|---|---|
| Pwnagotchi (imagen jayofelony) | v2.9.5.4 |
| Bettercap | v2.41.5 |
| Firmware WiFi | Nexmon (BCM43430) |
| Python | 3.13 |
| Sistema operativo | Debian GNU/Linux (trixie, arm64) |
| Kernel | 6.12.62+rpt-rpi-v8 |

---

## Configuración

### Archivo `config.toml`

Ubicación en el dispositivo: `/etc/pwnagotchi/config.toml`

```toml
[main]
name = "pwnagotchi"
lang = "en"
whitelist = ["FAMILIA VEGA", "FAMILIA VEGA_5G", "Repetidor_Chiriap"]
handshakes = "/home/pi/handshakes"

[main.personality]
advertise = true
deauth = true
associate = true
channels = []

[ui.display]
enabled = false
```

**Parámetros clave:**
- `whitelist` — redes excluidas de la captura (redes propias del laboratorio)
- `handshakes` — directorio de almacenamiento de archivos PCAP
- `ui.display.enabled = false` — pantalla deshabilitada, interacción por SSH

---

## Instalación y configuración

### 1. Grabar la imagen

Descargar la imagen de jayofelony desde:  
https://github.com/jayofelony/pwnagotchi

Seleccionar la versión **64-bit** para Raspberry Pi Zero 2 W. Verificar integridad:

```bash
shasum -a 256 pwnagotchi-*.img
```

Grabar con Raspberry Pi Imager seleccionando "Use custom".

### 2. Crear `config.toml` en la partición bootfs

Antes del primer arranque, crear el archivo `config.toml` en la raíz de la partición `bootfs` de la MicroSD (al mismo nivel que `config.txt`). Pwnagotchi lo detecta y lo copia automáticamente a `/etc/pwnagotchi/config.toml`.

### 3. Solución al problema del driver WiFi (Nexmon)

El módulo original del kernel tiene prioridad sobre el de Nexmon, lo que impide el modo monitor. Solución aplicada:

```bash
sudo nano /etc/modprobe.d/brcmfmac-nexmon.conf
```

Agregar:
```
blacklist brcmfmac
```

Luego:
```bash
sudo depmod -a
sudo update-initramfs -u
sudo reboot
```

Verificar que las interfaces aparecen correctamente:
```bash
iwconfig
# Debe mostrar wlan0 y wlan0mon
```

### 4. Directorio de handshakes

```bash
mkdir -p /home/pi/handshakes
```

Agregar en `config.toml`:
```toml
main.handshakes = "/home/pi/handshakes"
```

---

## Conexión al laptop

### Modo Manual (USB conectado)

```
IP del laptop (adaptador RNDIS): 10.0.0.1
IP de la RPi:                    10.0.0.2
```

Configurar el adaptador **Raspberry Pi USB Remote NDIS Network Device** en Windows:
- IP: `10.0.0.1`
- Máscara: `255.255.255.0`

Conectar por SSH:
```bash
ssh pi@10.0.0.2
```

### Compartición de internet

Para dar acceso a internet a la RPi a través del cable USB:

Panel de control → Conexiones de red → clic derecho en **Wi-Fi** → Propiedades → Compartir → activar y seleccionar el adaptador RNDIS.

### Modo Auto (captura activa)

Al desconectar el USB del laptop y alimentar con power bank, el dispositivo entra automáticamente en modo AUTO y comienza la captura.

### Dashboard web

Disponible en modo Manual en:
```
http://10.0.0.2:8080
```
Credenciales por defecto: `changeme / changeme`

---

## Transferencia de handshakes

Desde PowerShell en el laptop:

```powershell
scp pi@10.0.0.2:/home/pi/handshakes/*.pcap C:\Users\<usuario>\Desktop\
```

---

## Resultados de las pruebas

### Sesión de prueba — 26 de junio de 2026

| Parámetro | Valor |
|---|---|
| Duración de sesión | ~10 minutos |
| Épocas completadas | 3 |
| Redes detectadas | BARBERSHOP, BAR CHURUWIA, General-Zod |
| Handshakes capturados | 3 archivos PCAP |
| Reward promedio | 0.100 → 0.133 → 0.150 (creciente) |
| Estado emocional del agente | sad=0, bored=0 |
| Temperatura del dispositivo | 46–47°C |

**Archivos capturados:**
- `BARBERSHOP_f0090d5c612f.pcap`
- `BARCHURUWIA_40ed00e29ffd.pcap`
- `GeneralZod_320a190ddb37.pcap`

### Análisis en Wireshark

Filtro aplicado: `eapol`

Se verificaron 4 paquetes EAPOL por archivo, correspondientes a mensajes del 4-way handshake WPA2. El análisis confirma que el dispositivo captura correctamente el intercambio de autenticación.

---

## Marco legal

Este proyecto se ejecuta bajo los siguientes principios legales y éticos:

- Las pruebas se realizaron exclusivamente sobre redes propias o con autorización expresa del propietario
- El Código Orgánico Integral Penal (COIP) del Ecuador establece penas de 3 a 5 años por interceptación ilegal de comunicaciones
- La Ley Orgánica para el Fortalecimiento de la Ciberseguridad (mayo 2025) reconoce el hacking ético como actividad legítima cuando se cuenta con autorización del titular del sistema
- Las redes propias del entorno de pruebas están incluidas en el whitelist del dispositivo

---

## Problemas conocidos y soluciones

| Problema | Causa | Solución |
|---|---|---|
| SSH timeout en `10.0.0.2` | IP del adaptador RNDIS mal configurada | Asignar manualmente `10.0.0.1` al adaptador RNDIS en Windows |
| `wlan0` no aparece en `iwconfig` | Módulo original del kernel con prioridad sobre Nexmon | Blacklistear `brcmfmac` y recompilar initramfs |
| Firmware WiFi crasheando | `brcmf_fw_crashed` en dmesg | Resuelto al activar Nexmon correctamente |
| Pwnagotchi en modo manual al conectar USB | Comportamiento por diseño | Desconectar USB y alimentar por power bank para modo AUTO |
| Directorio `/root/handshakes` inaccesible | Permisos de root | Usar `/home/pi/handshakes` y configurar en `config.toml` |
| Internet perdido al desconectar USB | Compartición de conexión no persistente en Windows | Reactivar manualmente la compartición cada sesión |

---

## Estructura del repositorio

```
pwnagotchi-hacking-etico/
├── README.md
├── config.toml
└── evidencias/
    └── captura_log.jpg
```

---

## Referencias

- Jayofelony Pwnagotchi: https://github.com/jayofelony/pwnagotchi
- Documentación oficial: https://pwnagotchi.ai
- Nexmon firmware: https://github.com/seemoo-lab/nexmon
- Bettercap: https://www.bettercap.org
- COIP Ecuador — Delitos informáticos: Art. 229–234
- Ley Orgánica para el Fortalecimiento de la Ciberseguridad, Ecuador (2025)
