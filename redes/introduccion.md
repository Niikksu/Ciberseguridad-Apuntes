## Fundamentos

Las redes inalámbricas son redes de computadoras que utilizan conexiones de datos inalámbricas entre nodos de red, permitiendo que portátiles, smartphones y tabletas se comuniquen entre sí y con Internet sin cables físicos. Utilizan tecnología de radiofrecuencia (`RF`, radio frequency): cada dispositivo tiene un adaptador inalámbrico que convierte los datos en señales de RF y las envía por el aire; el resto de dispositivos reciben esas señales con sus propios adaptadores y las convierten de nuevo a un formato utilizable.

El alcance depende de la tecnología: una `LAN` inalámbrica (`WiFi`, `IEEE 802.11`) doméstica u de oficina pequeña tiene un alcance de unos cien metros, mientras que una `WWAN` (Wireless Wide Area Network) usa tecnología de telecomunicaciones móviles (`3G`, `4G LTE`, `5G`) y puede cubrir una ciudad o región entera. Para conectarse, el dispositivo debe estar dentro del alcance de la red y configurado con el nombre de red y la contraseña correctos.

La comunicación entre dispositivos se produce por RF en las bandas de `2.4 GHz` o `5 GHz`. Cuando un dispositivo (p. ej. un portátil) quiere transmitir, primero se comunica con el `WAP` (`Wireless Access Point`) para solicitar permiso. El WAP es el dispositivo central (p. ej. un router) que conecta la red inalámbrica a la red cableada y controla el acceso a la red; una vez que concede el permiso, el dispositivo transmisor envía los datos como señales de RF, que los adaptadores de los demás dispositivos reciben y convierten de vuelta a un formato utilizable para la aplicación o sistema apropiado.

La intensidad de la señal de RF y la distancia que puede recorrer dependen de la potencia del transmisor, la presencia de obstáculos y la densidad de ruido de RF en el entorno. Para garantizar una comunicación fiable a pesar de esto, las redes WiFi utilizan técnicas como la transmisión por espectro ensanchado (`spread spectrum transmission`) y la corrección de errores (`error correction`).

## Conexión y descubrimiento

El dispositivo debe estar configurado con el nombre de red / `SSID` (`Service Set Identifier`) y la contraseña. Para conectarse al router se utiliza el protocolo `IEEE 802.11`, que define los detalles técnicos de cómo los dispositivos inalámbricos se comunican entre sí y con los WAP. Cuando un dispositivo quiere unirse a una red, envía una trama de solicitud de conexión (`connection request frame`) o `association request`, que contiene:

| Campo | Descripción |
| --- | --- |
| `Dirección MAC` | Identificador único del adaptador inalámbrico del dispositivo. |
| `SSID` | Nombre de la red WiFi. |
| `Tasas de datos compatibles` | Tasas con las que el dispositivo puede comunicarse. |
| `Canales compatibles` | Frecuencias en las que el dispositivo puede comunicarse. |
| `Protocolos de seguridad compatibles` | P. ej. `WPA2` / `WPA3`. |

El dispositivo usa esta información para configurar su adaptador y conectarse al WAP. Una vez establecida la conexión, puede comunicarse con el WAP y con otros dispositivos, y acceder a Internet a través del WAP como puerta de enlace a la red cableada. Junto a `IEEE 802.11` suelen intervenir también otros protocolos como `TCP/IP`, `DHCP` y `WPA2` para asignar direcciones IP, enrutar tráfico y dar seguridad.

**El `SSID` se puede ocultar deshabilitando su difusión (`broadcast`)** — esto evita que los dispositivos que buscan redes vean ese `SSID` en la lista. Sin embargo, es una medida débil: el `SSID` sigue siendo visible en el paquete de autenticación, por lo que sigue siendo trivialmente recuperable mediante reconocimiento pasivo (capturando tráfico).

## WEP vs. WPA/WPA2/WPA3

