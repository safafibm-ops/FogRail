# FogRail 2.0: instrument flying for haul trucks

SIH26007 (NMDC, Bailadila) · Team LoopHole

## One line for the judges

> Pilots land in zero visibility every day. They don't see through cloud; they fly on instruments. FogRail gives every dumper a cockpit: a synthetic view of the road, a shared radar picture of the whole pit, and trucks that couple like train cars.

## Why v1 had to change

v1's headline was railway-style block signalling plus a speed governor tied to the confirmed-clear distance. It is sound, but judges can file it under things they already know:

- Commercial collision-avoidance systems (Hexagon, Cat MineStar Detect, Komatsu) already do proximity zones and speed intervention.
- Several SIH teams pitch V2V plus a dashboard, which looks the same from the outside.
- "Blocks" sounds like a rule set, not a technology.

v2 keeps v1's safety core underneath and changes the story on top. The new uniqueness comes from one borrowed idea that nobody in mining uses yet: **aviation already solved driving blind, under the name instrument flight.**

| Aviation (flying in cloud) | FogRail 2.0 (hauling in fog) |
|---|---|
| Synthetic vision display (terrain drawn from a database) | **Synthetic view HUD**: the clear-day 3D road, drawn from the truck's own position |
| TCAS: aircraft share positions and resolve conflicts together | **Fleet radar mesh**: every truck's radar feeds one shared picture of the road |
| Coupled approach and formation flight | **Fog Train**: V2V-coupled trucks with moving-block gaps and one shared token |
| ATC weather and runway visual range reports | **Weather mesh**: every truck is a visibility sensor, giving a live fog map and nowcast |
| Instrument procedures and failure checklists | **Safety core + fail-safe ladder** (kept from v1) |

## The four new pillars

### 1. Synthetic view: see the road as it looks on a clear day

- The windshield HUD (or a tablet) draws the actual haul road, bench wall, crest berm and drop-off as they look on a clear day, rendered from the truck's own RTK GNSS + IMU pose.
- The 3D road model is a **self-updating digital twin**: during every clear hour, fleet dashcams feed frames into a 3D reconstruction (Gaussian-splatting class, the 2024–26 state of the art for photoreal scene capture from ordinary video). No survey team needed.
- **Truth rule:** the scenery comes from the twin, but every live object (trucks, people, boulders) comes only from radar, thermal or V2V. Anything the twin can't vouch for is labelled. If a road has changed since the last clear pass, that stretch is drawn as **map stale, radar only**, and the speed cap tightens.
- **Why it's new:** NMDC's own trial DAS (Bacheli) enhances the camera image and reaches ~30–40 m. A synthetic view doesn't depend on the camera at all, so it shows the hairpin 300 m ahead in 3 m fog.

### 2. Fleet radar mesh: the whole fleet is one sensor

- Each truck's 77 GHz 4D imaging radar can only see ~120 m along a straight road. Around a bend the road leaves its field of view, and inside the hairpin its sight drops below 50 m.
- FogRail shares every radar scan over V2V (the same idea as the ETSI Collective Perception standard for cars, applied to a mine for the first time). A truck coming down the hairpin clears it for the truck going up.
- The governor uses the **road verified clear by any truck in the last 15 s**, not just the truck's own sensors. When a long stretch is verified clear, the blanket fog speed cap is lifted, and the stopping-distance rule still applies inside it.
- An untagged person or a rockfall seen by one truck appears on every connected truck's HUD as "seen by T4", hundreds of metres before its own sensors could see it.

### 3. Fog Train: trucks that couple like train cars

- Following trucks in V2V contact form a **moving-block** convoy, the way modern metro signalling (CBTC) replaced fixed blocks.
- The leader's brake intent reaches the follower's retrofit brake actuator in ~0.3 s instead of a 1.5 s human reaction, and the follower is allowed to count the leader's own braking distance. Gaps shrink safely.
- At single-lane hairpins and junctions, a coupled train goes through on **one token** instead of one truck at a time.
- Any truck that loses V2V, GNSS or radar drops out of the train and falls back to fixed 100 m blocks.

### 4. Weather mesh and nowcast: every truck is a weather station

- A cheap forward-scatter visibility sensor on each truck reports fog per road segment, so the control room sees a live fog map instead of one pit-wide number.
- A short-range nowcast (ML on the fleet readings plus humidity, dew point and wind) tells the control room which segment clears or closes in the next hour, so it can plan which face to load from and when to switch modes.

## What stays from v1

- **Safety core**, deterministic and offline: virtual blocks, right-of-way engine for single-lane hairpins and junctions, safe-speed governor `v·tr + v²/2g(μcosθ ± sinθ)`.
- **Zero roadside infrastructure.** No posts are needed.
- **Brand-agnostic retrofit kit**: RTK GNSS (NavIC + multi-constellation), IMU, 4D front radar, side berm radars, thermal camera, V2V radio (C-V2X sidelink), edge computer, visibility sensor, optional brake actuator for Fog Train.
- **Pathfinder mode** for brand-new roads, with supervisor approval of the map update.
- **UWB tags** for people on foot.
- **Fail-safe ladder**: GNSS lost → dead-reckoning at 15 km/h; V2V lost → dark truck, extra block held; radar fault → borrows the fleet picture at 20 km/h, or 12 km/h where none is fresh; total comms loss → controlled stop.
- **Eyes-up guidance**: HUD, haptic seat, voice cues in English and Hindi (Kannada for Donimalai).

