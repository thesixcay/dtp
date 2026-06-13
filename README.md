# 🔴 Laboratorio DTP Attack (Dynamic Trunking Protocol)

## 📖 Descripción

Este laboratorio demuestra cómo un atacante puede aprovechar una configuración insegura de **Dynamic Trunking Protocol (DTP)** para negociar automáticamente un enlace troncal con un switch Cisco y obtener acceso a múltiples VLANs.

DTP es un protocolo propietario de Cisco utilizado para negociar enlaces trunk entre dispositivos. Cuando un puerto está configurado en modo dinámico (`dynamic auto` o `dynamic desirable`), un atacante puede hacerse pasar por un switch legítimo y establecer un trunk sin autorización.

---

## 🎯 Objetivos

- Comprender el funcionamiento de DTP.
- Identificar configuraciones vulnerables.
- Simular un ataque DTP en un entorno controlado.
- Analizar el impacto del acceso a múltiples VLANs.
- Implementar contramedidas para mitigar el riesgo.

---

## 🖥️ Topología



## 🌐 Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP |
|------------|-----------|-------------|
| ISP | Ethernet0/0 | 200.10.100.1/24 |
| R2 | Ethernet0/0 | 200.10.100.1/24 |
| SW1 | VLAN20 | 23.6.2.2/24 |
| SW2 | VLAN20 | 23.6.2.3/24 |
| Gateway VLAN20 | - | 23.6.2.1 |

---

## 🏷️ VLANs Utilizadas

| VLAN | Descripción |
|--------|------------|
| 10 | Usuarios |
| 20 | Administración |
| 30 | VLAN Nativa |

---

# ⚙️ Configuración ISP

```cisco
hostname ISP

ip dhcp pool R
 network 200.10.100.0 255.255.255.0
 default-router 200.10.100.1
 dns-server 8.8.8.8

no ip domain lookup
ip cef
no ipv6 cef

interface Ethernet0/0
 ip address 200.10.100.1 255.255.255.0
 no shutdown

interface Ethernet0/1
 no ip address
 shutdown

interface Ethernet0/2
 no ip address
 shutdown

interface Ethernet0/3
 no ip address
 shutdown

ip route 0.0.0.0 0.0.0.0 Ethernet0/0

no ip http server
no ip http secure-server

line con 0
 logging synchronous

line vty 0 4
 login
 transport input none
```

---

# ⚙️ Configuración R2

```cisco
hostname R2

ip dhcp pool R
 network 200.10.100.0 255.255.255.0
 default-router 200.10.100.1
 dns-server 8.8.8.8

no ip domain lookup
ip cef
no ipv6 cef

interface Ethernet0/0
 ip address 200.10.100.1 255.255.255.0
 no shutdown

interface Ethernet0/1
 no ip address
 shutdown

interface Ethernet0/2
 no ip address
 shutdown

interface Ethernet0/3
 no ip address
 shutdown

ip route 0.0.0.0 0.0.0.0 Ethernet0/0

no ip http server
no ip http secure-server

line con 0
 logging synchronous

line vty 0 4
 login
 transport input none
```

---

# ⚙️ Configuración SW1

```cisco
hostname SW1

no ip domain-lookup
ip domain-name lab.local

ip cef
no ipv6 cef

username admin privilege 15 secret admin123

spanning-tree mode pvst
spanning-tree extend system-id

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 30
 switchport mode trunk

interface Ethernet0/1
 switchport access vlan 10
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 30

interface Ethernet0/2
 switchport access vlan 10
 switchport port-security maximum 2

interface Ethernet0/3
 no shutdown

interface Vlan20
 ip address 23.6.2.2 255.255.255.0
 no shutdown

ip default-gateway 23.6.2.1

no ip http server
no ip http secure-server

banner motd #
****************************************************
SW1 - ACCESO RESTRINGIDO
****************************************************
#

line con 0
 logging synchronous

line vty 0 4
 login local
```

---

# ⚙️ Configuración SW2

