# AquaTherm

**AquaTherm: An IoT-Based Water Quality Monitoring and Early Warning System for Bangus Fishponds**

A Thesis 1 project for the Bachelor of Science in Computer Science program, College of Computer Science, Don Mariano Marcos Memorial State University – South La Union Campus.

This repository is the central storehouse for all files related to the AquaTherm project — diagrams, documentation, and (in later phases) source code — for version control, adviser/panel checking, validation, and testing throughout the study.

## About AquaTherm

Milkfish (bangus) farming in La Union remains vulnerable to sudden water temperature and salinity swings that farmers currently have no fast way of detecting. AquaTherm is an IoT-based water quality monitoring and early warning system built for a milkfish fishpond in Barangay Tococ, Sto. Tomas, La Union. It uses an ESP32 sensor node fitted with a waterproof digital temperature sensor (DS18B20) and a salinity/conductivity sensor to continuously monitor pond conditions, automatically classifies readings as **Safe**, **Warning**, or **Critical**, and notifies the fishpond owner through a **dual-channel alert module** — SMS and mobile push notification — the moment conditions turn unsafe.

## Repository Structure

```
AquaTherm/
└── Charts & Diagrams/
    ├── 01_DFD.md                  — Data Flow Diagram (Level 0, 1, 2)
    ├── 02_Structured_Chart.md     — Structured (Hierarchy) Chart
    ├── 03_HIPO_Diagram.md         — HIPO Diagram + IPO Table
    ├── 04_Structured_English.md   — Structured English (Processes 2.0 & 3.0)
    ├── 05_Pseudo_Code.md          — Pseudocode (sensor node + server logic)
    ├── 06_ERD.md                  — Entity Relationship Diagram
    └── 07_Data_Dictionary.md      — Data Dictionary
```

## Charts & Diagrams

All charts and diagrams for Activity 1 are found in the **`Charts & Diagrams/`** folder. Each file contains its diagram (rendered from Mermaid syntax) followed by a **detailed explanation** of its components, purpose, and how it connects to the rest of the system design.

| # | Document | Description |
|---|---|---|
| 01 | [Data Flow Diagram](./Charts%20%26%20Diagrams/01_DFD.md) | Shows how data (sensor readings, alerts, reports) moves between AquaTherm and its external entities, across three levels of detail (Context, Level 1, Level 2). |
| 02 | [Structured Chart](./Charts%20%26%20Diagrams/02_Structured_Chart.md) | Breaks AquaTherm down into its five functional modules and their sub-functions. |
| 03 | [HIPO Diagram](./Charts%20%26%20Diagrams/03_HIPO_Diagram.md) | Combines a hierarchy chart with an Input-Process-Output table for each major process. |
| 04 | [Structured English](./Charts%20%26%20Diagrams/04_Structured_English.md) | Describes the water-quality classification and alert-generation logic in controlled, program-like statements. |
| 05 | [Pseudocode](./Charts%20%26%20Diagrams/05_Pseudo_Code.md) | Implementation-level logic for the ESP32 sensor node's monitoring loop and the server's classification/alerting functions. |
| 06 | [Entity Relationship Diagram](./Charts%20%26%20Diagrams/06_ERD.md) | Models the database entities (USER, FISHPOND, SENSOR_DEVICE, SENSOR_READING, ALERT, NOTIFICATION_LOG, NOTIFICATION_SETTING) and their relationships. |
| 07 | [Data Dictionary](./Charts%20%26%20Diagrams/07_Data_Dictionary.md) | Field-level reference (name, type, size, constraint, description) for every table in the ERD. |

> **Note on this revision:** These documents were updated following the adviser's feedback on the finalized Statement of Objectives. AquaTherm's monitored parameters are **water temperature** (via a DS18B20 waterproof digital temperature sensor) and **salinity** — dissolved oxygen (DO), used in the earlier draft, was dropped from scope, since DO sensors require a chemical filling solution and a replaceable membrane that add cost and maintenance beyond what is feasible for this undergraduate thesis. Each file above marks exactly what changed in its own **Revision Note**.

## Team

**Researchers**
- Avila, Kent Ivan B. (Leader)
- Bacal, Emmanuel Joseph O.
- Culminas, Gerardo R.
- Daz, Yuan Dec L.
- Llamido, John Michael J.

**Adviser:** Suguitan, Agnes S., DIT(c)

All group members are added as **collaborators** on this repository so that any member can commit, review, and maintain project files throughout Thesis 1 and Thesis 2.

## Contributing to This Repository

1. Clone the repository: `git clone <repository-url>`
2. Create a branch for your change: `git checkout -b update/<short-description>`
3. Commit your changes with a clear message and push the branch.
4. Open a pull request so other members (and the adviser, if added as a reviewer) can review before merging to `main`.

---
*This repository is maintained solely for academic purposes in fulfillment of the requirements of Thesis 1 and Thesis 2, BS Computer Science, DMMMSU South La Union Campus.*
