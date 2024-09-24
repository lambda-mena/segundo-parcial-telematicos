
# README: Configuración de Servidor FTP Seguro con TLS y Conexiones a través de Firewall

## Descripción del Proyecto

Este proyecto configura un **servidor FTP seguro** utilizando **vsftpd** en el **Servidor 2** con soporte para **TLS** y permite acceder al servidor a través de un **firewall** en el **Servidor 1**. Todas las conexiones FTP están cifradas, utilizando conexiones pasivas para transferencias de datos, y se canalizan a través del **Servidor 1**, que actúa como un intermediario para la red interna.

El objetivo es garantizar que:
1. Todas las solicitudes de acceso a los servicios FTP sean canalizadas a través del **Servidor 1** (firewall).
2. Las conexiones al servidor FTP sean **seguras** (usando TLS).

---

## Pasos realizados:

### 1. Configuración del Servidor FTP Seguro (Servidor 2)

Se instaló y configuró **vsftpd** en el **Servidor 2** para permitir conexiones seguras con **TLS** y habilitar el **modo pasivo**.

#### a) Instalación de vsftpd:
```bash
sudo apt-get update
sudo apt-get install vsftpd
```

#### b) Configuración de `/etc/vsftpd.conf`:

Se configuró el archivo de configuración de **vsftpd** para soportar conexiones seguras y usar el **modo pasivo**. Los cambios realizados fueron:

```bash
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES

# Habilitar SSL/TLS
ssl_enable=YES
force_local_logins_ssl=YES
force_local_data_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.key

# Configurar el rango de puertos pasivos y la IP del Servidor 1 (firewall)
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000
pasv_address=192.168.50.3  # IP del Servidor 1

log_ftp_protocol=YES
debug_ssl=YES
```

#### c) Reiniciar el servicio vsftpd:
Después de configurar el archivo, se reinició **vsftpd** para aplicar los cambios:

```bash
sudo systemctl restart vsftpd
```

---

### 2. Configuración del Firewall (Servidor 1)

El **Servidor 1** actúa como un firewall y redirige todas las conexiones entrantes a través del puerto **21** y los **puertos pasivos (40000-50000)** hacia el **Servidor 2**.

#### a) Habilitar el reenvío de IP:
Se habilitó el reenvío de IP en el **Servidor 1** para permitir el tráfico a través de este:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo nano /etc/sysctl.conf
# Asegurarse de que esté habilitada la línea:
net.ipv4.ip_forward=1
```

#### b) Configuración de las reglas de `iptables`:
Se configuraron las reglas de **iptables** para redirigir el tráfico FTP al **Servidor 2**.

```bash
# Redirigir el tráfico del puerto 21 al Servidor 2
sudo iptables -t nat -A PREROUTING -p tcp --dport 21 -j DNAT --to-destination 192.168.50.2:21

# Redirigir los puertos pasivos al Servidor 2
sudo iptables -t nat -A PREROUTING -p tcp --dport 40000:50000 -j DNAT --to-destination 192.168.50.2

# Enmascarar el tráfico para que el Servidor 2 vea las conexiones provenientes del Servidor 1
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.50.2 --dport 21 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.50.2 --dport 40000:50000 -j MASQUERADE
```

#### c) Configuración del firewall (UFW):
En el **Servidor 1**, se permitió el puerto **21** y el rango de puertos **40000-50000** para el FTP pasivo.

```bash
sudo ufw allow 21/tcp
sudo ufw allow 40000:50000/tcp
```
d)Paso para habilitar el reenvío en UFW:
Edita el archivo de configuración de UFW para habilitar el reenvío de tráfico:
```bash sudo nano /etc/default/ufw
```
Busca la línea:
DEFAULT_FORWARD_POLICY="DROP"

Cámbiala a:

DEFAULT_FORWARD_POLICY="ACCEPT"

f)Reinicia UFW para aplicar los cambios:
sudo ufw disable
sudo ufw enable
---


