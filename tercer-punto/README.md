# Tercer Punto: Cliente Linux con DNS sobre TLS
- El primer paso para lograr hacer la resolución de dominios por TLS es configurar el archivo /etc/systemd/resolved.conf
- En el archivo cambiaremos la configuración para especificar DNS que estan sirviendo por TLS y que habilitar la configuración para realizar las peticiones sobre TLS
  ```bash
  [Resolve]
  DNS=1.1.1.1 1.0.0.1
  FallbackDNS=8.8.8.8 8.8.4.4
  Domains=~.
  #LLMNR=no
  #MulticastDNS=no
  DNSSEC=yes
  DNSOverTLS=yes
  #Cache=yes
  #DNSStubListener=yes
  #ReadEtcHosts=yes
  ```
- Reiniciar los servicios o cerrar sesión               
  ```bash
  $ sudo systemctl restart NetworkManager
  # O también si este comando no funciona podemos usar para reiniciar el servicio
  $ sudo systemctl restart systemd-resolved
  ```
- Revisamos que los cambios no esten dando errores
  ```bash
  $ resolvectl status
  ```
El resultado debe verse algo parecido a esto
  ```bash
  Global
        LLMNR setting: no
  MulticastDNS setting: no
    DNSOverTLS setting: yes
        DNSSEC setting: yes
      DNSSEC supported: yes
    Current DNS Server: 1.1.1.1
          DNS Servers: 1.1.1.1
                        1.0.0.1
  Fallback DNS Servers: 8.8.8.8
                        8.8.4.4
            DNS Domain: ~.
  ```
