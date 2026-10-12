# Hive Heating Schedule (Octopus Cosy)

A Home Assistant automation that turns a Hive thermostat on/off and boosts it
on a fixed schedule, timed to match Octopus Energy's **Cosy** tariff cheap
windows (04:00–07:00, 13:00–16:00, 22:00–24:00) and expensive windows
(00:00–04:00, 07:00–13:00, 16:00–22:00).

## What it does

| Time  | Action |
|-------|--------|
| 00:00 | Heating **off** |
| 04:00 | Heat mode → **24°C** |
| 07:00 | Heating **off** |
| 13:00 | Full boost (`hive.boost_heating_on`) for 3h @ **27°C** — preheats before the long evening off-block |
| 16:00 | Heating **off** (backstop, in case the 13:00 boost already expired) |
| 22:00 | Checks the Met.no forecast low before 04:00 → if **below 5°C**, full boost for 2h @ **29°C**; otherwise Heat mode → **24°C** |

## Prerequisites

1. **Home Assistant** with the official **Hive** integration set up and
   working (`Settings → Devices & Services → Hive`), giving you a
   `climate.*` entity for your thermostat.
2. A **weather entity** that provides hourly forecasts (this was built
   against Met.no's `weather.forecast_home`, but any weather integration
   that supports `weather.get_forecasts` with `type: hourly` works).
3. A Hive Receiver model **SLR1x or SLR2x** (the `hive.boost_heating_on`
   service is part of HA's core Hive integration and needs a real Hive
   account connection — it does not work over local/offline control).

## Setup

1. Copy [`heating_schedule_octopus_cosy.yaml`](heating_schedule_octopus_cosy.yaml)
   into your Home Assistant config, or paste it directly into the UI:
   - **Settings → Automations & Scenes → + Create Automation → ⋮ → Edit in YAML**
   - Paste the file contents, then **Save**.
2. **Replace every `climate.hive`** with your own thermostat's entity ID.
   Find yours under **Settings → Devices & Services → Entities**, filter by
   `climate.`.
3. **Replace `weather.forecast_home`** with your own weather entity if it's
   named differently.
4. Adjust the triggers/temperatures to match your own tariff's cheap/expensive
   windows and comfort preferences — the times and temperatures are plain
   YAML values, easy to tweak.
5. Switch to the **visual editor** (⋮ → Edit in visual editor) to confirm it
   parses without errors before relying on it.

## Notes / gotchas

- The 22:00 branch uses a **timezone-aware** datetime comparison
  (`as_datetime` + a Jinja `for`/`if` loop) rather than comparing ISO
  strings directly — weather forecast timestamps are typically UTC, while
  `now()` is local time, so naive string comparison gives wrong results
  whenever your local offset isn't `+00:00` (e.g. during BST/DST).
- `hive.boost_heating_on` auto-expires after its `time_period`, so the
  16:00 "off" trigger is a deliberate belt-and-braces step, not strictly
  required if the preceding boost timed out exactly on schedule.
- `max_temp` on most Hive thermostats is 32°C — keep boost temperatures
  under that.

## License

GPL-3.0 — see [LICENSE](LICENSE).
