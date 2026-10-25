<div align="center">

# 🚦 Junction iQ
### Smart Traffic Management System for Urban Congestion Control

*A rule-based adaptive traffic signal controller with emergency-vehicle priority, built on SUMO, Python and TraCI.*

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)
![SUMO](https://img.shields.io/badge/SUMO-Simulator-2E7D32)
![TraCI](https://img.shields.io/badge/TraCI-API-1A237E)
![SIH](https://img.shields.io/badge/SIH-25050-orange)

**Cycle time reduced by 40.27% (118 s → 70.48 s) across 25 test scenarios**

</div>

---

## 📺 Demo Video

<!--
Option A: edit this README on GitHub, drag your .mp4 into the editor, and paste the generated link below.
Option B: use the YouTube thumbnail line (replace YOUR_VIDEO_ID) and delete Option A.
-->

   [![Watch the demo](PPT%27s/demo_thumbnail.png)](PPT%27s/Demo.mp4)

<!-- [![Watch the demo](https://img.youtube.com/vi/YOUR_VIDEO_ID/maxresdefault.jpg)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID) -->

---

## ❗ Problem

> **Smart Traffic Management System for Urban Congestion Control**, SIH 2025, Problem ID 25050 (Government of Odisha)

Fixed-timer signals ignore how many vehicles are actually waiting:

- **Fixed timers:** lights change on a countdown, even when no one is there
- **Long waits:** vehicles wait while demand differs between lanes
- **Emergency delays:** ambulances get stuck in congestion at junctions
- **Blind system:** green is often given to an empty road

**Existing solutions fall short:** fixed-time signals don't adapt, manual control needs constant monitoring, sensors give limited position data, and metro-city dynamic signals (Pune, Bengaluru) ignore vehicle type.

## 💡 Our Solution

Junction iQ reads live traffic through TraCI and decides **which lane gets green, for how long, and whether an emergency vehicle should override the cycle**.

- **Traffic-responsive timing:** green duration follows the actual count and type of vehicles
- **Vehicle-type weights:** a bus or truck needs more time than a motorcycle (inspired by the IRC PCU concept)
- **Emergency priority:** ambulances and fire trucks get a clear green path
- **Testable:** 25 scenarios were simulated and compared against fixed timing

## ✨ Key Features

- Mixed traffic: car 55%, motorcycle 25%, truck 10%, bus 5%, van 5%
- Per-lane counting by type within 80 m of the stop line
- Adaptive timer (min 10 s, max 40 s) plus a 3 s yellow phase
- Empty lanes are skipped
- Emergency detection within 100 m, with other lanes blinking red
- Tie-break: nearest emergency vehicle first; if within 2 m, ambulance before fire truck
- If the interrupted lane had ≤ 10 s of green, it restarts after the emergency
- Live on-screen count, timer and signal colour per lane; optional sound effects

## 🧠 How It Works

![Flowchart](PPT%27s/All%20Rule%20Poster.png)
1. Generate random traffic and emergency vehicles, then start SUMO
2. Count vehicles by type in each lane (E2, E4, E6, E8)
3. Calculate the timer:
   ```
   demand = Σ (weight[type] × count[type])
   timer  = clamp(ceil(demand), 10, 40)      # green = timer − 3 s yellow
   ```
4. Start from Lane 1 (E2) and cycle through lanes
5. **Emergency vehicle detected?** Give its lane green, others blink red, then yellow and continue. **No?** Continue the timer, then yellow and the next lane
6. Skip any lane with zero vehicles

| Vehicle | Weight (s) | Vehicle | Weight (s) |
|---|---:|---|---:|
| Motorcycle | 0.5 | Bus / Truck / Fire truck | 2.2 |
| Car / Ambulance | 1.0 | Van | 1.4 |

## 🛠️ Technology & Architecture

| Tool | Role |
|---|---|
| **NETEDIT** | Build roads, lanes, junctions and signals |
| **SUMO** | Free, open-source microscopic traffic simulator |
| **Python** | Rule-based logic and control |
| **TraCI** | Real-time link between Python and SUMO |

![Architecture](PPT%27s/image_1.png)

## 📊 Results

Each of the 25 cases was compared with a fixed-time cycle of **118 s**.

`Optimization (%) = (Time without logic − Time with logic) / Time without logic × 100`

| | Fixed-time | Junction iQ |
|---|:---:|:---:|
| Average cycle time | 118 s | **70.48 s** |
| Average reduction | – | **40.27 %** |

| Scenario | With logic | Improvement |
|---|:---:|:---:|
| Only one lane has vehicles | 13 s | 88.98 % |
| Less congestion, no E.V. | 30 s | 74.58 % |
| More congestion, no E.V. | 55 s | 53.39 % |
| More congestion, 1 E.V. | 84–93 s | 21–29 % |
| More congestion, 2 E.V. | 104 s | 11.86 % |
| **E.V. count more** | 165 s | **−39.83 %** |

Gains are largest when demand is uneven. Each emergency vehicle adds interruptions, and the very high E.V. case is slower than fixed timing, which is a known limitation. Full data: ![Observation Table](PPT%27s/table.png)

## ⚙️ Setup & Usage

**Requirements:** Python 3.8+, [SUMO](https://sumo.dlr.de/docs/Downloads.php), and the `SUMO_HOME` environment variable set.

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
pip install traci
pip install pygame        # optional, for sound
python mainLogic.py
```

Run without the GUI: `USE_GUI=0 python mainLogic.py` (Windows PowerShell: `$env:USE_GUI="0"; python mainLogic.py`)

Settings (vehicle count, timers, detection distance, sound) are constants at the top of `mainLogic.py`.

## 📁 Structure

```
├── mainLogic.py              # Controller: traffic, rules, signals, sound
├── Visuals.sumocfg           # SUMO config
├── *.net.xml                 # Network from NETEDIT
├── IsolationArielImage.png   # Background image
├── sounds/                   # .wav files (hum, horns, sirens)
└── docs/                     # Images, PDFs, presentations
```
`Editfile.rou.xml` and `gui_settings.xml` are generated on each run.

## 🚀 Future Scope

- **Multiple junctions:** coordinate signals across intersections
- **YOLO integration:** real-time vehicle detection and counting from cameras
- **Real-world testing:** CCTV cameras and IoT sensors
- **AI prediction:** forecast congestion and optimise timings
- **Better E.V. handling:** reduce slowdown in emergency-heavy cases

## 👥 Team & Contributions

**Group 04, PBL** | Coordinator: Mrs. Yogita Narule | Evaluator: Mr. Sanjay Patil

| Role | Contribution | Member |
|---|---|---|
| Environment | NETEDIT roads, lanes, junctions, traffic setup | `<Name>` |
| Backend Logic | Python, TraCI integration, rule-based signal logic | `<Name>`, `<Name>` |
| Documentation | Coordination, design docs, testing reports, presentations | `<Name>` |

## 🤝 Contributing

Fork the repo, create a branch (`git checkout -b feature/your-feature`), commit, push, and open a Pull Request.

---

<div align="center">

**Junction iQ**: *making every second at the signal count.* 🚦

</div>
