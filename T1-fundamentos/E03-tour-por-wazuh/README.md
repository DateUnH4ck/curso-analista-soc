# E03 · Tour por Wazuh: alertas, agentes y Threat Hunting

📺 **Vídeo:** [Tour por Wazuh: aprende a leer una alerta · Curso SOC #3](https://www.youtube.com/watch?v=2MfaS0M-F3I)
🧪 **Laboratorio:** [soc-lab](https://github.com/DateUnH4ck/soc-lab) (fase 1)
🎯 **Objetivo:** saber dónde está cada cosa en Wazuh, distinguir un evento de una alerta, leer una alerta campo a campo y encontrarla con filtros.

## Requisitos previos
- Haber visto el [E02](../E02-monta-tu-soc/) y tener soc-lab montado.
- El laboratorio encendido: `./lab.sh up` (en Windows, `.\lab.ps1 up`).

En este episodio **no hay ataques**: el primero llega en el E04. Para tener
alertas que leer, hacemos dos cambios de administrador en los servidores
víctima, que Wazuh vigila en tiempo real.

## Capítulos del vídeo

| Minuto | Tema |
|---|---|
| [0:00](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=0s) | Esto es una alerta |
| [0:30](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=30s) | Lo que vamos a hacer hoy |
| [1:26](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=86s) | ¿Evento o alerta? |
| [2:14](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=134s) | Lo que no alerta, no se guarda |
| [2:43](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=163s) | Anatomía de una alerta |
| [3:42](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=222s) | Los niveles de Wazuh |
| [4:38](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=278s) | El mapa de Wazuh |
| [5:28](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=328s) | Cómo se busca |
| [6:23](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=383s) | Arrancamos el laboratorio |
| [7:15](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=435s) | Entramos en Wazuh: la portada |
| [8:07](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=487s) | Los agentes |
| [9:26](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=566s) | Inventario: IT Hygiene y puertos |
| [10:31](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=631s) | Threat Hunting |
| [11:31](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=691s) | Primer cambio: la web de srv-web |
| [13:37](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=817s) | Leemos la alerta campo a campo |
| [17:30](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1050s) | Segundo cambio: un fichero en srv-ssh |
| [19:57](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1197s) | Buscar como un analista |
| [20:42](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1242s) | Dónde viven las reglas |
| [21:30](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1290s) | Resumen |
| [22:08](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1328s) | Reto #3 |
| [23:06](https://www.youtube.com/watch?v=2MfaS0M-F3I&t=1386s) | Próximo episodio |

## Conceptos clave

**Evento frente a alerta**

```
Evento ──► Decoder ──────────► Reglas ─────────────► Alerta
una línea  entiende la línea   la compara con        solo si coincide una
de log     y saca sus campos   miles de patrones     regla de nivel 3 o más
```

De millones de eventos al día, al analista le llegan unas pocas alertas.

**Lo que no alerta, no se guarda.** En soc-lab, igual que en muchos SOC reales,
Wazuh solo guarda las alertas (`log_alert_level` 3 y `logall` desactivado). Un
evento que no coincide con ninguna regla no aparece después en Threat Hunting.
Tu visibilidad depende de tus reglas: por eso el lab trae reglas propias para
lo que Wazuh no detecta de serie.

**Toda alerta responde a cinco preguntas**

| Pregunta | Campos |
|---|---|
| ¿Cuándo y dónde? | `timestamp`, `agent.name` |
| ¿Qué regla ha saltado? | `rule.id`, `rule.description` |
| ¿Cuánto importa? | `rule.level` |
| ¿De qué tipo es? | `rule.groups`, `rule.mitre` |
| ¿Cuál es la prueba? | `full_log`; en un cambio de fichero, `syscheck.diff` |

**Los niveles de Wazuh (0 a 16)**

| Nivel | Significado | Ejemplos del laboratorio |
|---|---|---|
| 0–2 | Ruido | En soc-lab ni se guardan |
| 3–6 | Información | Inicio de sesión correcto: 3 · Contraseña fallida: 5 |
| 7–11 | Hay que mirarlo | Fichero vigilado modificado: 7 · Fuerza bruta: 10 |
| 12–16 | Urgente | DCSync (regla del lab 100100): 12 |

El nivel lo decide quien escribe la regla. Es una pista para priorizar, no la
verdad absoluta.

**El mapa de Wazuh: cinco sitios**

| Dónde | Para qué |
|---|---|
| Portada | Agentes y alertas de las últimas 24 h por severidad |
| Menú ☰ › Agents management › Summary | ¿Qué máquina es esta? Sistema, IP, versión e inventario |
| Menú ☰ › Threat intelligence › Threat Hunting | Todas las alertas: tu sitio de trabajo |
| Menú ☰ › Endpoint security | Módulos como File Integrity Monitoring y Configuration Assessment |
| Menú ☰ › Server management › Rules | De dónde sale cada alerta |

**Trucos de Wazuh 4.14 que salen en el vídeo**
- **Puertos abiertos de una máquina:** en la ficha del agente, icono ↗ de la
  tarjeta *System inventory* → IT Hygiene → Network → **Listeners**. Los dos
  puertos altos en `127.0.0.11` son el DNS interno de Docker, nada sospechoso.
- **Agente fijado:** si llegas a Threat Hunting desde la ficha de un agente,
  ese agente queda fijado (chincheta arriba a la derecha y filtro `agent.id`).
  Quítalo para ver las alertas de todas las máquinas.
- **Añadir una columna:** en *Events*, el botón **«N available fields»** (no
  *Columns*, que solo reordena). También desde el detalle de una alerta, con
  el icono *Toggle column in table*.
- **El reloj:** si una búsqueda sale vacía, revisa el rango de tiempo de arriba
  a la derecha. Los logs van en UTC y el dashboard muestra tu hora local.

## Comandos y consultas del vídeo

**Primer cambio: una línea en la web de srv-web**

```bash
./lab.sh shell srv-web
echo '<!-- Revision de contenidos -->' >> /usr/share/nginx/html/index.html
exit
```

Alerta: **550 · Integrity checksum changed.** · nivel 7 · srv-web · ATT&CK
T1565.001. En `syscheck.diff` aparece la línea añadida.

**Segundo cambio: un fichero nuevo en srv-ssh**

```bash
./lab.sh shell srv-ssh
echo 'acordarse de cambiar la contraseña' > /home/soporte/notas.txt
exit
```

Alerta: **554 · File added to the system.** · nivel 5 · srv-ssh · sin ATT&CK.

**Búsquedas en Threat Hunting › Events**

```
agent.name: srv-web
rule.groups: syscheck
rule.level >= 7
rule.level >= 7 and not agent.name: wazuh.manager
```

Con `rule.groups: syscheck`, añade la columna `syscheck.path` para ver de un
vistazo qué fichero cambió en cada alerta.

**Qué vigila el FIM en tiempo real en soc-lab**

| Servidor | Rutas |
|---|---|
| srv-web | `/usr/share/nginx/html` |
| srv-ssh | `/etc/ssh` y `/home` |

## Cómo lo verías en un SOC real

Una alerta de integridad de ficheros en un servidor web es de las más
habituales. El N1 se hace dos preguntas:

1. **¿Es real?** Sí, si los hashes de antes y de después (`syscheck.md5_before`
   y `syscheck.md5_after`) son distintos y el `syscheck.diff` muestra el cambio.
2. **¿Es malicioso?** Depende. Lo primero es comprobar si había un **cambio
   planificado** (un ticket de gestión de cambios, un despliegue de la web). Si
   lo había, se documenta como falso positivo y se cierra. Si no, puede ser el
   principio de un *defacement* o de una web shell, y se escala.

Antes de decidir, el N1 añade contexto desde el inventario del agente: qué
máquina es, qué tiene instalado y qué puertos expone.

Y una idea para el día a día: casi todas las alertas que ves sin haber hecho
nada vienen de la revisión de configuración (SCA) y del propio Wazuh. Conocer
ese ruido de fondo es lo que te permite ver lo que se sale de lo normal.

## Mapeo MITRE ATT&CK

En este episodio no hay ningún ataque: los dos cambios los hacemos nosotros como
administradores. Aun así, la alerta 550 trae una técnica asociada, porque es lo
que **podría** estar haciendo un atacante que modifica ficheros:

| Táctica | Técnica | ID |
|---|---|---|
| Impact | Data Manipulation: Stored Data Manipulation | [T1565.001](https://attack.mitre.org/techniques/T1565/001/) |

La alerta 554 (fichero nuevo) no trae técnica: no todas las alertas tienen
ATT&CK. El primer ataque de verdad, con su mapeo, llega en el E04.

## Reto
Ver [reto.md](reto.md).

## Para saber más
- [Clasificación de reglas de Wazuh (niveles 0–16)](https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html)
- [File Integrity Monitoring en Wazuh](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html)
- [T1565.001 en MITRE ATT&CK](https://attack.mitre.org/techniques/T1565/001/)
- [soc-lab en GitHub](https://github.com/DateUnH4ck/soc-lab)
