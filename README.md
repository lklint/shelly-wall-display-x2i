# Shelly Wall Display X2i — Home Assistant Dashboards

Two Home Assistant dashboards built for the [Shelly Wall Display X2i](https://www.shelly.com/products/shelly-wall-display-x2i-black), a 1440×720 panoramic touchscreen. One's for monitoring solar/battery power, the other for a water pump setup — but both share the same look and structure, so they're easy to adapt to whatever you're actually monitoring.

Both dashboards use a full-bleed background image with a right-hand icon nav rail instead of Home Assistant's default page tabs, styled in a shared color palette.

> The right-hand nav rail idea was inspired by [WidowMaker/MiniPanel](https://github.com/WidowMaker/MiniPanel) — a similarly-styled X2i dashboard. See [Credits](#credits) below for more detail on what's shared and what isn't.

| | |
|---|---|
| **power-shed/** | Solar + battery monitoring, live energy flow diagram, 3 cameras |
| **pump-shed/** | Pump on/off control, tank level gauge, 7-day trend graph, 2 cameras |

---

## Before you start

You'll need:

1. **Home Assistant**, with the dashboards accessible on your local network.
2. **HACS** installed, with two frontend cards added:
   - [`button-card`](https://github.com/custom-cards/button-card) — powers the tiles, toggles, and nav rail.
   - [`card-mod`](https://github.com/thomasloven/lovelace-card-mod) — used sparingly, to style a couple of native cards (the gauge and history graph) to match.

   *HACS → Frontend → Explore & Download Repositories → search each name → Download → restart if prompted.*

3. **(Power shed only)** The native [Home Assistant Energy dashboard](https://www.home-assistant.io/home-assistant-core/en/latest/energy/) already configured with your solar/battery sensors as sources (Settings → Dashboards → Energy). The "Flow" tab won't show anything without this.

---

## Setup steps

### 1. Upload the background image

Each folder has its own `background.png` (and a `.svg` source, if you want to tweak the vector version yourself). Copy the PNG into your Home Assistant `www` folder — this is usually `/config/www/`, though depending on how you access your HA filesystem (Samba share, File Editor add-on, SSH) it may just show up as a folder called `www`.

Anything in that folder becomes accessible at `http://<your-ha-ip>:8123/local/<filename>`. Once uploaded, check it loads by visiting that URL directly in a browser — if you get a 404, you've uploaded to the wrong folder.

> Both dashboards expect the file to be named exactly `background.png`. If you rename it, update the `background: url(...)` lines in the YAML to match (there are 3–4 of these per file, one per view).

### 2. Create the dashboard

In Home Assistant: **Settings → Dashboards → + Add Dashboard → New dashboard from scratch**, give it a name, then open it and use the **three-dot menu (top right) → Edit Dashboard → three-dot menu again → Edit in YAML**. Delete whatever's there and paste in the contents of `dashboard.yaml`.

### 3. Replace the placeholder entities

Every entity ID in both files is a placeholder (e.g. `sensor.battery_state_of_charge`, `switch.bore_pump`) — they won't match anything in your system yet. Use your editor's find-and-replace to swap each one for your real entity ID. Full list below.

### 4. Set the dashboard as the display's home screen

On the X2i itself: **Device Settings → Network → Home Assistant** — it should auto-discover your HA instance on the local network, or you can enter the URL manually. Once connected, point it at the dashboard you just created.

### 5. Hide the display's own bottom bar (optional but recommended)

Recent X2i firmware lets you hide its built-in app/dock bar so the dashboard gets the full screen. Same **Settings → Network → Home Assistant** screen — look for an option to hide the app bar and disable side-scrolling. If you don't see it, check for a firmware update first.

### 6. Kiosk mode inside Home Assistant

Both YAML files already include a `kiosk_mode:` block at the top (using the [Kiosk Mode](https://github.com/NemesisRE/kiosk-mode) HACS integration — install that too if you haven't) which hides HA's own header/sidebar, since the nav rail built into these dashboards replaces them. No extra setup needed here as long as `kiosk-mode` is installed.

---

## Entities used — Power Shed

| Entity ID (placeholder) | Used for |
|---|---|
| `light.shed_light` | Shed light toggle |
| `sensor.battery_state_of_charge` | Battery % (tile + color threshold) |
| `sensor.solar_power` | Live solar output |
| `binary_sensor.solar_generating` | Solar tile's on/off tint |
| `sensor.battery_power` | Battery charge/discharge rate + direction |
| `binary_sensor.battery_charging` | Battery Flow tile's on/off tint |
| `sensor.daily_solar_yield` | Today's solar yield (kWh) |
| `sensor.pv_voltage` | PV voltage |
| `sensor.pv_current` | PV current |
| `camera.shed_camera_1` | Camera tab, feed 1 |
| `camera.shed_camera_2` | Camera tab, feed 2 |
| `camera.shed_camera_3` | Camera tab, feed 3 |

The **Flow** tab uses Home Assistant's native `energy-distribution` card and doesn't need its own entity — it pulls from whatever you've already configured in Settings → Dashboards → Energy.

## Entities used — Pump Shed

| Entity ID (placeholder) | Used for |
|---|---|
| `switch.bore_pump` | Bore pump toggle |
| `switch.transfer_pump` | Transfer pump toggle |
| `switch.pump_shed_light` | Shed light toggle |
| `sensor.tank_level_litres` | Tank level gauge + 7-day trend graph |
| `camera.pump_shed_camera_1` | Cameras tab, feed 1 |
| `camera.pump_shed_camera_2` | Cameras tab, feed 2 |

Also update the gauge's `max:` value (search for `SET THIS` in the YAML) to match your own tank's capacity in litres — it ships set to `10000` as a placeholder.

---

## Customizing

- **Colors**: both dashboards use a 4-color palette (`#fccc38`, `#f3393d`, `#17afdc`, `#5358a1`) throughout the templates and background art. Find-and-replace these hex codes to re-theme.
- **Background art**: each `background.svg` is hand-built (not AI-generated raster art) — soft corner glows, a diagonal stripe texture, and a themed motif (lightning bolts for power, water drops for pump), all layered over a dark base. Edit the SVG directly and re-export to PNG, or swap in your own image entirely.
- **Adding more tiles**: both files use two reusable `button_card_templates` — `stattile` (numeric readouts) and `toggletile` (on/off switches) — so adding another tile is usually just copying an existing block and changing the `entity`/`name`/`icon`.
- **Screen size**: built for the X2i's 1440×720 panel. Should mostly work on other panoramic displays, but tile sizing and the graph height constraints were tuned against this specific resolution.

---

## A note on graphs

Home Assistant's native chart cards (`history-graph`, and `sensor` cards with `graph: line`) render taller than you'd expect on a small screen, and neither exposes a height setting on its own. Both dashboards route around this with `card-mod` (capping height with `max-height` + `overflow: hidden`), or by giving a graph its own dedicated tab with nothing else competing for space. If you add your own graph card, keep this in mind.

---

## Credits

The right-hand nav rail concept — a fixed icon rail replacing the display's own page tabs — was inspired by [WidowMaker/MiniPanel](https://github.com/WidowMaker/MiniPanel), a Shelly Wall Display X2i dashboard also built on `button-card`.

That project uses a full absolute-position canvas with a background image, on-screen clock/weather widgets, and JavaScript templates for family presence tracking — a lot more going on than what's here. These dashboards borrow the *idea* (icon rail on the right, replacing HA's own tabs) but implement it differently: a plain CSS Grid split between a scrollable content area and a fixed-width nav column, with no absolute positioning and no shared code. Worth a look if you want on-screen clock/weather widgets or fancier per-user theming, since neither of those made it into this build.
