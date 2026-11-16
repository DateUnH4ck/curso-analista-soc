# Solución del reto #3

## Parte 1 · La ficha de srv-ssh

| Dato | Respuesta |
|---|---|
| Sistema operativo | Amazon Linux 2023 |
| Versión del agente | Wazuh v4.14.9 |

Las víctimas de soc-lab se construyen sobre la imagen oficial del agente de
Wazuh, que usa Amazon Linux 2023. La versión del agente es la misma que la del
manager, la que marca `WAZUH_VERSION` en el fichero `.env` del laboratorio.

## Parte 2 · Tu propia alerta

| Dato | Respuesta |
|---|---|
| Número de regla | **550** · «Integrity checksum changed.» |
| Nivel | **7** |
| `syscheck.diff` | Tu frase, precedida de `>` |

El `>` indica una línea **añadida** al fichero. Si hubieras borrado una línea,
aparecería con `<`.

¿Por qué salta en segundos? Porque soc-lab vigila la carpeta de la web
(`/usr/share/nginx/html`) en **tiempo real**: el agente se entera del cambio en
cuanto se guarda el fichero y lo envía al manager.

Si no te sale:
- Comprueba el rango de tiempo (arriba a la derecha) y pulsa **Refresh**.
- Mira si hay un agente fijado arriba a la derecha (por ejemplo, srv-ssh). Si lo
  hay, quítalo: estarías viendo solo las alertas de esa máquina.

## Extra · Una búsqueda

```
rule.level >= 7 and not agent.name: wazuh.manager
```

- `rule.level >= 7` deja solo lo que hay que mirar.
- `and not agent.name: wazuh.manager` quita las alertas que genera el propio
  Wazuh, para quedarte con lo que pasa en los servidores.

También vale con el ID del agente del manager, que siempre es el `000`:

```
rule.level >= 7 and not agent.id: 000
```
