# Camera condivisa tra più client (go2rtc)

## Il problema

L'endpoint RTSPS delle Bambu X1/X2/H2 accetta **una sola connessione**.
Se apri la live in Bambu Studio, Home Assistant la perde — e viceversa.
(Le P1x ne reggono 2; A1/P1P/P1S usano un protocollo diverso su porta 6000 e
non sono raggiungibili in questo modo.)

## La soluzione

**go2rtc** tiene aperta l'unica connessione consentita e la ridistribuisce a N client.

```yaml
# go2rtc.yaml
streams:
  x2d:
    - rtspx://bblp:${BAMBU_ACCESS_CODE}@192.168.1.XX:322/streaming/live/1

api:
  listen: ":1984"
rtsp:
  listen: ":8554"
```

### Credenziali

Non scrivere l'access code nel file. Con 1Password CLI:

```
# .env.tpl
BAMBU_ACCESS_CODE=op://Private/Bambu X2D/access-code
```

```bash
op run --env-file=.env.tpl -- go2rtc -config go2rtc.yaml
```

Senza 1Password: un `.env` fuori dal repo, con permessi ristretti.
In ogni caso **`.env` e `go2rtc.yaml` con segreti non vanno committati**.

### Consumo in Home Assistant

Integrazione **Generic Camera** → `rtsp://<host-go2rtc>:8554/x2d`.
Poi sostituisci `camera.MY_PRINTER_camera` con la nuova entità nel `dashboard.yaml`.

Altri endpoint utili:

| Uso | URL |
|---|---|
| MJPEG | `http://<host>:1984/api/stream.mjpeg?src=x2d` |
| Snapshot | `http://<host>:1984/api/frame.jpeg?src=x2d` |

## Trappole

1. **`rtspx://`, non `rtsps://`** — la stampante presenta un certificato self-signed;
   `rtspx` è lo schema go2rtc che salta la validazione. È la causa numero uno di
   "funziona in VLC ma non in go2rtc".
2. **Riavvia la stampante** dopo aver abilitato LAN Liveview o dopo un aggiornamento
   firmware, altrimenti lo stream si connette e cade dopo 1-3 secondi.
3. **Nessun altro client deve puntare alla stampante**: tutti a go2rtc, altrimenti
   torni al problema di partenza.
4. **Frame rate basso**: ~0.5-2 fps. Non è un guasto, sono camere da camera chiusa.
5. Su alcuni firmware recenti il toggle **LAN Only Liveview si disattiva da solo**
   dopo un update. Se un giorno lo stream muore, controlla prima quello.
