# 🌐 Instalación y Configuración del Servidor ISC-DHCP en Debian

[cite_start]Este documento detalla la **instalación y configuración del servicio ISC-DHCP-SERVER** en un entorno de red virtualizado (VirtualBox), utilizando **Debian** como servidor y máquinas clientes con **Ubuntu Server** y **Windows 10**[cite: 19, 23, 184, 209].

[cite_start]La práctica incluye la gestión del servicio con `systemctl` (arranque, parada y monitorización) [cite: 19][cite_start], la configuración de clientes para la recepción de direcciones dinámicas [cite: 20][cite_start], y el análisis del proceso de concesión y renovación de IP mediante captura de tráfico con `tcpdump`[cite: 21, 231].

***

## 📋 Índice y Pasos Principales

1.  [cite_start]**Instalación previa del servicio** [cite: 24][cite_start]: Preparación de la máquina virtual con Debian (versiones 11, 12 o 13) [cite: 25] y actualización del sistema con `sudo apt update` y `sudo apt upgrade`[cite: 28, 29].
2.  [cite_start]**Instalación del servicio ISC\_DHCP-SERVER** [cite: 30][cite_start]: Ejecución del comando `sudo apt install isc-dhcp-server -y`[cite: 32, 40].
3.  [cite_start]**Configuración de red del servidor**[cite: 52]:
    * [cite_start]Verificación de tarjetas de red con `ip a`[cite: 53].
    * Edición del fichero de configuración de red (`/etc/network/interfaces`) [cite: 74, 76] para asignar una IP estática (`10.15.0.1/24`) a una interfaz (e.g., `enp0s8`) [cite: 90, 91] y DHCP a otra (e.g., `enp0s3`)[cite: 86].
    * [cite_start]Aplicación de cambios con `sudo systemctl restart networking`[cite: 94].
4.  [cite_start]**Configuración del servicio de DHCP**[cite: 121]:
    * Edición del fichero de configuración principal: `/etc/dhcp/dhcpd.conf`[cite: 122, 125].
    * [cite_start]Definición de parámetros globales y la subred dinámica[cite: 123].
    * [cite_start]Configuración de la interfaz de escucha en `/etc/default/isc-dhcp-server` [cite: 150] (establecida a `enp0s8`) [cite_start][cite: 164].
5.  [cite_start]**Verificación del servicio**: Reinicio y comprobación del estado con `sudo systemctl restart isc-dhcp-server` [cite: 168] [cite_start]y `sudo systemctl status isc-dhcp-server`[cite: 169].
6.  [cite_start]**Configuración y verificación en clientes**: Configuración de clientes (Ubuntu Server [cite: 184] [cite_start]y Windows 10 [cite: 209][cite_start]) para recibir direcciones dinámicas (utilizando `iface unp0s8 inet dhcp` en Linux [cite: 191] [cite_start]e `ipconfig` en Windows [cite: 215]).
7.  [cite_start]**Análisis y captura de tráfico** [cite: 231][cite_start]: Instalación de `tcpdump` [cite: 234] [cite_start]y captura del tráfico DHCP con `sudo tcpdump -i enp0s8 port 67 or port 68 -n -vv`[cite: 236].

***

## ⚙️ Configuración del Servidor DHCP

### Fichero `/etc/dhcp/dhcpd.conf` (Parámetros Principales)

| Parámetro | Valor (Ejemplo) |
| :--- | :--- |
| **Dominio Global** | `option domain-name "hamza.org";` [cite: 136] |
| **DNS Globales** | `option domain-name-servers 8.8.8.8, 8.8.4.4;` [cite: 137] |
| **Tiempo de Concesión (Defecto/Máx)** | `default-lease-time 600; max-lease-time 7200;` [cite: 138, 139] |
| **Subred** | `subnet 10.15.0.0 netmask 255.255.255.0 { ... }` [cite: 141] |
| **Rango de IPs** | `range 10.15.0.2 10.15.0.10;` [cite: 142] |
| **Router** | `option routers 10.15.0.1;` [cite: 143] |
| **Broadcast** | `option broadcast-address 10.15.0.255;` [cite: 144] |
| **Dominio Subred** | `option domain-name "hamzanetwork";` [cite: 145] |

### Interfaz de Escucha (`/etc/default/isc-dhcp-server`)

```bash
# Interfaz de escucha para IPv4.
INTERFACESV4="enp0s8" [cite: 164]
## 🔄 Proceso de Concesión DHCP (DORA)

[cite_start]El proceso de asignación de una IP (DORA) se compone de cuatro paquetes, capturados y analizados con `tcpdump`[cite: 232, 237]:

| Paquete | Acrónimo | Función |
| :--- | :--- | :--- |
| **1. DHCP Discover** | **Discover** | [cite_start]El cliente envía un mensaje de difusión (*broadcast*) a la red para localizar servidores DHCP disponibles[cite: 239, 244]. |
| **2. DHCP Offer** | **Offer** | [cite_start]El servidor responde ofreciendo una dirección IP y parámetros de configuración al cliente[cite: 240, 264]. |
| **3. DHCP Request** | **Request** | [cite_start]El cliente solicita formalmente la dirección IP ofrecida, confirmando su elección[cite: 241, 279]. |
| **4. DHCP Acknowledge** | **ACK** | [cite_start]El servidor confirma la solicitud y asigna definitivamente la dirección IP al cliente[cite: 242, 298]. |
