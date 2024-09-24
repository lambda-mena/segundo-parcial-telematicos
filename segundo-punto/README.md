
# Configuración de Servidor DNS Maestro/Esclavo y Autenticación PAM en Apache

## Primera Parte: Configuración de DNS Maestro/Esclavo

### Configuración del Servidor DNS Maestro (VM1)

1. **Instalar Bind9:**
   ```bash
   sudo apt-get install bind9 bind9utils bind9-doc
   ```

2. **Configurar el archivo de zona:**
   - Edita el archivo `/etc/bind/named.conf.local` para incluir la configuración del DNS maestro.
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

   - Agrega la siguiente configuración:
   ```bash
   zone "byteme.com" {
       type master;
       file "/etc/bind/db.byteme.com";
       allow-transfer { 192.168.50.2; };  # IP del esclavo
   };
   ```

### Configuración del Servidor DNS Esclavo (VM2)

1. **Instalar Bind9:**
   ```bash
   sudo apt-get install bind9 bind9utils bind9-doc
   ```

2. **Configurar el archivo de zona esclavo:**
   - Edita el archivo `/etc/bind/named.conf.local` para incluir la configuración del DNS esclavo.
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

   - Agrega la siguiente configuración:
   ```bash
   zone "byteme.com" {
       type slave;
       masters { 192.168.50.3; };  # IP del maestro
       file "/var/cache/bind/db.byteme.com";
   };
   ```

3. **Crear el Directorio para los Archivos de Zona:**
   ```bash
   sudo mkdir -p /var/cache/bind
   sudo chown bind:bind /var/cache/bind
   ```

4. **Verificar la Configuración en la Máquina Esclavo:**
   ```bash
   dig @192.168.50.2 www.byteme.com
   ```

## Segunda Parte: Configuración de Autenticación PAM en Apache

1. **Instalar Apache2:**
   ```bash
   sudo apt-get install apache2
   ```

2. **Configurar Apache para usar PAM:**
   - Edita el archivo de configuración de Apache para el sitio.
   ```bash
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```

   - Agrega la siguiente configuración para proteger el directorio `/archivos_privados`:
   ```apache
   <Directory "/archivos_privados">
       AuthType Basic
       AuthName "Restricted Content"
       AuthBasicProvider PAM
       AuthPAMService apache2
       Require valid-user
       AuthPAMUserGroup www-data
   </Directory>
   ```

3. **Habilitar la autenticación PAM en Apache:**
   ```bash
   sudo a2enmod auth_basic
   ```
