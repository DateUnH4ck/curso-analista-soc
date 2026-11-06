# E01 · Qué es un SOC y qué hace un analista N1 y N2

📺 **Vídeo:** [Qué hace REALMENTE un analista SOC (N1 vs N2) · Curso SOC #1](https://www.youtube.com/@DateUnH4ck)
🧪 **Laboratorio:** todavía no hace falta; montamos [soc-lab](https://github.com/DateUnH4ck/soc-lab) en el E02
🎯 **Objetivo:** entender qué es un SOC, cómo llega una alerta hasta un analista y qué hace cada nivel (N1, N2 y N3) con ella.

## Requisitos previos
- Ninguno. Si quieres saber cómo está organizado el curso, mira antes el E00.

## Capítulos del vídeo

| Minuto | Tema |
|---|---|
| 0:00 | Gancho: un ataque a las 3 de la mañana |
| 1:38 | Presentación del curso |
| 2:21 | Qué es un SOC |
| 6:53 | Niveles N1, N2 y N3 |
| 9:54 | El caso: la misma alerta vista por un N1 y un N2 |
| 13:37 | Qué piden las ofertas en España |
| 15:02 | Reto y próximo episodio |

## Conceptos clave

**SOC** (Security Operations Center): el equipo que protege a una organización
las 24 horas. Su trabajo se resume en tres verbos: **vigila, detecta y responde**.

**Las tres patas de un SOC**

| Pata | Qué es |
|---|---|
| Personas | Los analistas, organizados por niveles (N1, N2, N3) |
| Procesos | Playbooks, tiempos de respuesta pactados y reglas de escalado |
| Tecnología | Las herramientas que recogen logs, lanzan alertas y gestionan casos |

**De los logs al analista**

```
Fuentes (equipos, servidores, firewalls, cloud)
        │  logs
        ▼
SIEM (junta, ordena y guarda los logs)  +  EDR (vigila dentro de cada equipo)
        │  alerta
        ▼
Analista (revisa, investiga y decide)
```

En el curso el SIEM principal es **Wazuh**. Más adelante veremos también
Elastic, Splunk y Microsoft Sentinel.

**El ciclo del SOC:** recoger → detectar → revisar → investigar → responder → mejorar → vuelta a empezar.

**Dónde están los SOC**
- **Interno:** el equipo propio de una empresa grande.
- **Gestionado (MSSP):** un proveedor externo que vigila a muchos clientes a la vez.
- **Híbrido:** una parte interna y otra externalizada.

Buena parte de los puestos de N1 en España están en SOC gestionados y
consultoras, y lo normal es trabajar a turnos.

**Los niveles** (N1, L1 y Tier 1 son lo mismo)

| | N1 | N2 | N3 |
|---|---|---|---|
| Pregunta | ¿Es real? | ¿Qué ha pasado y hasta dónde? | ¿Cómo lo paramos y evitamos que vuelva? |
| Trabajo | Triaje, contexto, cierre de falsos positivos y escalado | Investigación, correlación entre herramientas y contención | Incidentes mayores, forense, malware avanzado, detecciones nuevas |
| Experiencia orientativa | Primer empleo | 2-3 años en SOC | Muchos años de incidentes reales |

- **Falso positivo:** la alerta salta pero no hay ataque. Se documenta el porqué y se cierra.
- **Verdadero positivo:** la alerta corresponde a un ataque real. Se sigue el playbook y se escala.

## El caso del episodio

Alerta: **fuerza bruta por SSH** contra `srv-ssh`, desde `172.30.10.66`, contra el usuario `soporte`.

### Lo que hace el N1

Tres preguntas antes de nada:

1. **¿Qué máquina es la de origen?** Una IP privada: el ataque sale de dentro de la red.
2. **¿Contra qué usuario?** `soporte`, una cuenta que existe en el servidor.
3. **¿Ha funcionado algún intento?** Sí: después de la ráfaga de fallos hay un inicio de sesión correcto.

Veredicto: **verdadero positivo, severidad alta**. Toca escalar con un ticket
que el N2 pueda continuar sin preguntar nada:

```
Alerta:        Fuerza bruta por SSH
Hora:          03:13:59
Origen:        172.30.10.66 (IP interna)
Destino:       srv-ssh
Usuario:       soporte
Prueba:        inicio de sesión correcto de soporte justo después de la ráfaga de fallos
Comprobado:    la IP es interna, la cuenta existe y el acceso se ha conseguido
Clasificación: verdadero positivo · severidad alta
Acción:        escalado a N2 según el playbook
```

### Lo que hace el N2

1. **Línea temporal:** cuándo empezó, cuándo entró y qué pasó después.
2. **Qué hizo dentro:** comandos, intentos de `sudo` y persistencia (tareas programadas, claves SSH nuevas).
3. **Alcance:** la misma IP también escaneó `srv-web`.
4. **Contención:** bloquear la IP, deshabilitar la cuenta `soporte` o cambiar su contraseña y, si hace falta, aislar el servidor.
5. **MITRE ATT&CK:** fuerza bruta, cuenta válida y acceso remoto por SSH (tabla de abajo).
6. **Mejora:** desactivar el acceso SSH con contraseña y usar claves.

Este mismo ataque lo lanzamos y lo cazamos en Wazuh en el E04.

## Comandos y consultas del vídeo

Este episodio es teórico: no hay comandos. El laboratorio se monta en el E02.
Si quieres adelantarte, las instrucciones están en
[soc-lab](https://github.com/DateUnH4ck/soc-lab).

## Cómo lo verías en un SOC real

En una muestra de ofertas de analista SOC en España (es una orientación, no un estudio):

- **N1:** manejar un SIEM (Splunk, QRadar, Microsoft Sentinel o Elastic) y un
  EDR (Microsoft Defender o CrowdStrike), trabajar con tickets y playbooks y
  documentar bien. El inglés aparece casi siempre y, en Barcelona, también el
  catalán. En algunas ofertas suman las certificaciones ofensivas y los CTF.
- **N2:** 2-3 años de experiencia en SOC, investigación a fondo, análisis de
  malware para sacar indicadores, crear y ajustar reglas de detección, threat
  hunting, scripting, automatización y cloud.

Microsoft Sentinel y Microsoft Defender salen en muchas ofertas, así que en el
curso también veremos KQL y Splunk.

## Mapeo MITRE ATT&CK

| Táctica | Técnica | ID |
|---|---|---|
| Credential Access | Brute Force: Password Guessing | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) |
| Initial Access / Persistence | Valid Accounts: Local Accounts | [T1078.003](https://attack.mitre.org/techniques/T1078/003/) |
| Lateral Movement | Remote Services: SSH | [T1021.004](https://attack.mitre.org/techniques/T1021/004/) |

## Reto
Ver [reto.md](reto.md).

## Para saber más
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Documentación de Wazuh](https://documentation.wazuh.com/current/index.html)
