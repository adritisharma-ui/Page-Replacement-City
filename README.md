# Memory City

**A city with limited space. Three algorithms. One goal: manage memory efficiently.**

Memory City is an interactive Operating Systems project that turns page replacement into a city-management simulation. Memory is a digital city, frames are apartments and pages are residents. When every apartment is full and a new resident arrives, FIFO, LRU or Optimal decides who has to leave.

All algorithm logic is written in **C++17** and served through a **Crow** REST API. The browser (HTML, CSS, vanilla JavaScript, Chart.js) only animates the results the C++ backend returns. No algorithm is reimplemented in JavaScript.

![Home](docs/screenshots/home.png)

## Features

- **Live city**: watch residents move in and out of apartments, with Play, Pause, Next, Previous, Restart and speed control. Page hits, faults, occupancy and a "city is full" alert update at every step.
- **Three-city comparison**: FIFO, LRU and Optimal run side by side on the same input, in sync. The algorithm with the fewest faults becomes **City of the Year** (calculated, with ties reported).
- **Analytics**: metrics table, bar charts, ranking and a written explanation, all generated from backend results.
- **Smart City Manager**: after every replacement, a plain-language reason for the eviction.
- **City Challenge**: predict which resident is evicted next at Beginner, Intermediate and Advanced levels, with scoring and feedback.
- **Input handling**: comma or space separated reference strings, presets (including Belady's anomaly), random generator and validation on both the browser and server.

| Live city | Three-city comparison |
|---|---|
| ![Live city](docs/screenshots/live-city.png) | ![Compare](docs/screenshots/compare.png) |

| Build your city | Challenge |
|---|---|
| ![Build](docs/screenshots/build.png) | ![Challenge](docs/screenshots/challenge.png) |

## The analogy

| Operating System concept | In Memory City |
|---|---|
| Main memory | The digital city |
| Frame | An apartment |
| Page | A resident |
| Page hit | The resident already lives here |
| Page fault | The resident needs a home and is not in the city |
| Page replacement | Evicting a resident to make room |

## Algorithms

| Algorithm | Evicts | Implementation |
|---|---|---|
| FIFO | The resident who entered memory first. Hits do not change the order. | `backend/fifo.cpp`, queue of arrival order |
| LRU | The resident whose last use is oldest. Hits refresh the time. | `backend/lru.cpp`, last-used time per frame |
| Optimal | The resident whose next use is farthest away. A resident never used again goes first. | `backend/optimal.cpp`, look-ahead |

Rules shared by all three: empty apartments are filled first (lowest number first). For Optimal, if several residents are never used again, the lowest apartment number is evicted.

## Tech stack

- **Backend**: C++17, [Crow](https://github.com/CrowCpp/Crow) v1.3.1 (single header, included), standalone Asio, CMake
- **Frontend**: HTML5, CSS3, vanilla JavaScript, [Chart.js](https://www.chartjs.org/) 4.4.1 (loaded from a CDN)

## Getting started (macOS)

```bash
xcode-select --install        # C++ compiler (skip if installed)
brew install cmake asio       # build tool and networking headers

git clone <your-repo-url>
cd MEMORY-CITY/backend
mkdir -p build && cd build
cmake .. && make
./run_tests                   # algorithm tests
./memory_city ../../frontend  # start the server
```

Open **http://localhost:18080**. Stop the server with `Ctrl + C`. An internet connection is needed for fonts and Chart.js.

If `backend/third_party/crow_all.h` is missing:

```bash
curl -L -o backend/third_party/crow_all.h https://github.com/CrowCpp/Crow/releases/download/v1.3.1/crow_all.h
```

### Troubleshooting

- `no type named 'io_service' in namespace 'asio'`: your Crow header is older than v1.3.1. Download v1.3.1 with the command above.
- `Address already in use`: an old server is still running. Run `pkill memory_city` and start it again.
- Blank page or 404: start the server from `backend/build` with the path `../../frontend`. Do not open `index.html` directly.

## API

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/health` | Server status |
| POST | `/api/simulate/fifo` | Run FIFO |
| POST | `/api/simulate/lru` | Run LRU |
| POST | `/api/simulate/optimal` | Run Optimal |
| POST | `/api/simulate/compare` | Run all three on the same input |

Request:

```json
{ "frames": 3, "references": [1, 2, 3, 4, 1, 2, 5] }
```

Each result contains totals (hits, faults, replacements, ratios, average utilization) and a `steps` array. Every step has the incoming page, hit or fault, frame contents after the step (`-1` = empty), evicted page (`-1` = none), frame index, explanation and running counts.

Limits: 1 to 10 frames, up to 100 references, pages 0 to 99. Invalid requests return HTTP 400 with `{"error": "..."}`.

## Project structure

```
MEMORY-CITY/
├── backend/
│   ├── main.cpp            Crow server and JSON conversion
│   ├── simulation.h        Shared Step / SimulationResult structures
│   ├── fifo.cpp / .h
│   ├── lru.cpp / .h
│   ├── optimal.cpp / .h
│   ├── tests/              Unit tests with hand-verified answers
│   ├── third_party/        crow_all.h
│   └── CMakeLists.txt
├── frontend/
│   ├── index.html
│   ├── css/                style, dashboard, city, animations
│   └── js/                 api, city, simulation, analytics, challenge, main
├── tools/verify_api.py     Cross-check against an independent Python version
└── docs/screenshots/
```

## Testing

- `./run_tests` checks the textbook string `7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1` with 3 frames (FIFO 15, LRU 12, Optimal 9 faults), Belady's anomaly string, edge cases (one frame, empty input, invalid values) and 2000 random cases confirming that hits plus faults equal the references and that Optimal never has more faults than FIFO or LRU.
- With the server running, `python3 tools/verify_api.py` compares 300 random inputs on all three algorithms with an independent implementation, frame by frame.

## Try these examples

| Input | Frames | What it shows |
|---|---|---|
| `1 2 3 4 1 2 5` | 3 | FIFO 7, LRU 7, Optimal 5 faults |
| `1 2 3 4 1 2 5 1 2 3 4 5` | 3 then 4 | Belady's anomaly: FIFO gets 9 faults with 3 frames but 10 with 4 |
| `7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1` | 3 | Classic textbook case: FIFO 15, LRU 12, Optimal 9 |

## Notes

Optimal needs to know future references, so real operating systems cannot use it. It serves as the benchmark that FIFO and LRU are compared against.

Made as an Operating Systems course project.
