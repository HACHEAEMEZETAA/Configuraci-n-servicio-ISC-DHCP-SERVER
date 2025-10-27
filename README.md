# 🌐 Instalación y Configuración del Servidor ISC-DHCP en Debian

Este documento detalla la **instalación y configuración del servicio ISC-DHCP-SERVER** en un entorno de red virtualizado (VirtualBox), utilizando **Debian** como servidor y máquinas clientes con **Ubuntu Server** y **Windows 10**[cite: 19, 23, 184, 209].

La práctica incluye la gestión del servicio con `systemctl` (arranque, parada y monitorización) [cite: 19][cite_start], la configuración de clientes para la recepción de direcciones dinámicas, y el análisis del proceso de concesión y renovación de IP mediante captura de tráfico con `tcpdump`.

***

## 📋 Índice y Pasos Principales

1.  **Instalación previa del servicio**: Preparación de la máquina virtual con Debian (versiones 11, 12 o 13)  y actualización del sistema con `sudo apt update` y `sudo apt upgrade`.
2.  **Instalación del servicio ISC\_DHCP-SERVER** : Ejecución del comando `sudo apt install isc-dhcp-server -y`.
3.  **Configuración de red del servidor**:
    * Verificación de tarjetas de red con `ip a`.
    * Edición del fichero de configuración de red (`/etc/network/interfaces`)  para asignar una IP estática (`10.15.0.1/24`) a una interfaz (e.g., `enp0s8`)  y DHCP a otra (e.g., `enp0s3`).
    * Aplicación de cambios con `sudo systemctl restart networking`.
4.  **Configuración del servicio de DHCP**:
    * Edición del fichero de configuración principal: `/etc/dhcp/dhcpd.conf`.
    * Definición de parámetros globales y la subred dinámica.
    * Configuración de la interfaz de escucha en `/etc/default/isc-dhcp-server` (establecida a `enp0s8`) .
5.  **Verificación del servicio**: Reinicio y comprobación del estado con `sudo systemctl restart isc-dhcp-server` `sudo systemctl status isc-dhcp-server`.
6.  **Configuración y verificación en clientes**: Configuración de clientes (Ubuntu Server y Windows 10) para recibir direcciones dinámicas (utilizando `iface unp0s8 inet dhcp` en Linux e `ipconfig` en Windows).
7.  **Análisis y captura de tráfico**: Instalación de `tcpdump` y captura del tráfico DHCP con `sudo tcpdump -i enp0s8 port 67 or port 68 -n -vv`.

***

## ⚙️ Configuración del Servidor DHCP

### Fichero `/etc/dhcp/dhcpd.conf` (Parámetros Principales)

| Parámetro | Valor (Ejemplo) |
| :--- | :--- |
| **Dominio Global** | `option domain-name "hamza.org";` |
| **DNS Globales** | `option domain-name-servers 8.8.8.8, 8.8.4.4;` |
| **Tiempo de Concesión (Defecto/Máx)** | `default-lease-time 600; max-lease-time 7200;` |
| **Subred** | `subnet 10.15.0.0 netmask 255.255.255.0 { ... }` |
| **Rango de IPs** | `range 10.15.0.2 10.15.0.10;`  |
| **Router** | `option routers 10.15.0.1;` |
| **Broadcast** | `option broadcast-address 10.15.0.255;` |
| **Dominio Subred** | `option domain-name "hamzanetwork";` |

# Interfaz de escucha para IPv4.
INTERFACESv4="enp0s8"

## Proceso de Concesión DHCP (DORA)

El proceso de asignación de una IP (DORA) se compone de cuatro paquetes, capturados y analizados con `tcpdump`:

| Paquete | Acrónimo | Función |
| :--- | :--- | :--- |
| **1. DHCP Discover** | **Discover** | El cliente envía un mensaje de difusión (**broadcast**) a la red para localizar servidores DHCP. |
| **2. DHCP Offer** | **Offer** | El servidor responde ofreciendo una dirección IP y parámetros de configuración al cliente. |
| **3. DHCP Request** | **Request** | El cliente solicita formalmente la dirección IP ofrecida, confirmando su elección. |
| **4. DHCP Acknowledge** | **ACK** | El servidor confirma la solicitud y asigna definitivamente la dirección IP al cliente. |
