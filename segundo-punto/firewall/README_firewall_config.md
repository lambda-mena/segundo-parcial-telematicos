
# Configuración del Firewall (VM1) para Redirección de DNS

## Pasos para Configurar el Firewall (VM1)

### 1. Instalar `ufw` y `iptables`
```bash
sudo apt update
sudo apt install ufw iptables-persistent -y
```

### 2. Configurar `ufw` para permitir tráfico DNS
```bash
sudo ufw allow in on eth0 to any port 53 proto udp
sudo ufw allow in on eth0 to any port 53 proto tcp
```

### 3. Configurar reglas de `iptables` para redireccionar el tráfico DNS al servidor esclavo (VM2)
```bash
sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j DNAT --to-destination 192.168.50.2:53
sudo iptables -t nat -A PREROUTING -p tcp --dport 53 -j DNAT --to-destination 192.168.50.2:53
sudo iptables -A FORWARD -p udp --dport 53 -d 192.168.50.2 -j ACCEPT
sudo iptables -A FORWARD -p tcp --dport 53 -d 192.168.50.2 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -p udp --dport 53 -d 192.168.50.2 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -p tcp --dport 53 -d 192.168.50.2 -j MASQUERADE
```

### 4. Habilitar el redireccionamiento de IP
```bash
sudo nano /etc/ufw/sysctl.conf
# Descomentar las líneas:
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1

# Aplicar cambios
sudo sysctl -p
```

### 5. Habilitar y verificar `ufw`
```bash
sudo ufw enable
sudo ufw status verbose
```

### 6. Verificación
Consulta DNS desde un cliente:
```bash
nslookup www.byteme.com 192.168.50.4
```