| | WEP | WPA / WPA2 / WPA3 |
| --- | --- | --- |
| Cifrado | `RC4`, claves de `40/64/104 bits` | `AES`, claves de `128 bits` |
| Vector de inicialización | `IV` de 24 bits — pequeño, vulnerable a fuerza bruta | N/A (esquemas modernos) |
| Autenticación | `PSK` compartida — la misma clave cifra y autentica — o handshake `challenge-response` de 4 pasos con `CRC` | `PSK` (Personal) o `802.1X`/RADIUS (Enterprise) |

### WEP en detalle

`WEP` (`Wired Equivalent Privacy`) usa el algoritmo de cifrado `RC4`, lo que ya lo hace vulnerable a ciertos ataques. Existen dos versiones:

| Protocolo | IV | Clave secreta |
| --- | --- | --- |
| `WEP-40` / `WEP-64` | 24 bits | 40 bits |
| `WEP-104` | 24 bits | 80 bits |

La clave se divide en un `IV` (`Initialization Vector`) — un valor pequeño incluido en la cabecera del paquete junto con los datos cifrados, que asegura que cada clave sea única — y una `clave secreta` (bits aleatorios usados para cifrar). Como el `IV` es relativamente pequeño (24 bits), se le puede aplicar fuerza bruta probando cada combinación posible y determinar el valor correcto, lo cual permite después descifrar los datos del paquete y comprometer la red.

WEP usa además una `clave compartida` tanto para cifrado como para autenticación (a diferencia de esquemas modernos que separan ambos roles), lo que reduce el margen de error del atacante: comprometer la clave compromete ambas funciones a la vez.

#### Handshake de desafío-respuesta de WEP

Es el proceso para establecer una conexión "segura" entre un WAP y un cliente bajo WEP, mediante intercambio de paquetes:

| Paso | Quién | Descripción |
| --- | --- | --- |
| 1 | Cliente | Envía un paquete de solicitud de asociación al WAP pidiendo acceso. |
| 2 | WAP | Responde con un paquete de respuesta de asociación que incluye una cadena de desafío. |
| 3 | Cliente | Calcula una respuesta al desafío usando la clave secreta compartida y la envía de vuelta. |
| 4 | WAP | Calcula la respuesta esperada con la misma clave y envía un paquete de respuesta de autenticación. |

Como algunos paquetes pueden perderse, se integra una suma de verificación `CRC` (`Cyclic Redundancy Check`): se calcula un valor `CRC` por paquete transmitido, y el receptor lo recalcula y compara con el original para detectar corrupción de datos.

**Debilidad estructural de WEP (la que habilita los ataques prácticos)**: el diseño del mecanismo `CRC` tiene un fallo que permite descifrar un único paquete *sin* conocer la clave de cifrado. Esto ocurre porque el valor `CRC` se calcula sobre los datos en **texto plano** (`plaintext`) del paquete, no sobre los datos ya cifrados — el `CRC` se incluye en la cabecera junto a los datos cifrados, y aunque su propósito original es solo verificar integridad, puede usarse en la práctica para inferir los datos en claro del paquete incluso estando cifrado.

### WPA

`WPA` (`Wi-Fi Protected Access`) proporciona un nivel de seguridad más alto y no es susceptible a los mismos ataques que WEP. Usa métodos de autenticación más seguros, como una `PSK` (`Pre-Shared Key`) o un servidor de autenticación `802.1X`. Aunque dispositivos muy antiguos pueden no soportarlo, es compatible con la mayoría de dispositivos y sistemas modernos. Puede emplear distintos algoritmos de cifrado, incluido `AES` (`Advanced Encryption Standard`), con claves de `128 bits`.

Recomendación general de la fuente: toda red inalámbrica, especialmente en infraestructuras críticas como oficinas, debería implementar como mínimo `WPA2`, o idealmente `WPA3`.

WPA tiene dos modos:

1. **WPA-Personal** — diseñado para redes domésticas y de pequeñas empresas.
2. **WPA-Enterprise** — diseñado para organizaciones más grandes; usa un servidor de autenticación centralizado (p. ej. RADIUS o `TACACS+`) para verificar la identidad de los clientes.

## Características de seguridad de una red WiFi

Además del protocolo de cifrado en sí, una red WiFi combina varias capas de seguridad:

