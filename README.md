# Suricata IDS: Fundamentos, Modos y Outputs (pre-reglas)

Este proyecto documenta cómo configuré y utilicé Suricata para análisis IDS (antes del desarrollo de reglas personalizadas). Incluye ejemplos en modo offline (PCAP) y online, así como revisión de logs y buenas prácticas para operación básica.

## Objetivos
- Entender qué es Suricata (IDS/IPS/NSM) y sus modos
- Ejecutar Suricata con PCAPs y en tiempo real
- Revisar y validar configuración (`suricata.yaml`)
- Interpretar salidas principales (eve.json, fast.log, http.log, stats.log)

## ¿Qué es Suricata?
- Open-Source por OISF, con roles de IDS, IPS y NSM
- DPI hasta capa de aplicación, alto rendimiento y multihilo
- Basado en reglas; múltiples entradas (pcap, interfaces, NFQ, AF_PACKET) y salidas (EVE, logs)

## Modos de operación
| Modo | Función | Ventajas | Limitaciones |
|---|---|---|---|
| IDS | Monitoreo pasivo (alertas). | Visibilidad sin impacto en tráfico. | No bloquea. |
| IPS (NFQ) | Intercepta y bloquea. | Protección proactiva. | Requiere tuning; latencia. |
| NSM | Solo logging. | Forense/troubleshooting. | No protege en tiempo real. |

## Configuración y reglas del sistema
Rutas comunes de reglas y configuración:

```bash
ls -la /etc/suricata/rules
more /etc/suricata/suricata.yaml
```

![Listado de reglas](./images/Pasted%20image%2020250625115253.png)
![suricata.yaml](./images/Pasted%20image%2020250625115422.png)

## Inputs: Offline vs Online
- Offline (PCAP):
```bash
suricata -r /home/htb-student/pcaps/suspicius.pcap
```

![PCAP offline](./images/Pasted%20image%2020250625120137.png)

- Online (libpcap/AF_PACKET) y NFQ (IPS):
```bash
sudo suricata --pcap=ens160 -vv
sudo iptables -I FORWARD -j NFQUEUE
sudo suricata -q 0   # IPS via NFQ
sudo suricata --af-packet=ens160  # Alto rendimiento
```

![Interfaz online](./images/Pasted%20image%2020250625120437.png)
![AF_PACKET + replay](./images/Pasted%20image%2020250625121518.png)

## Dónde mirar logs
Por defecto, Suricata escribe en `/var/log/suricata/`:
- `eve.json` (JSON estructurado: alertas, HTTP, DNS, TLS, flows)
- `fast.log` (alertas básicas)
- `http.log` (eventos HTTP)
- `stats.log` (métricas de ejecución)

Ejemplos y visualización:
```bash
jq -c 'select(.event_type == "alert")' /var/log/suricata/eve.json
cat /var/log/suricata/fast.log | head
cat /var/log/suricata/http.log | head
cat /var/log/suricata/stats.log | less
```

![Logs en /var/log/suricata](./images/Pasted%20image%2020250625121613.png)
![fast.log](./images/Pasted%20image%2020250625122425.png)
![stats.log](./images/Pasted%20image%2020250625122553.png)

## Consejos rápidos
- Valida config: `suricata -T -c /etc/suricata/suricata.yaml`
- Actualiza reglas: `sudo suricata-update && sudo kill -USR2 $(pidof suricata)`
- Si vas a IPS: prueba primero en laboratorio y con reglas mínimas
- Mantén `eve.json` como fuente principal e integra con un SIEM para correlación

## Stack y requisitos
- Suricata 6+
- jq (para filtrar `eve.json`)
- PCAPs de laboratorio

## Nota
Este write-up cubre uso operativo base previo al desarrollo de reglas personalizadas (esa parte se dejó para otra práctica).
