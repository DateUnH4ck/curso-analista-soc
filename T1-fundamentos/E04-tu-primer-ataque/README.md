# E04 · Tu primer ataque y tu primera alerta

📺 **Vídeo:** [Atacamos un servidor y lo cazamos en Wazuh (fuerza bruta SSH) · Curso SOC #4](https://www.youtube.com/watch?v=Iv22FFnAJDk)
🧪 **Laboratorio:** [soc-lab](https://github.com/DateUnH4ck/soc-lab) (fase 1, versión 1.0.0 o superior; mira la tuya con `./lab.sh version`)
🎯 **Objetivo:** lanzar una fuerza bruta por SSH contra tu propio laboratorio, entender qué deja en el log, encontrar la alerta que de verdad importa en Wazuh e investigarla como un N1 y un N2.

## Requisitos previos
- Haber visto el [E03](../E03-tour-por-wazuh/): sabes moverte por Wazuh y leer una alerta.
- El laboratorio encendido: `./lab.sh up` (en Windows, `.\lab.ps1 up`).

> **La regla de oro:** atacar un sistema sin permiso por escrito es un delito.
> Todo lo de este episodio es contra **tu propio laboratorio**: las víctimas
> solo son accesibles dentro de la red del lab, los puertos están atados a tu
> máquina y las contraseñas débiles están puestas a propósito.

Es el caso que contamos en el [E01](../E01-que-es-un-soc/) con un N1 y un N2;
hoy lo reproducimos de verdad.

## Capítulos del vídeo

| Minuto | Tema |
|---|---|
| [0:00](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=0s) | Introducción |
| [0:32](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=32s) | Lo que vamos a hacer hoy |
| [1:01](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=61s) | Solo contra lo que es tuyo |
| [1:25](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=85s) | ¿Qué es una fuerza bruta? |
| [2:10](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=130s) | Qué deja el ataque en el log |
| [2:42](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=162s) | De línea de log a alerta |
| [3:12](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=192s) | Misma línea, alerta distinta |
| [3:25](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=205s) | La 40112: la alerta que importa |
| [3:57](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=237s) | ¿A qué hora pasó? UTC y hora local |
| [4:24](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=264s) | Laboratorio: la línea base |
| [6:13](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=373s) | Entramos en la Kali |
| [7:00](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=420s) | Lanzamos la fuerza bruta con hydra |
| [7:52](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=472s) | Entramos como el atacante |
| [9:22](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=562s) | Volvemos al SOC: la ráfaga de alertas |
| [10:12](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=612s) | Leemos la 40112 campo a campo |
| [11:38](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=698s) | ¿Entró? Pivotamos por la IP |
| [13:06](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=786s) | El log en bruto y la hora en UTC |
| [13:39](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=819s) | Lo que Wazuh ve… y lo que no |
| [14:51](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=891s) | Esto, en un SOC real |
| [15:21](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=921s) | Lo que te llevas |
| [15:49](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=949s) | Lo que hicimos, con su nombre (ATT&CK) |
| [16:20](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=980s) | Reto #4 |
| [16:40](https://www.youtube.com/watch?v=Iv22FFnAJDk&t=1000s) | Lo que viene: el E05 |

## Conceptos clave

**Fuerza bruta: tres variantes que no hay que mezclar**

| Variante | Qué es | ATT&CK |
|---|---|---|
| Adivinar contraseñas | Un usuario, muchas contraseñas. **Es lo de hoy** | T1110.001 |
| Password spraying | Una contraseña contra muchos usuarios (llega con Active Directory) | T1110.003 |
| Credential stuffing | Usuarios y contraseñas filtrados de otras webs | T1110.004 |

Atacamos SSH porque es la puerta de administración de casi cualquier Linux: si
acepta contraseñas, una débil basta. La herramienta es `hydra`; nuestro
diccionario tiene 17 contraseñas, los de verdad tienen millones.

**Qué deja en el log.** Cada intento es una línea en `/var/log/secure` de
srv-ssh, con la hora en **UTC**:

```
… srv-ssh sshd-session[…]: Failed password for soporte from 172.30.10.66 port 58358 ssh2
… srv-ssh sshd-session[…]: Accepted password for soporte from 172.30.10.66 port 39958 ssh2
```

Resultado, usuario, IP de origen, puerto y proceso (en las versiones nuevas de
OpenSSH se llama `sshd-session`, no `sshd`). Para el servidor no hay «ataque»:
solo líneas normales repetidas muchas veces… y una que al final entra.
Convertir eso en una alerta útil es el trabajo del SIEM.

**De línea a alerta: cada regla mira su evento.** Wazuh genera una alerta por
cada línea, con la regla más concreta que encaje, y además tiene reglas que
**cuentan**:

| Regla | Nivel | Qué dice | Cuándo salta |
|---|---|---|---|
| 5760 · 5503 · 5557 | 5 | Contraseña fallida (sshd, PAM, unix_chkpwd) | En cada intento: una línea de log, una alerta |
| 2502 | 10 | User missed the password more than one time | Varios fallos seguidos |
| 5763 | 10 | sshd: brute force | 8 fallos desde la misma IP |
| **40112** | **12** | **Multiple authentication failures followed by a success** | **Fallos y después un acceso correcto desde la misma IP** |
| 5715 | 3 | sshd: authentication success | Un acceso limpio, sin fallos antes |

**La idea clave del episodio:** la misma línea `Accepted password` da la
**40112 (nivel 12)** si antes hubo fallos desde esa IP, y la **5715 (nivel 3)**
si el acceso es limpio. Misma línea, alerta distinta según el contexto.

**No toda alerta trae la IP.** La 40112 trae `data.srcip` y `data.dstuser`,
pero otras alertas clave del ataque no:

| Alerta | ¿Trae `data.srcip`? |
|---|---|
| 40112 · fallos + acceso | ✅ Sí |
| 5405 · Unauthorized user attempted to use sudo | ❌ No |
| 550 / 554 · cambios de fichero (FIM) | ❌ No |

Por eso la historia completa no se monta solo filtrando por IP: se monta
pivotando también **por servidor y por hora**.

**¿A qué hora pasó?** Los logs y la Kali van en UTC; el dashboard de Wazuh, en
tu hora local (en España, en noviembre, una hora más). Una línea de las 14:53
en el log sale a las 15:53 en el dashboard. **En el ticket, la hora siempre en
UTC y diciendo que es UTC.**

## Comandos y consultas del vídeo

**1 · La línea base, antes de atacar**

```bash
./lab.sh up
./lab.sh status
```

En **Threat Hunting › Events**, rango *Last 15 minutes* (y sin agente fijado):

```
agent.name: srv-ssh and rule.groups: authentication_failed
```

Tiene que salir **0 resultados**: así, lo que aparezca después es nuestro.

**2 y 3 · Desde la Kali, la fuerza bruta**

```bash
./lab.sh attacker
wc -l ~/wordlists/passwords.txt     # 17 contraseñas
date -u                             # apunta la hora de inicio en UTC
hydra -t 4 -l soporte -P ~/wordlists/passwords.txt ssh://172.30.10.11
```

`-t 4` son cuatro conexiones a la vez (en SSH no conviene subirlo), `-l` el
usuario y `-P` el diccionario. Resultado: `login: soporte  password: sunshine`.

**4 · Entramos como el atacante**

```bash
ssh soporte@172.30.10.11            # contraseña: sunshine
whoami
id
uname -n
cat /etc/passwd
sudo cat /etc/shadow                # soporte is not in the sudoers file
exit
exit
```

**5 a 7 · Volvemos al SOC**

```
agent.name: srv-ssh
agent.name: srv-ssh and rule.groups: authentication_failed
data.srcip: 172.30.10.66
data.srcip: 172.30.10.66 and rule.id: 40112
```

- Abre la **40112** y léela campo a campo: `rule.level` 12, `rule.mitre` con
  T1110 y T1078, `data.srcip` y `data.dstuser`.
- Abre la **5763** y busca `previous_output` en la pestaña **JSON** (no sale en
  la vista de tabla): 7 líneas de fallo, más la 8.ª en `full_log`. Justo el
  umbral de la regla.
- Añade las columnas `data.srcip` y `data.dstuser` desde **«N available
  fields»**.
- Hay más alertas que contraseñas probadas: cada intento deja varias líneas
  (sshd y PAM) y cada una puede ser una alerta.

**8 · El log en bruto**

```bash
./lab.sh shell srv-ssh
tail -n 20 /var/log/secure
grep -c "Failed password" /var/log/secure
exit
```

`./lab.sh shell` es un `docker exec`: no deja huella en el log ni genera
alertas.

**9 · Lo que Wazuh ve… y lo que no**

```
whoami
rule.groups: syscheck and agent.name: srv-ssh
```

La búsqueda de `whoami` no devuelve nada: **Wazuh no ve los comandos que se
ejecutan dentro del servidor.** Pero al cerrar la sesión, bash escribe el
historial `/home/soporte/.bash_history`, el FIM lo vigila y en el
`syscheck.diff` de su alerta aparecen los comandos del atacante.

- Si el historial **no existía**, la alerta es la **554** (*File added to the
  system*); si ya existía y cambia, la **550** (*Integrity checksum changed*).
- Es un golpe de suerte, no una prueba fiable: el historial solo se escribe al
  salir, y un atacante con cuidado lo desactiva o lo borra. Para ver de verdad
  lo que pasa dentro hacen falta **auditd o un EDR** (llegan en la temporada 3).

## Cómo lo verías en un SOC real

Un servidor con SSH abierto a internet recibe miles de intentos al día, así que
la alerta de fuerza bruta (5763), ella sola, es **ruido de fondo**. Lo que se
vigila es la **combinación**: fallos y, después, un acceso desde la misma IP.
Justo lo que marca la 40112.

El N1 responde las tres preguntas del E01 y escala:

1. **¿De dónde viene?** De una IP interna: la Kali (172.30.10.66).
2. **¿Contra qué usuario?** Contra una cuenta real: `soporte`.
3. **¿Alguno ha funcionado?** Sí. **Verdadero positivo, severidad alta.**

El N2 sigue con «¿qué hizo dentro?»: monta la línea temporal por servidor y por
hora (el sudo denegado, el historial de comandos) y decide la contención.

Cómo se evita: claves en vez de contraseñas, no exponer SSH directamente a
internet, bloquear IPs tras varios fallos (Wazuh puede hacerlo solo con su
respuesta activa) y doble factor.

## Mapeo MITRE ATT&CK

| Táctica | Técnica | ID | ¿Lo vio Wazuh? |
|---|---|---|---|
| Credential Access | Brute Force: Password Guessing | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | Sí · 5763 y 40112 |
| Initial Access | Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) | Sí · 40112 |
| Privilege Escalation | Abuse Elevation Control Mechanism | [T1548](https://attack.mitre.org/techniques/T1548/) | Sí · 5405 (sudo denegado) |
| Discovery | System Owner/User Discovery | [T1033](https://attack.mitre.org/techniques/T1033/) | No en vivo · solo en el historial (550/554) |

## Reto
Ver [reto.md](reto.md).

## Para saber más
- [T1110 Brute Force en MITRE ATT&CK](https://attack.mitre.org/techniques/T1110/)
- [Clasificación de reglas de Wazuh (niveles 0–16)](https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html)
- [File Integrity Monitoring en Wazuh](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html)
- [soc-lab en GitHub](https://github.com/DateUnH4ck/soc-lab)