- **Cifrado** — algoritmos como `WEP`, `WPA2` o `WPA3` protegen la confidencialidad de los datos transmitidos.
- **Control de acceso** — por defecto se exige contraseña; puede reforzarse exigiendo además un identificador único como la dirección MAC.
- **Cortafuegos** (`firewall`) — sistema que controla el tráfico entrante y saliente según reglas predeterminadas; los routers WiFi suelen traer uno integrado que bloquea tráfico entrante desde Internet y protege contra ciberamenazas.

## Autenticación: EAP, LEAP, PEAP, TACACS+

`EAP` (`Extensible Authentication Protocol`) es el framework base de autenticación usado en diversos contextos de red. `LEAP` y `PEAP` son implementaciones concretas sobre `EAP`, pensadas para asegurar redes inalámbricas y usadas a menudo junto con WEP o WPA como capa adicional de seguridad — ver comparación detallada en [[Protocolos de Autenticación]]:

- **`LEAP`** (`Lightweight EAP`) usa una clave compartida para la autenticación — la misma clave sirve para cifrado y autenticación —, lo que hace relativamente fácil obtener acceso a la red si esa clave se ve comprometida.
- **`PEAP`** (`Protected EAP`) usa `TLS` tunelizado: establece una conexión segura entre el dispositivo y el WAP mediante un certificado digital, y un túnel cifrado protege el proceso de autenticación completo — mucho más resistente a ataques.

`TACACS+` (`Terminal Access Controller Access-Control System Plus`) se usa para autenticar y autorizar el acceso a dispositivos de red como routers y switches. Cuando un WAP envía una solicitud de autenticación a un servidor `TACACS+`, esa solicitud suele incluir credenciales del usuario y otra información de sesión, y es probable que el **paquete de solicitud completo** vaya cifrado (vía `SSL`/`TLS` o `IPsec`, según la configuración del servidor y las capacidades del WAP) para proteger su confidencialidad e integridad y evitar que sea interceptada, manipulada o sustituida por una solicitud maliciosa.

## Ataques

- **Fuerza bruta sobre el IV de WEP**: al ser de solo 24 bits, es factible romper el cifrado y descifrar paquetes.
- **Ataque de desasociación (`disassociation attack`)**: afecta potencialmente a *todas* las redes inalámbricas (no solo WEP). El WAP usa tramas de desasociación (`disassociation frames`) para desconectar legítimamente a un cliente; el ataque consiste en falsificar y enviar esas tramas a uno o más clientes para forzar su desconexión, obligándolos a volver a conectarse. Puede lanzarse tanto desde **dentro** como desde **fuera** de la red, dependiendo de la ubicación del atacante y las medidas de seguridad vigentes. Su propósito directo es interrumpir la comunicación WAP-cliente (causando inconvenientes), pero su uso más relevante ofensivamente es como **paso previo a un ataque `MITM`**: forzar la reconexión de un cliente permite capturar un nuevo handshake o exponerlo a conectarse a un punto de acceso falso, abriendo la puerta a ataques posteriores.

## Hardening

- **Deshabilitar la difusión del SSID**: cuando se difunde, el SSID se incluye en las tramas de baliza (`beacon frames`) que el WAP transmite regularmente para anunciar la red; al deshabilitarlo, el WAP deja de emitir esas balizas y la red no aparece para dispositivos que no estén ya conectados. Es una mitigación **débil**: no suficiente por sí sola, ya que el SSID sigue siendo recuperable del paquete de autenticación.
- **Preferir WPA-Enterprise sobre WPA-Personal**: usar un servidor de autenticación centralizado (RADIUS o `TACACS+`) en vez de solo una PSK compartida, especialmente en organizaciones grandes.
- **Filtrado de MAC**: permite al WAP aceptar o rechazar conexiones según la dirección MAC del dispositivo. Es una medida débil por sí sola — ver [[Direcciones MAC y ARP]] para por qué es fácilmente evadible (spoofing de MAC).
- **Implementar `EAP-TLS`**: usa certificados digitales e infraestructura de clave pública (`PKI`, Public Key Infrastructure) para verificar la identidad de los clientes y establecer conexiones seguras, proporcionando autenticación y cifrado sólidos que protegen frente a acceso no autorizado e interceptación de datos sensibles.
