# Trovare i propri entity_id

I nomi delle entità di `ha-bambulab` **variano** in base a modello, versione
dell'integrazione, modalità di connessione e lingua di Home Assistant.
Non fidarti di liste trovate online (nemmeno di questa): leggile dal tuo sistema.

## Dump completo

**Strumenti per sviluppatori → Modello**, incolla:

```jinja
{% for e in states %}{{ e.entity_id }}
{% endfor %}
```

Troppo lungo? Filtra per il nome della stampante:

```jinja
{% for e in states if 'x2d' in e.entity_id or 'ams' in e.entity_id %}{{ e.entity_id }}
{% endfor %}
```

## I 4 segnaposto da sostituire

| Segnaposto | Come riconoscerlo | Esempio reale |
|---|---|---|
| `MY_PRINTER` | prefisso della maggior parte dei sensori | `sensor.`**`x2d_soggiorno`**`_print_status` |
| `MY_AMS1` | device AMS separato | `sensor.`**`ams_01`**`_tray_1` |
| `MY_AMS2` | secondo AMS | `sensor.`**`ams_02`**`_tray_1` |
| `MY_SERIAL` | contiene il numero di serie | `sensor.`**`x2d_00m09a123456789`**`_externalspool_external_spool` |

> Attenzione: `MY_SERIAL` **non** coincide con `MY_PRINTER`. Le entità delle bobine
> esterne usano un prefisso diverso, col seriale. Se reinstalli l'integrazione,
> quel prefisso cambia e vanno riaggiornate.

## Attributi di uno slot AMS

Per vedere cosa puoi mostrare nel sottotitolo degli slot:

**Strumenti per sviluppatori → Stati** → cerca `sensor.MY_AMS1_tray_1` → colonna attributi.

Tipici: `color` (formato `RRGGBBAA`, 8 cifre), `remain`, `nozzle_temp_min`,
`nozzle_temp_max`, `type`, `name`, `empty`, `unknown`.

Semantica di `empty` / `unknown` secondo la documentazione dell'integrazione:

| `empty` | `unknown` | Significato |
|---|---|---|
| `true` | — | nessuna bobina utilizzabile (slot vuoto, in caricamento, o scansione RFID in corso) |
| `false` | `true` | bobina presente ma non configurata (sulla stampante appare `?`) |
| `false` | `false` | bobina presente con profilo corretto |

## Entità che potrebbero non esistere

Dipende da modello e modalità di connessione:

| Entità | Condizione |
|---|---|
| `button.*`, `number.*`, `select.*` | solo in **LAN Only Mode** |
| `camera.*` | richiede **LAN Only Liveview** |
| `image.*_cover_image` | richiede **Store Sent Files on External Storage** + USB |
| `fan.*` | **non esiste** su H2/X2: solo `sensor.*_fan_speed` |
| `update.*` | non sempre presente |
| `*_left_nozzle_*` / `*_right_nozzle_*` | solo stampanti dual-nozzle |