## How it differs from what exists

| | NMDC DAS trial | OEM CAS (Cat, Komatsu, Hexagon) | Autonomous haulage | Other SIH teams | **FogRail 2.0** |
|---|---|---|---|---|---|
| Works at 3–5 m visibility | Partly (30–40 m enhanced image) | Alerts only | Yes, very costly | Mostly simulated | **Yes, keeps hauling** |
| Sees beyond its own sensors | No | No | Partly | No | **Fleet radar mesh** |
| Driver sees the road ahead | Enhanced camera | No | No driver | Lines on a screen | **Clear-day synthetic view** |
| Raises throughput, not just alerts | No | No | Yes | Rarely | **Verified-clear governor + Fog Train** |
| Mixed fleet | Yes | Brand-locked | Brand-locked | Yes | **Yes, retrofit** |
| Roadside infrastructure | None | Some | Heavy network | Often posts | **None** |

## Questions judges will ask, and the answers

- **"What if the synthetic view is wrong?"** It is scenery only. Nothing that moves is drawn from the twin, and stale stretches are flagged and slowed. The safety decisions use radar, V2V and the block logic, never the picture.
- **"Mine roads change every day."** The twin rebuilds from fleet dashcams every clear hour. A road cut overnight is flagged stale; the first truck runs as pathfinder and the supervisor approves the new map.
- **"What if the mesh shares a wrong detection?"** A false hazard only slows trucks. A missed hazard is covered by the truck's own radar and thermal camera, which always run.
- **"Fog Train sounds like autonomy."** The driver still drives. The kit only adds braking when the leader brakes, which is what Level 9 intervention in commercial CAS already does. Without the actuator the truck simply stays on fixed blocks.
- **"Cost?"** One retrofit kit per truck and no roadside hardware. The prototype costs about ₹25–40k (verify before quoting).

## Prototype (updated)

The browser simulator in [`prototype/index.html`](../prototype/index.html) now shows all four pillars:

- **Synthetic view** toggle on the HUD (clear-day 3D render vs raw fog). The Bench 3 "new road cut" event shows the stale-map warning.
- **Fleet radar mesh** on the map: radar cones, road scanned in the last 15 s, and "seen by T4" labels on the HUD. The *Untagged person inside the hairpin* event shows a truck being warned ~400 m before its own sensors could see the person.
- **Fog Train**: amber couplings on the map, a coupling banner on the HUD, and log lines such as "Fog Train T5+T2 cleared through Junction J2 on one token".
- **Weather mesh**: a drifting fog bank, per-segment visibility chips on the map, and a 60-minute nowcast per segment.
- **Compare switches** for the mesh and Fog Train, plus English/Hindi voice cues.

Simulated results, 3-hour shift, 6 dumpers, 100 t payload:

| Pit visibility | FogRail 2.0 | Vision-only | Clear-day reference |
|---|---|---|---|
| 5 m (dense fog) | 9,400 t | 3,700 t | 9,900 t |
| 25 m (fog) | 9,700 t | 9,000 t | 9,900 t |

- At 5 m the fleet radar mesh cuts the average cycle from 11.6 to 11.3 min compared with own-radar-only, with zero near-misses.
- Fog Train does not change tonnage in this small 6-truck sim, because trucks rarely bunch. Pitch it as safer following plus fewer hairpin waits, not as a throughput number, until a larger-fleet run shows one.
- All geometry, grades, limits and payloads are illustrative, not NMDC survey data.

## Hardware demo plan (1:10 fog chamber)

- Two RC trucks with ESP32, UWB tag, 24 GHz radar (HLK-LD2450), IMU, and ESP-NOW as the V2V link.
- A tablet as the HUD, running the synthetic view from the track's 3D model.
- Demos:
  1. Fog machine on, synthetic view on the tablet.
  2. Truck B's radar spots an obstacle behind the hairpin, and truck A's HUD shows it.
  3. The two trucks couple into a Fog Train.
  4. Pull V2V, and the trucks fall back to blocks.

## Slide text to update

- **Tagline:** "Pilots land blind. Our dumpers haul blind, safely."
- **Innovation:**
  - Clear-day synthetic view from a self-updating 3D twin
  - Fleet-wide shared radar picture: see around the hairpin
  - Fog Train: V2V-coupled convoys with moving-block gaps
  - Every truck is a fog sensor: live fog map and nowcast
- **Safety core:** deterministic blocks + speed governor + fail-safe ladder; AI only for reconstruction, detection and nowcasting.