```cisco
hostname SW2

no ip domain-lookup
ip domain-name osvaldo.local

ip cef
no ipv6 cef

spanning-tree mode pvst
spanning-tree extend system-id

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 30
 switchport mode trunk

interface Ethernet0/1
 shutdown

interface Ethernet0/2
 switchport access vlan 10
 switchport port-security maximum 2
 spanning-tree portfast edge

interface Ethernet0/3
 shutdown

interface Vlan20
 ip address 23.6.2.3 255.255.255.0
 no shutdown

ip default-gateway 23.6.2.1

no ip http server
no ip http secure-server

line con 0
 logging synchronous

line vty 0 4
 login
```

---

# 🚨 ¿Cómo funciona el ataque DTP?

1. El atacante conecta un equipo Kali Linux a un puerto del switch.
2. El puerto vulnerable permite negociación DTP.
3. El atacante envía tramas DTP haciéndose pasar por un switch Cisco.
4. El switch negocia automáticamente un enlace trunk.
5. El atacante obtiene acceso a múltiples VLANs.
6. Puede capturar tráfico mediante herramientas como Wireshark o Scapy.

---

# 💥 Impacto

- Acceso a VLANs restringidas.
- Captura de tráfico entre segmentos de red.
- Robo de credenciales.
- Ataques Man-in-the-Middle.
- Movimiento lateral dentro de la red.

---

# 🔎 Verificación

Comandos utilizados para verificar la negociación de trunks:

```cisco
show interfaces trunk
```

```cisco
show interfaces switchport
```

```cisco
show vlan brief
```

```cisco
show dtp interface
```

---

# 🛡️ Contramedidas

Deshabilitar DTP en todos los puertos de acceso:

```cisco
interface range Ethernet0/1 - 3
 switchport mode access
 switchport nonegotiate
```

Configurar manualmente los puertos trunk:

```cisco
interface Ethernet0/0
 switchport mode trunk
 switchport nonegotiate
```

Deshabilitar puertos no utilizados:

```cisco
interface range Ethernet0/2 - 3
 shutdown
```

Habilitar Port Security:

```cisco
interface Ethernet0/2
 switchport port-security
 switchport port-security maximum 2
```

---

# 📚 Conclusión

El protocolo DTP facilita la administración de enlaces trunk en entornos Cisco, pero también puede convertirse en una vulnerabilidad cuando se deja habilitado en puertos de acceso. Un atacante puede negociar un enlace trunk no autorizado y acceder a tráfico de múltiples VLANs. La mejor práctica consiste en deshabilitar DTP y configurar manualmente todos los enlaces trunk necesarios.

# 🔐 Solución del Ataque DTP

Para evitar ataques DTP es necesario impedir que los puertos negocien automáticamente enlaces trunk.

## 1. Configurar los puertos de usuario en modo Access

```cisco
interface Ethernet0/1
 switchport mode access
```

O para varios puertos:

```cisco
interface range Ethernet0/1 - 3
 switchport mode access
```

---

## 2. Deshabilitar DTP

```cisco
interface range Ethernet0/1 - 3
 switchport nonegotiate
```

Esto evita que el switch envíe o procese mensajes DTP.

---

## 3. Configurar manualmente los enlaces Trunk

Los puertos que realmente necesiten ser trunk deben configurarse manualmente:

```cisco
interface Ethernet0/0
 switchport mode trunk
 switchport nonegotiate
```

---

## 4. Deshabilitar puertos no utilizados

```cisco
interface range Ethernet0/2 - 3
 shutdown
```

---

## 5. Habilitar Port Security

```cisco
interface Ethernet0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
```

---

## 6. Verificar la configuración

Verificar que no existan puertos negociando trunks mediante DTP:

```cisco
show interfaces trunk
```

```cisco
show interfaces switchport
```

```cisco
show dtp interface
```

---

# ✅ Resultado

Después de aplicar estas configuraciones:

- Los puertos de usuario no podrán convertirse en trunks.
- Los mensajes DTP serán ignorados.
- Un atacante no podrá negociar acceso a otras VLANs.
- Se reduce significativamente el riesgo de VLAN Hopping y acceso no autorizado a la red.
