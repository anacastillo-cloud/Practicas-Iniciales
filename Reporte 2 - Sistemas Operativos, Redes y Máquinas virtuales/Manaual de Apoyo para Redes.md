# Manual fácil: cómo conectar dos computadoras en red (por cable y por Wi-Fi)

Este manual explica, paso a paso y sin palabras raras y sin tecnisismos, cómo conectar dos computadoras para que se "vean" entre sí y puedan compartir información. Sirve tanto si usas Windows como si usas Linux, y tanto si te conectas por cable como por Wi-Fi.

> Idea general: piensa en cada computadora como si fuera una casa. Para que el cartero le pueda entregar una carta, la casa necesita una **dirección** (eso es la "IP") y tiene que estar dentro del mismo **barrio** (eso es la "red" o "subred"). Si dos casas están en barrios distintos, el cartero normal no sabe cómo llegar.

---

## 1. Antes de empezar debes de entender esto:

No hace falta ser experto. Solo necesitas entender 4 ideas:

**1. La dirección IP** es como la dirección de tu casa dentro de la red. Ejemplo: `192.168.1.10`. Ninguna otra computadora en esa misma red debe tener exactamente esa misma dirección (si dos casas tienen la misma dirección, el cartero se confunde).

**2. La máscara de subred** es como decir "todas las casas de este barrio". Normalmente se ve así: `255.255.255.0`. En la práctica, esto significa que dos computadoras solo se pueden hablar directamente si comparten los primeros 3 números de su IP. Por ejemplo, `192.168.1.10` y `192.168.1.20` SÍ están en el mismo barrio. Pero `192.168.1.10` y `192.168.2.20` NO.

**3. El cable o el Wi-Fi** es simplemente el camino por donde viaja la información. Puede ser un cable de red (parecido a un cable de teléfono, pero más grueso) o una conexión inalámbrica (Wi-Fi).

**4. El "firewall" (cortafuegos)** es como un guardia de seguridad en la puerta de la casa. A veces este guardia bloquea visitas aunque la dirección esté bien escrita. Es la causa número 1 de que "todo esté bien configurado pero no funcione".

> **Advertencia importante:** el error más común de todos NO es escribir mal la IP. Es que el firewall (guardia de seguridad) de una de las dos computadoras está bloqueando la conexión. Si algo no funciona, revisa esto primero.

---

## 2. Conectar dos computadoras por cable

### Paso 1: Conecta el cable físicamente

Conecta un cable de red (cable de red, tipo el que usan los routers) entre las dos computadoras, o entre cada computadora y el mismo switch/router. Si conectas las dos computadoras directo con un solo cable, cualquier cable de red moderno funciona (ya no hace falta un cable especial "cruzado").

### Paso 2: Ponle una dirección (IP) a cada computadora

Ambas computadoras deben quedar en el mismo "barrio" (ver punto 1). Un ejemplo fácil que puedes copiar tal cual:

| Computadora | Dirección IP que le pones | Máscara |
|---|---|---|
| Computadora A | `192.168.1.10` | `255.255.255.0` |
| Computadora B | `192.168.1.11` | `255.255.255.0` |

**En Windows:**
1. Ve a Configuración → Red e Internet → Cambiar opciones del adaptador.
2. Clic derecho sobre tu conexión de red → Propiedades.
3. Selecciona "Protocolo de Internet versión 4 (TCP/IPv4)" → Propiedades.
4. Marca "Usar la siguiente dirección IP" y escribe los datos de la tabla de arriba.
5. Guarda los cambios.

**En Linux:**
1. Abre la configuración de red (el ícono de red arriba a la derecha, normalmente) o usa una terminal.
2. Si usas terminal, este comando le pone la IP de forma temporal (se borra si reinicias):
   ```
   sudo ip addr add 192.168.1.11/24 dev eth0
   ```
   *(`eth0` es el nombre de la tarjeta de red; si no es ese nombre, escribe `ip link show` para ver cómo se llama la tuya).*

### Paso 3: Comprueba que se "ven" entre sí

Este es el paso más importante: probar si realmente funciona.

**El comando mágico se llama `ping`.** Sirve para decir "hola, ¿me escuchas?" y ver si la otra computadora responde. Se usa igual en Windows y en Linux.

1. Abre una terminal (en Windows se llama "Símbolo del sistema" o "PowerShell"; en Linux se llama "Terminal").
2. Escribe:
   ```
   ping 192.168.1.11
   ```
   (usa la IP de la OTRA computadora, no la tuya).
3. Si ves líneas como estas, ¡funcionó!
   ```
   Respuesta desde 192.168.1.11: bytes=32 tiempo=1ms
   ```
4. Si ves "tiempo de espera agotado" o "host de destino inaccesible", algo está fallando (ve a la sección de problemas comunes, más abajo).

> Ejemplo de la vida real: es como llamar por teléfono y decir "¿me escuchas?". Si la otra persona contesta "sí, te escucho", el ping funcionó.

---

<img width="600" height="400" alt="WhatsApp Image 2026-08-13 at 9 16 49 AM" src="https://github.com/user-attachments/assets/29447d57-9d44-4ee8-a81f-5956b8c5549c" />
<img width="600" height="400" alt="WhatsApp Image 2026-08-13 at 9 16 48 AM" src="https://github.com/user-attachments/assets/c93e5feb-b00a-4e21-be52-f4afbfca88d2" />
<img width="600" height="400" alt="WhatsApp Image 2026-08-13 at 9 16 48 AM (1)" src="https://github.com/user-attachments/assets/76e5d9b8-450b-430e-b75b-699753246b08" />
<img width="600" height="400" alt="WhatsApp Image 2026-08-13 at 9 16 48 AM (2)" src="https://github.com/user-attachments/assets/9c13d709-fdca-4682-9591-047b9af2039e" />
<img width="700" height="400" alt="WhatsApp Image 2026-08-13 at 9 16 47 AM" src="https://github.com/user-attachments/assets/f00ac073-a1e2-4ccf-876c-58cedb485c49" />

