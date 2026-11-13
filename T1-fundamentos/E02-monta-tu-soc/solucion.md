# Solución del reto #2

Los tres datos no tienen una respuesta correcta: depende de tu equipo y de tu
conexión. Lo que sí tiene solución es el extra.

## ¿Por qué la Kali no puede hacer ping a Wazuh?

Porque **están en redes distintas y la Kali no tiene forma de llegar a la red
del SIEM**.

1. La Kali solo tiene una tarjeta de red, en la `dmz` (172.30.10.66). No tiene
   ninguna en la red `soc` (172.30.20.0/24), que es donde vive Wazuh.
2. Para llegar a otra red, el paquete tendría que pasar por un router. Docker
   aísla entre sí las redes que crea: el tráfico de la `dmz` hacia la `soc` se
   descarta. Por eso el ping no recibe respuesta y termina con **100 % de
   paquetes perdidos**.
3. Las víctimas sí llegan a Wazuh porque tienen una segunda tarjeta en la red
   `soc`, y es la que usan para enviar sus logs.

Está hecho a propósito. Es **segmentación de red**, lo mismo que se hace en una
empresa: los equipos expuestos a ataques no deberían llegar a las herramientas
de seguridad. Si un atacante entra en un servidor de la DMZ, no puede ver el
SIEM, ni borrar logs, ni apagar alertas.

## Para comprobarlo tú

Desde la Kali (`./lab.sh attacker`):

```bash
grep kali /etc/hosts            # la única IP de la Kali: 172.30.10.66
ping -c 2 172.30.10.11          # misma red: responde
ping -c 2 -W 2 172.30.20.10     # otra red: no responde
```
