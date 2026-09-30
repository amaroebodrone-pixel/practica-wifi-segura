# 🔐 Informe de Auditoría de Red Wi-Fi Insegura

> **Práctica:** Seguridad en el aire – cómo sobrevivir a una Wi-Fi pública
> **Rol:** Auditor de seguridad junior
> **Autor:** Amaro
> **Herramientas:** Navegador web + DevTools (F12 → pestaña *Network*), conceptos de Wireshark y VPN

---

## 1. Introducción

Cuando nos conectamos a la Wi-Fi de una cafetería, un aeropuerto o una plaza, compartimos el mismo "aire" con decenas de desconocidos. Todo lo que nuestro dispositivo envía viaja por ondas de radio hasta el router, y cualquier persona conectada a la misma red puede intentar escucharlo.

El objetivo de esta auditoría es:

- Comprobar en la práctica la diferencia entre **HTTP** y **HTTPS**.
- Identificar qué información queda **expuesta** en una conexión HTTP sin cifrar.
- Explicar cómo una **VPN** protege el tráfico mediante **cifrado**, **encapsulamiento** y un **túnel** seguro.
- Proponer **3 Reglas de Oro** para navegar de forma segura en redes públicas.

---

## 2. Sitio analizado

| Dato | Valor |
|---|---|
| **Sitio auditado** | `http://neverssl.com` |
| **Host de referencia del escenario** | `http://ejemplo-inseguro.com` |
| **Motivo de elección** | Sitio que **nunca** usa SSL/TLS, pensado para demostrar tráfico HTTP en texto plano |
| **Método de análisis** | DevTools del navegador (F12) → pestaña *Network* → recarga → primera solicitud |

> `neverssl.com` se usa como ejemplo real de un sitio inseguro. Cualquier sitio que funcione solo con HTTP, como el host hipotético `ejemplo-inseguro.com`, expondría exactamente la misma información.

---

## 3. Evidencia observada

### 3.1 Pasos realizados

1. Ingresé a `http://neverssl.com`.
2. Abrí las herramientas de desarrollador con **F12**.
3. Seleccioné la pestaña **Network (Red)**.
4. Recargué la página (**F5**).
5. Hice clic en la **primera solicitud** (el documento principal).

### 3.2 Datos de la solicitud

| Campo | Valor observado |
|---|---|
| **URL solicitada** | `http://neverssl.com/` |
| **Método HTTP** | `GET` |
| **Host** | `neverssl.com` |
| **Protocolo** | `HTTP/1.1` (sin cifrado, puerto **80**) |
| **Código de estado** | `200 OK` |

### 3.3 Headers enviados (Request Headers)

```http
GET / HTTP/1.1
Host: neverssl.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-AR,es;q=0.9,en;q=0.8
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```

---

## 4. Análisis

### 4.1 ¿Qué protocolo utiliza el sitio?

El sitio utiliza **HTTP**, no HTTPS. Se comprueba porque:

- La URL empieza con `http://` y no con `https://`.
- El navegador muestra el aviso **"No seguro"** junto a la barra de direcciones (no aparece el candado).
- En DevTools la conexión figura como `HTTP/1.1` sobre el **puerto 80**, sin negociación TLS.

Esto significa que toda la comunicación viaja en **texto plano**. Usando la analogía del curso, **HTTP es una postal escrita a mano**: cualquiera que la toque puede leerla. **HTTPS**, en cambio, es una **caja fuerte cerrada con llave**.

### 4.2 ¿Qué información puede observarse durante la solicitud?

Durante la solicitud quedan visibles, entre otros:

1. **Host**: el dominio visitado (`neverssl.com`).
2. **URL completa**: incluida la ruta exacta y cualquier parámetro (por ejemplo `?usuario=...&busqueda=...`).
3. **Método HTTP**: `GET` (o `POST` si se enviara un formulario, con **todo su contenido** visible).
4. **User-Agent**: revela el sistema operativo, el navegador y su versión.
5. **Headers**: idioma preferido (`Accept-Language`), tipos de contenido aceptados y **cookies** si las hubiera.
6. **El contenido completo de la página** que devuelve el servidor.

### 4.3 Riesgos encontrados al usar HTTP en una Wi-Fi pública

| Riesgo | Descripción | Qué podría obtener el atacante |
|---|---|---|
| **Sniffing (intercepción)** | Alguien en la misma red captura los paquetes con herramientas como **Wireshark** | URLs, formularios, usuarios y contraseñas enviados por HTTP |
| **Man-in-the-Middle (MitM)** | El atacante se hace pasar por el router (por ejemplo, con *ARP spoofing*) y todo el tráfico pasa por su equipo | Puede **leer y además modificar** las páginas: inyectar publicidad, malware o formularios falsos |
| **Evil Twin (red gemela maligna)** | Hotspot falso con nombre creíble, como `Starbucks_Gratis` | Control total del tráfico desde el primer momento |
| **Robo de sesión (session hijacking)** | Se capturan las **cookies de sesión** que viajan sin cifrar | Entrar a la cuenta de la víctima **sin conocer la contraseña** |
| **Perfilado y pérdida de privacidad** | Se registra qué sitios se visitan y en qué horarios | Hábitos, intereses, dispositivo usado e identidad |

