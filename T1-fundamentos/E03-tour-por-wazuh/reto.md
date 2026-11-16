# Reto #3 · Investiga tu propio laboratorio

Tres partes, todas en tu [soc-lab](https://github.com/DateUnH4ck/soc-lab).
Déjame tus respuestas en los [comentarios del vídeo](https://www.youtube.com/watch?v=2MfaS0M-F3I).

## Parte 1 · La ficha de srv-ssh

En **Agents management › Summary**, abre srv-ssh y apunta:

| Dato | Tu respuesta |
|---|---|
| Sistema operativo | |
| Versión del agente | |

## Parte 2 · Tu propia alerta

Haz tu propio cambio en la web de srv-web, con la frase que quieras:

```bash
./lab.sh shell srv-web
echo '<!-- tu frase -->' >> /usr/share/nginx/html/index.html
exit
```

Encuentra la alerta en **Threat Hunting › Events** y apunta:

| Dato | Tu respuesta |
|---|---|
| Número de regla (`rule.id`) | |
| Nivel (`rule.level`) | |
| Qué línea aparece en `syscheck.diff` | |

## Extra · Una búsqueda

Escribe una búsqueda que muestre **solo las alertas de nivel 7 o más que no
vengan del manager**. La hicimos en el vídeo: intenta escribirla sin mirar.

```
tu búsqueda:
```

¿Quieres comprobar tus respuestas? Mira la [solución](solucion.md).