---

### Paso extra: ver el camino que recorre la información

Si quieres ver por dónde "viaja" la conexión (útil cuando hay varios routers de por medio, no tanto si están conectadas directo):

- En Windows: `tracert 192.168.1.11`
- En Linux: `traceroute 192.168.1.11` (si no lo tienes instalado, instálalo con `sudo apt install traceroute`)

---

## 3. Conectar dos computadoras por Wi-Fi

Es casi lo mismo que por cable, con dos diferencias:

1. En vez de un cable, ambas computadoras deben conectarse **a la misma red Wi-Fi** (mismo nombre de red, la misma contraseña).
2. La dirección IP normalmente **no la pones tú a mano**: te la entrega automáticamente el router (esto se llama DHCP, pero no necesitas recordar ese nombre, solo que "el router la reparte solo").

### Paso 1: Conéctate a la red Wi-Fi

En Windows y en Linux es igual de sencillo: haces clic en el ícono de Wi-Fi, eliges el nombre de la red y escribes la contraseña.

> **Advertencia:** las dos computadoras deben conectarse exactamente a la **misma red**. Si tu router tiene una red de "2.4 GHz" y otra de "5 GHz" con nombres distintos, asegúrate de que ambas computadoras estén en la misma.

### Paso 2: Descubre qué dirección IP te dieron

**En Windows:**
```
ipconfig
```
Busca la línea que dice "Dirección IPv4".

**En Linux:**
```
ip addr show
```
Busca la línea que empieza con "inet" dentro de tu conexión Wi-Fi (normalmente se llama `wlan0` o algo parecido).

### Paso 3: Prueba la conexión igual que con el cable

Exactamente igual que antes: usa `ping` con la IP de la otra computadora.

```
ping 192.168.1.25
```

Si responde, ¡ya está funcionando! 🎉

---

## 4. Problemas comunes (y cómo arreglarlos, en palabras simples)

### "No me responde el ping"

| Posible causa | Cómo se explica fácil | Qué hacer |
|---|---|---|
| El guardia de seguridad (firewall) está bloqueando | Windows a veces bloquea estas "visitas" por seguridad, sobre todo si tu red está marcada como "Pública" | Cambia el tipo de red a "Privada" en Windows, o crea una regla que permita el ping (ICMP) |
| Están en barrios distintos | Tienen IP que no coinciden en los primeros números | Revisa que ambas IP empiecen igual, ejemplo: `192.168.1.x` en ambas |
| No están en la misma red física | Un cable mal conectado, o conectados a Wi-Fi distintos | Revisa el cable o que ambos estén en la misma red Wi-Fi |
| Dos computadoras con la misma dirección | Es como si dos casas tuvieran la misma dirección postal, se genera un choque | Cambia la IP de una de las dos computadoras |

### "Me sale un aviso de dirección IP duplicada"

Significa que dos computadoras están usando exactamente la misma "dirección de casa". Solución: cambia la IP de una de ellas por otra que no esté en uso, por ejemplo cambia el último número (de `.10` a `.15`).

### "Funciona de A hacia B, pero no de B hacia A"

Esto pasa más de lo que parece. No asumas que porque una computadora ve a la otra, la otra también la ve a ella. **Siempre prueba el ping en los dos sentidos.**

### "Estoy usando máquinas virtuales y no se conectan"

Revisa el modo de red de la máquina virtual. Si está en modo "NAT", generalmente NO se van a poder ver entre sí. Cámbialo a modo "Puente" (bridge) para que se comporten como computadoras normales dentro de tu red.

---

## 5. Consejos y advertencias generales

- **Anota las IP que vas a usar antes de empezar**, para no improvisar y equivocarte a mitad de camino.
- **Prueba siempre en los dos sentidos** (A hacia B, y B hacia A).
- **No muestres tu contraseña de Wi-Fi ni la de tu router** si vas a tomar capturas de pantalla.
- **El firewall es casi siempre el culpable** cuando "todo parece estar bien configurado" pero no conecta.
- **Si algo no funciona, cambia una sola cosa a la vez** y vuelve a probar. Si cambias muchas cosas juntas, no vas a saber cuál era el problema real.
- **Cuidado con redes públicas** (café, aeropuerto, universidad): muchas veces están configuradas para que los dispositivos conectados NO se vean entre sí, por seguridad. Esto no es un error tuyo, es a propósito.

---

## 6. Palabras que suenan complicadas, explicadas fácil

| Palabra | Qué significa en simple |
|---|---|
| IP / dirección IP | La "dirección de casa" de tu computadora dentro de la red |
| Máscara de subred | Indica qué otras direcciones están en tu mismo "barrio" |
| Ping | Es como un "hola, ¿me escuchas?" que le mandas a otra computadora |
| Firewall / cortafuegos | Un guardia de seguridad que decide qué conexiones deja pasar |
| DHCP | El router repartiendo direcciones IP automáticamente, sin que tú tengas que escribirlas |
| Router / punto de acceso | El aparato que reparte la conexión, por cable o por Wi-Fi |
| Wi-Fi | La versión "sin cables" de conectar computadoras a una red |

---
