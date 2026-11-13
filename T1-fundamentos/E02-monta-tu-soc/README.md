# E02 · Monta tu SOC en casa con un comando (soc-lab)

📺 **Vídeo:** [Monta tu propio SOC en casa con UN comando (gratis) · Curso SOC #2](https://www.youtube.com/watch?v=ffHTpqhKk6M)
🧪 **Laboratorio:** [soc-lab](https://github.com/DateUnH4ck/soc-lab) (fase 1)
🎯 **Objetivo:** tener soc-lab funcionando en tu ordenador, entender cómo está montado por dentro y comprobar que el SIEM, las víctimas y la Kali funcionan.

## Requisitos previos
- Haber visto el [E01](../E01-que-es-un-soc/): qué es un SOC y qué hace un analista N1 y N2.
- Un ordenador con Windows, Mac o Linux, x86_64 o ARM64.
- **Docker Desktop** (en Linux, Docker Engine + Compose v2) con **8 GB de RAM** asignados (mínimo 6) y unos **20 GB de disco** libres.
- **Git**, para descargar el laboratorio.

> **¿Windows y poca RAM para Docker?** Con WSL 2, Docker usa por defecto la mitad
> de la RAM del PC. Para cambiarla, sigue la
> [guía de memoria](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/requisitos.md#cómo-dar-más-memoria-a-docker).

Guías de instalación paso a paso: [Windows](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/windows.md) ·
[Mac](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/mac.md) ·
[Linux](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/linux.md)

## Capítulos del vídeo

| Minuto | Tema |
|---|---|
| [0:00](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=0s) | Tu SOC en casa con un comando |
| [0:20](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=20s) | Lo que vamos a hacer hoy |
| [0:55](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=55s) | ¿Máquinas virtuales o contenedores? |
| [2:28](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=148s) | El plano del laboratorio |
| [3:48](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=228s) | Las dos redes: por qué el atacante no ve el SIEM |
| [4:47](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=287s) | Atacar, defender y seguridad del lab |
| [5:46](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=346s) | Requisitos y comandos |
| [7:25](https://www.youtube.com/watch?v=ffHTpqhKk6M&t=445s) | Setup paso a paso |

## Conceptos clave

**Contenedores en vez de máquinas virtuales**

| | Máquinas virtuales | Contenedores (Docker) |
|---|---|---|
| Qué llevan | Un sistema operativo completo cada una | Solo lo necesario para su servicio |
| Memoria | Cada una reserva la suya | Comparten una sola VM Linux ligera (en Mac y Windows la crea Docker Desktop) |
| Arranque | Minutos | Segundos |
| Montaje | A mano | Solo, a partir de una receta |

Por dentro, un contenedor se comporta como un ordenador: tiene su IP, sus
usuarios, sus procesos y sus logs. Para el SIEM es una máquina más de la red.

**No hay que instalar Kali.** La máquina atacante es un contenedor construido
desde la imagen oficial `kalilinux/kali-rolling`, con solo las herramientas del
curso (nmap, hydra, nikto, whatweb y diccionarios en `~/wordlists`). No tiene
escritorio: es una terminal.

**El plano y el comando.** `docker-compose.yml` describe qué máquinas hay, de
qué imagen sale cada una, en qué red está y qué IP tiene. `./lab.sh setup`
comprueba los requisitos, lee ese plano, descarga y construye lo que haga falta
y lo arranca todo: **6 contenedores** en marcha.

**Wazuh por dentro**

```
Agentes (srv-web, srv-ssh) ──► Manager ──► Indexer ──► Dashboard
  envían los logs          aplica reglas   guarda      la web donde
                           y crea alertas  y busca     trabajas
```

**Las dos redes**

| Pieza | Red `dmz` (172.30.10.0/24) | Red `soc` (172.30.20.0/24) |
|---|---|---|
| kali | 172.30.10.66 | — (no tiene tarjeta aquí) |
| srv-web | 172.30.10.10 | 172.30.20.x |
| srv-ssh | 172.30.10.11 | 172.30.20.x |
| Wazuh (manager) | — | 172.30.20.10 |

- La Kali ataca a las víctimas por la DMZ.
- Las víctimas envían sus logs a Wazuh por la red `soc`. Por eso en Wazuh verás
  srv-ssh con una IP 172.30.20.x aunque lo ataques en 172.30.10.11.
- La Kali solo está en la DMZ: **no ve el SIEM**.

**Atacar y defender**
- Atacar: `./lab.sh attacker` abre la terminal de la Kali (prompt rojo
  «kali ☠ atacante»). Para salir, `exit`.
- Defender: el navegador, en https://localhost (usuario `admin`, contraseña `SecretPassword`).

**Seguridad del laboratorio**
- Todos los puertos se publican solo en `127.0.0.1`: nadie de tu red puede entrar.
- Las víctimas tienen contraseñas débiles a propósito, y solo se llega a ellas desde dentro del lab.
- Las credenciales de Wazuh son las de serie: no lo expongas a internet ni metas datos reales.
- Ataca solo a las máquinas de este laboratorio.

## Comandos y consultas del vídeo

**Instalación**

```bash
docker --version
docker compose version
git clone https://github.com/DateUnH4ck/soc-lab.git
cd soc-lab
./lab.sh setup
```

En Windows, desde PowerShell:

```powershell
git clone https://github.com/DateUnH4ck/soc-lab.git
cd soc-lab
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned   # solo la primera vez
.\lab.ps1 setup
```

Al terminar verás **Wazuh listo** con la dirección (https://localhost), el
usuario y la contraseña.

**Comprobar que todo funciona**

```bash
./lab.sh status          # 6 contenedores en marcha y los agentes srv-web y srv-ssh activos
./lab.sh attacker        # entra en la Kali
```

Ya dentro de la Kali:

```bash
ping -c 2 172.30.10.11                      # srv-ssh responde: misma red
curl -s http://172.30.10.10 | grep title    # la web de «Hackerlab Logística S.L.»
ping -c 2 -W 2 172.30.20.10                 # el manager de Wazuh no responde
exit
```

**Los cinco comandos del curso**

| Mac / Linux | Windows (PowerShell) | Qué hace |
|---|---|---|
| `./lab.sh setup` | `.\lab.ps1 setup` | La primera vez: lo instala todo |
| `./lab.sh up` | `.\lab.ps1 up` | Arranca el laboratorio |
| `./lab.sh down` | `.\lab.ps1 down` | Lo apaga sin borrar nada |
| `./lab.sh status` | `.\lab.ps1 status` | Estado de contenedores y agentes |
| `./lab.sh attacker` | `.\lab.ps1 attacker` | Entra en la Kali |

`./lab.sh reset` borra todas las alertas y agentes, y pide confirmación. Úsalo
solo para empezar de cero.

Si algo falla: [solución de problemas](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/troubleshooting.md)
o abre un [issue](https://github.com/DateUnH4ck/soc-lab/issues) con tu sistema y la salida de `./lab.sh status`.

## Cómo lo verías en un SOC real

Lo que comprobamos con el último ping es **segmentación de red**: en una
empresa bien montada, los equipos que pueden recibir ataques (la DMZ) no
llegan a las herramientas de seguridad. Si un atacante entra en un servidor
expuesto, no debería poder ver el SIEM, ni borrar sus logs, ni apagar las
alertas.

En un SOC, además, que un agente deje de enviar logs es una señal que hay que
vigilar: un equipo que se queda en silencio puede ser una avería o alguien
tapando sus huellas. Por eso conviene revisar a menudo que los agentes están
activos, igual que hacemos con `./lab.sh status`.

## Mapeo MITRE ATT&CK

En este episodio no hay ningún ataque: solo montamos el laboratorio y
comprobamos las redes. El mapeo empieza en el E04, con el primer ataque.

## Reto
Ver [reto.md](reto.md).

## Para saber más
- [soc-lab en GitHub](https://github.com/DateUnH4ck/soc-lab)
- [Requisitos y memoria de Docker](https://github.com/DateUnH4ck/soc-lab/blob/main/docs/requisitos.md)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)
- [Componentes de Wazuh](https://documentation.wazuh.com/current/getting-started/components/index.html)
