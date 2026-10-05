# DOOM — Dynamic Operations & Observability Monitor

A full-stack cloud monitoring dashboard deployed on **Google Cloud Compute Engine (IaaS)**. It turns the live health of a Google Cloud VM into a **"Cloud Pet"** that grows, gets sick or evolves based on real server performance.

> CM51207 Cloud Computing · Government Polytechnic, Pune

---

## Features

- **Live VM telemetry**: real CPU, RAM, disk, network I/O and uptime, read from the Linux kernel with `psutil`
- **Cloud Pet**: server metrics are converted into 6 pet stats shown on the dashboard
- **5 evolution stages**: the pet evolves as it earns XP and stays healthy
- **Actions**: Purge, Optimize and Harden buttons give XP and trigger cloud maintenance tasks
- **Security audit (simulated)**: scans common ports and calculates a 0–100 security score
- **Storage cleanup (simulated)**: scans temp and log folders and reports reclaimed space
- **Auto-diary**: generates human-readable log entries for each action
- **Persistent state**: pet progress is saved to `pet_state.json`
- **Failsafe mode**: if `psutil` isn't available, the app uses simulated metrics so the dashboard doesn't crash
- **Dockerized** for portable deployment

---

## How Server Metrics Become Pet Stats

| Server metric | Pet stat | Rule |
|---|---|---|
| CPU % | Energy | `100 − cpu_percent` (high CPU = low energy) |
| RAM % | Health | `100 − ram_percent` (high RAM = sick pet) |
| Disk % | Hunger | `disk_percent` (full disk = hungry pet) |
| Security score | Defense | Taken directly from the scanner |
| Open ports | Happiness | Fewer open ports = happier pet |
| Network I/O | Intelligence | More network activity = smarter pet |

## Evolution Stages

| Stage | Name | XP required | Min health |
|---|---|---|---|
| 0 | CloudEgg | 0 | 0% |
| 1 | ByteBaby | 50 | 60% |
| 2 | NimbusKid | 150 | 70% |
| 3 | CumuloBird | 350 | 80% |
| 4 | CloudDragon | 700 | 90% |

## Actions

| Button | What it does | XP |
|---|---|---|
| Purge | Runs the storage cleanup | +25 |
| Optimize | Play action | +30 |
| Harden | Secures high-risk ports and IAM roles | +40 |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3, Flask 3.0.3 |
| System metrics | psutil 5.9.8 |
| HTTP | requests 2.31.0 |
| Production server | gunicorn 21.2.0 |
| Frontend | HTML5, Tailwind CSS (CDN), Vanilla JavaScript |
| Icons | Lucide Icons (CDN) |
| State storage | JSON file (`pet_state.json`) |
| Deployment | Google Cloud Compute Engine |
| Containers | Docker, Docker Compose |

---

## Project Structure

```
doom-app/
├── app.py                 # Flask server (entry point)
├── requirements.txt       # Python dependencies
├── start.sh               # one-click Linux startup script
├── Dockerfile
├── docker-compose.yml
├── metrics/
│   ├── collector.py       # reads CPU, RAM, disk, network via psutil
│   └── calculator.py      # converts raw metrics into pet stats
├── pet/
│   ├── engine.py          # evolution logic and XP
│   ├── journal.py         # auto-diary entries
│   ├── events.py          # random cloud events
│   └── pet_state.json     # saved pet progress
├── security/
│   └── scanner.py         # simulated port scanner and firewall audit
├── storage/
│   └── cleanup.py         # simulated temp file cleaner
└── static/
    ├── index.html         # dashboard layout (bento grid)
    ├── style.css          # glassmorphism dark theme, animations
    └── app.js             # polls the API every 3 s, updates the UI
```

---

## API Endpoints

| Method | Route | Returns |
|---|---|---|
| GET | `/` | Dashboard page |
| GET | `/api/pet/status` | Live metrics, pet stats, security report |
| POST | `/api/pet/action` | Runs an action: `{"action": "feed"}` |
| GET | `/api/pet/journal` | Auto-diary entries |
| GET | `/api/pet/events` | Recent cloud events |
| GET | `/api/audit` | Root operations / command log |
| GET | `/api/pet/co-mapping` | CO1–CO6 syllabus mapping |

---

## How It Works

```
Browser (app.js) ── every 3 s ──► GET /api/pet/status
                                        │
                                   app.py (Flask)
                                        ├── metrics/collector.py  → reads real VM stats
                                        ├── metrics/calculator.py → converts to pet stats
                                        ├── security/scanner.py   → security score
                                        └── pet/engine.py         → XP, stage, mood
                                        │
Browser ◄──────────── one JSON response ┘
renderDashboard() updates all bars and numbers
```

---

## How to Run

### Local
```bash
pip install -r requirements.txt
python app.py
```
Open `http://localhost:<PORT>` (use the port set in `app.py`).

### Linux one-click
```bash
chmod +x start.sh
./start.sh
```

### Docker
```bash
docker compose up --build
```

### Google Cloud (Compute Engine)
1. Create a VM instance (Linux).
2. Install Python 3 or Docker on the VM.
3. Copy the project to the VM and run it with one of the methods above.
4. Allow the app's port in the VPC firewall rules.
5. Open `http://<VM_EXTERNAL_IP>:<PORT>`.

---

## Course Outcome Mapping

| CO | Topic | How DOOM covers it |
|---|---|---|
| CO1 | Cloud fundamentals | Pet models virtualization, elasticity and on-demand provisioning |
| CO2 | IaaS / PaaS / SaaS | Runs on IaaS (GCP VM); Flask API acts as PaaS; dashboard acts as SaaS |
| CO3 | Resource management | Real-time CPU and RAM monitoring tied to pet health |
| CO4 | Cloud storage | `pet_state.json` persistence and the storage cleanup module |
| CO5 | Cloud security | Port scanner, firewall simulation, principle of least privilege |
| CO6 | Commercial cloud | Deployed on Google Cloud, Dockerized for portability |

---

## Limitations

- The security scanner and storage cleanup are **simulated**. They don't change real firewall rules or delete files.
- If `psutil` fails to load, the dashboard shows **simulated** metrics instead of real ones.
- Pet state is stored in a single JSON file, not a database.

## Future Scope

- Real firewall changes through the GCP API
- Store state in Cloud SQL or Firestore
- Alerts by email or Slack when health drops
- Monitor multiple VMs at once
- Use WebSockets instead of polling every 3 seconds