**Mitos desmentidos:**

- *"Si la red tiene contraseña, es segura"*: **falso**. Si la clave está impresa en el ticket, todos los clientes tienen la misma "llave" y pueden atacar a los demás desde adentro.
- *"Solo me hackean si descargo algo"*: **falso**. Con solo navegar por sitios HTTP ya se exponen la identidad, los hábitos y las sesiones abiertas.

---

## 5. Cómo ayuda una VPN

Una **VPN (Red Privada Virtual)** crea un **túnel** seguro entre el dispositivo y el servidor VPN. Funciona como un **"túnel blindado"** que atraviesa la red pública.

```
SIN VPN
[Tu PC] ──── HTTP en texto plano ────> [Router Wi-Fi público] ───> Internet
                     👀 El atacante lee todo

CON VPN
[Tu PC] ══════ TÚNEL CIFRADO ══════> [Router Wi-Fi] ══════> [Servidor VPN] ───> Internet
                     👀 El atacante solo ve "ruido" ilegible
```

### 5.1 Cifrado

Antes de salir del dispositivo, **todo el tráfico se cifra** con algoritmos robustos (por ejemplo **AES-256** o **ChaCha20**, según el protocolo: OpenVPN, WireGuard o IPsec). Aunque el atacante capture los paquetes con Wireshark, solo verá **datos cifrados ilegibles**.

### 5.2 Túnel seguro y encapsulamiento

La VPN aplica **encapsulamiento**: cada paquete original (con su Host, URL, headers y contenido) se **envuelve completo dentro de otro paquete cifrado** dirigido al servidor VPN. En la red local solo se ve que existe una conexión hacia el servidor VPN, sin saber qué sitios se visitan ni qué se envía.

### 5.3 Protección del tráfico

- Los ataques **Man-in-the-Middle** y **Evil Twin** pierden efectividad porque el atacante no puede leer ni modificar el contenido del túnel sin la clave.
- Las **cookies de sesión** y las credenciales quedan protegidas en el tramo de la red pública.
- Incluso el tráfico de un sitio **HTTP** como `neverssl.com` queda cifrado **dentro del túnel** mientras atraviesa la Wi-Fi pública.

### 5.4 Privacidad

- El dueño de la red, el router y los demás clientes **no pueden ver qué sitios se visitan**.
- Las **consultas DNS** también viajan por el túnel, si la VPN está bien configurada.
- Los sitios de destino ven la **IP del servidor VPN**, no la IP real del usuario.

### 5.5 Limitación importante

La VPN protege el tramo **entre el dispositivo y el servidor VPN**. Desde el servidor VPN hasta un sitio HTTP, el tráfico vuelve a viajar **sin cifrar**. Por eso la VPN **complementa** a HTTPS, no lo reemplaza. Además, hay que usar un **proveedor VPN confiable**, porque este sí puede ver el tráfico que sale del túnel.

---

## 6. 🏆 Mis 3 Reglas de Oro para redes Wi-Fi públicas

### 🥇 Regla 1: Si no hay candado, no hay datos
Solo ingresar contraseñas, datos personales o bancarios en sitios con **HTTPS** (candado 🔒 en la barra de direcciones). Activar el modo **"Solo HTTPS"** en el navegador y **nunca aceptar advertencias de certificado inválido** en una red pública.

### 🥈 Regla 2: Siempre dentro del túnel
Activar una **VPN confiable** antes de navegar en cualquier red pública. De esta forma todo el tráfico viaja **cifrado y encapsulado**, y el atacante de la mesa de al lado solo ve ruido ilegible.

### 🥉 Regla 3: Verificar la red y reducir la exposición
Confirmar con el personal el **nombre exacto de la red** para evitar Evil Twins. Desactivar la **conexión automática** a redes abiertas y el uso compartido de archivos, **olvidar la red** al irse y evitar operaciones sensibles (home banking, compras). Para esas operaciones, conviene usar los **datos móviles** del celular.

---

## 7. Conclusión

La auditoría demostró que un sitio servido por **HTTP**, como `neverssl.com`, expone en texto plano el host, la URL, el método, los headers y el User-Agent, y que cualquier persona en la misma Wi-Fi pública podría capturarlos con una herramienta como Wireshark. La combinación de **HTTPS** y una **VPN**, junto con hábitos de navegación responsables, reduce drásticamente estos riesgos.

> **Conectarse a una Wi-Fi pública es como hablar en voz alta en una plaza: si no querés que te escuchen, usá un túnel cifrado.**

---

### 📚 Referencias
- [NeverSSL](http://neverssl.com): sitio de prueba sin HTTPS
- [MDN Web Docs: HTTP](https://developer.mozilla.org/es/docs/Web/HTTP)
- [Wireshark: documentación oficial](https://www.wireshark.org/docs/)
- Material del curso: Módulo 3 (HTTP vs HTTPS) y Unidad final (Wi-Fi pública y VPN)
