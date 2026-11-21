# Reto #4 · Prueba todos los usuarios

Repite el ataque en tu [soc-lab](https://github.com/DateUnH4ck/soc-lab), pero
esta vez probando **todos los usuarios de la lista** con `-L` (L mayúscula):

```bash
./lab.sh attacker
date -u
hydra -t 4 -L ~/wordlists/usuarios.txt -P ~/wordlists/passwords.txt ssh://172.30.10.11
```

Déjame tus respuestas en los [comentarios del vídeo](https://www.youtube.com/watch?v=Iv22FFnAJDk).

## Las tres preguntas

| Pregunta | Tu respuesta |
|---|---|
| ¿Cuántas cuentas caen? ¿Cuáles? | |
| ¿Qué alertas nuevas aparecen que en el vídeo no salían? (número de regla y descripción) | |
| ¿Cómo distingues en Wazuh un usuario que **existe** de uno que **no existe**? | |

## Extra · La hora

Busca en Threat Hunting el **primer acceso correcto** del ataque y apunta su
hora dos veces:

| Dato | Tu respuesta |
|---|---|
| Hora en UTC (como iría en el ticket) | |
| Hora en el dashboard (tu hora local) | |

**Pista:** apunta el `date -u` de antes de lanzar hydra y empieza a buscar desde
esa hora.

La solución la comentamos en el próximo episodio y la subiremos aquí.
