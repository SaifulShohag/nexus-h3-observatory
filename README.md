<div align="center">

<img src="https://github.com/user-attachments/assets/284380e8-ac2d-4d22-9f67-56ab45ea6cdf" alt="Nexus-H3 — Urban Mobility Observatory" width="520"/>

# NEXUS-H3

**A Resource-Frugal Urban Mobility Observatory for Continuous Telemetry Preservation and Reproducible Benchmarking**

*Continuously preserving Grand Paris' ephemeral mobility telemetry as an open, reproducible spatiotemporal benchmark — for less than €5 a month.*

[![Cadence](https://img.shields.io/badge/ingestion-3%20min%20cadence-ff69b4)](#-the-benchmark) [![Spatial Index](https://img.shields.io/badge/spatial%20index-Uber%20H3%20Res--9-blue)](#-the-benchmark) [![Cells](https://img.shields.io/badge/benchmark%20cells-8%2C277-9cf)](#-the-benchmark) [![Features](https://img.shields.io/badge/schema-37%20features-success)](#-the-benchmark) [![Cost](https://img.shields.io/badge/operating%20cost-%3C%E2%82%AC5%2Fmonth-brightgreen)](#-cost) [![License: MIT](https://img.shields.io/badge/code%20license-MIT-yellow)](#-privacy--licensing) [![Data: ODbL](https://img.shields.io/badge/data%20license-ODbL%201.0-orange)](#-privacy--licensing) [![Workbench](https://img.shields.io/badge/live%20workbench-nexus--h3.vercel.app-8A2BE2)](https://nexus-h3.vercel.app)

</div>

---

> **TL;DR** — Real-time urban mobility APIs are *ephemeral*: Île-de-France Mobilités (IDFM) transit delays (~82 MB payload carrying ~19,500 active vehicle journeys / ~30,000 departure events every 3 minutes) and Vélib' station availability are broadcast live and never archived publicly. **Nexus-H3** is an open urban mobility observatory that continuously polls, harmonizes, and preserves these streams onto an Uber H3 Resolution-9 hexagonal grid across Grand Paris — publishing them as a reproducible spatiotemporal benchmark (**8,277 cells × 37 features**, 3-minute cadence, **~5.7M multi-modal observations/day**). By decoupling normalized dynamic telemetry from static contextual layers prior to on-demand Star-Schema compilation, Nexus-H3 bounds annual storage to **55–60 GB/year** (~2.08B raw records) instead of **>400 GB/year** of denormalized tables. It runs 24/7 on a single 1 GB RAM ARM64 VM (<€5/month), offloading heavy analytical joins to *your* browser (DuckDB-Wasm) and *your* free Google Colab or local runtime. No enterprise cloud warehouse. No egress fees. Zero PII.

---

## 📡 Why Nexus-H3?

Modern metropolitan mobility systems operate as tightly coupled, multi-modal networks where road traffic, public transit reliability, shared micromobility, and meteorological shocks interact at sub-kilometer and minute-level scales. Academic research groups studying these dynamics face three structural barriers:

| Structural Barrier | The Reality | Nexus-H3's Solution |
|:---|:---|:---|
| **1. The Ephemeral API Problem** | IDFM's SIRI Lite vehicle departures and Smovengo's Vélib' GBFS availability are **never distributed as public historical archives**. If a fault-tolerant pipeline is not actively polling, operational deviations cannot be reconstructed retrospectively. | A lease-coordinated observatory continuously captures, harmonizes, and archives live telemetry every 3 minutes. |
| **2. Benchmark Availability & Scale** | Established benchmarks (METR-LA, PeMS-BAY, NYC Taxi, CitiBike) are static snapshots or single-modality logs lacking multimodal transit, 3D urban morphology, weather, or telecom infrastructure context. | A **continuously growing** 37-column benchmark fusing 7 dynamic mobility/weather streams with 24 static urban morphology, transit, and 5G infrastructure features. |
| **3. The Cloud Warehouse & Egress Wall** | 24/7 city-wide ingestion at 3-minute intervals generates ~2.08 billion raw records per year; hosting and querying denormalized tables (>400 GB/yr) on commercial cloud warehouses incurs unsustainable compute and egress costs. | **Frugal disaggregation**: normalized ZSTD archival (55–60 GB/yr) on zero-egress storage + a <€5/month edge VM + client-side DuckDB compilation. |

---

## 🏙️ The Benchmark

<div align="center">
<img src="https://github.com/user-attachments/assets/e12d5069-4524-46e2-8fcd-16fdfca4a504" alt="Grand Paris multi-modal benchmark on the H3 hexagonal grid" width="100%"/>
</div>

<div align="center">

| Dimension / Indicator | Specification |
|:---|:---|
| Spatial tessellation | Uber H3 Resolution-9 (~0.105 km² per cell) |
| Grand Paris core extent ($N$) | **8,277** contiguous H3 cells (Depts. 75, 92, 93, 94) |
| Active micromobility cells ($N_{\text{active}}, \sigma > 0.5$) | **1,183** H3 cells (814 Paris Intra-Muros / 369 Suburbs) |
| Compiled Star-Schema | **37 columns** (6 spatiotemporal keys + 7 dynamic + 24 static) |
| Native polling cadence | **3 minutes** (480 nominal observation windows / day) |
| Daily ingestion throughput | **~5.7 million** raw multi-modal observations / day (~11k–12k regional H3 rows/window) |
| Annual storage footprint (1 year) | **~55–60 GB/yr** normalized ZSTD (~2.08B raw records vs. >400 GB/yr denormalized) |
| Coordinate reference systems | Dual CRS: Geodetic **WGS84** (`EPSG:4326`) + Metric **Lambert-93** (`EPSG:2154`) |
| Temporal regime cutover | Strict **Mode B** (Historical) / **Mode A** (Live) separation at `2026-10-17T00:00:00Z` |

</div>

> ℹ️ *Note on spatial extent:* During ingestion, Nexus-H3 captures ~11,000–12,000 regional Île-de-France H3 cells per window to prevent boundary truncation; during compilation, DuckDB spatially crops records to the canonical **8,277-cell** Grand Paris core (of which 7,678 cells carry non-empty OSM building/landuse polygons).

* **Spatiotemporal Keys & Dual CRS (6 columns):** `window_start`, `h3_index_r9`, geodetic centroids (`cell_lat`, `cell_lon`), and planar metric coordinates (`x_l93`, `y_l93`).
* **Layer 1 — Dynamic Mobility & Meteorological Telemetry (7 columns):** `velib_mechanical_bikes`, `velib_ebikes`, `velib_available_docks`, `idfm_active_departures`, `idfm_mean_delay_sec`, `temperature_2m`, and `precipitation`.
* **Layer 0 — Static Infrastructure & 3D Urban Morphology (24 columns):**
  * *Critical Infrastructure (6 columns):* IDFM GTFS physical stop density and modal diversity (**37,954** regional stops across **4,854** core cells), ANFR cellular and 5G NR antennas and operator diversity (**7,447** antennas across **3,150** core cells), and permanent inductive road-loop counting arcs (**754** dense core cells).
  * *3D Urban Morphology (18 columns):* OpenStreetMap building counts, Lambert-93 footprint areas (m²), mean heights and floor levels, dominant building and land-use typologies, green/water/commercial areas, road/cycleway/rail/utility network lengths, and categorized POI densities (**285,051** POIs across **7,365** core cells).

<details>
<summary><b>📋 Full 37-Feature Schema & Data Dictionary (click to expand)</b></summary>

| # | Column Name | Layer / Source | Type | Physical Unit / Description | Zero-Fill Fallback |
|---:|:---|:---|:---|:---|:---:|
| 1 | `window_start` | Key (Temporal) | `TIMESTAMP` | UTC observation window start timestamp (3m, 15m, or 1h) | Required |
| 2 | `h3_index_r9` | Key (Spatial) | `VARCHAR` | Uber H3 Resolution-9 hexadecimal cell index (~0.105 km²) | Required |
| 3 | `cell_lat` | Key (CRS: WGS84) | `DOUBLE` | H3 cell centroid geodetic latitude (`EPSG:4326`, degrees) | Required |
| 4 | `cell_lon` | Key (CRS: WGS84) | `DOUBLE` | H3 cell centroid geodetic longitude (`EPSG:4326`, degrees) | Required |
| 5 | `x_l93` | Key (CRS: L93) | `DOUBLE` | Metric easting in RGF93 / Lambert-93 (`EPSG:2154`, meters) | Required |
| 6 | `y_l93` | Key (CRS: L93) | `DOUBLE` | Metric northing in RGF93 / Lambert-93 (`EPSG:2154`, meters) | Required |
| 7 | `velib_mechanical_bikes` | Layer 1 (Vélib' GBFS) | `BIGINT` | Available mechanical bicycles in cell (`NULL` in Mode B) | `0` |
| 8 | `velib_ebikes` | Layer 1 (Vélib' GBFS) | `BIGINT` | Available electric bicycles in cell (`NULL` in Mode B) | `0` |
| 9 | `velib_available_docks` | Layer 1 (Vélib' GBFS) | `BIGINT` | Available empty return docks in cell (`NULL` in Mode B) | `0` |
| 10 | `idfm_active_departures` | Layer 1 (IDFM SIRI) | `BIGINT` | Active transit vehicle departures in window (`NULL` in Mode B) | `0` |
| 11 | `idfm_mean_delay_sec` | Layer 1 (IDFM SIRI) | `DOUBLE` | Mean departure delay across active calls in seconds (`NULL` in Mode B) | `0.0` |
| 12 | `temperature_2m` | Layer 1 (Open-Meteo) | `DOUBLE` | Ambient air temperature at 2 meters above ground (°C) | `15.0` |
| 13 | `precipitation` | Layer 1 (Open-Meteo) | `DOUBLE` | Total liquid-equivalent precipitation during window (mm) | `0.0` |
| 14 | `transit_stop_count` | Layer 0 (IDFM GTFS) | `BIGINT` | Physical rail, metro, tram, and bus stop points within H3 cell | `0` |
| 15 | `distinct_transit_modes` | Layer 0 (IDFM GTFS) | `BIGINT` | Distinct transit modes serving cell (Metro, RER, Tram, Bus) | `0` |
| 16 | `cellular_antenna_count` | Layer 0 (ANFR) | `BIGINT` | Total active commercial cellular radio antennas within cell | `0` |
| 17 | `antenna_5g_count` | Layer 0 (ANFR) | `BIGINT` | Active 5G NR cellular antennas (3.5 GHz / 2.1 GHz / 700 MHz) | `0` |
| 18 | `distinct_telecom_operators` | Layer 0 (ANFR) | `BIGINT` | Distinct mobile network operators present in cell (max 4) | `0` |
| 19 | `monitored_road_arc_count` | Layer 0 (Paris Data) | `BIGINT` | Permanent inductive-loop road traffic counting arcs within cell | `0` |
| 20 | `building_count` | Layer 0 (OSM 3D) | `BIGINT` | Total building polygons whose centroid intersects the H3 cell | `0` |
| 21 | `total_building_footprint_sqm` | Layer 0 (OSM 3D) | `DOUBLE` | Cumulative ground footprint area of buildings in cell (m², L93) | `0.0` |
| 22 | `avg_building_height_m` | Layer 0 (OSM 3D) | `DOUBLE` | Mean building height across structures in cell (m) | `0.0` |
| 23 | `avg_building_levels` | Layer 0 (OSM 3D) | `DOUBLE` | Mean above-ground floor count across structures in cell | `0.0` |
| 24 | `dominant_building_type` | Layer 0 (OSM 3D) | `VARCHAR` | Modal building typology tag (e.g., `apartments`, `commercial`) | `'none'` |
| 25 | `green_and_field_area_sqm` | Layer 0 (OSM Landuse) | `DOUBLE` | Total park, forest, grass, and vegetated area in cell (m²) | `0.0` |
| 26 | `commercial_industrial_area_sqm` | Layer 0 (OSM Landuse) | `DOUBLE` | Total commercial, retail, and industrial landuse area in cell (m²) | `0.0` |
| 27 | `water_area_sqm` | Layer 0 (OSM Landuse) | `DOUBLE` | Total river, canal, and basin surface water area in cell (m²) | `0.0` |
| 28 | `dominant_landuse` | Layer 0 (OSM Landuse) | `VARCHAR` | Primary landuse classification by area (e.g., `residential`, `mixed`) | `'mixed'` |
| 29 | `total_road_length_m` | Layer 0 (OSM Network) | `DOUBLE` | Total vehicular highway and street centerline length in cell (m) | `0.0` |
| 30 | `cycleway_length_m` | Layer 0 (OSM Network) | `DOUBLE` | Total dedicated cycleway and bike-lane segment length in cell (m) | `0.0` |
| 31 | `rail_transit_length_m` | Layer 0 (OSM Network) | `DOUBLE` | Total heavy rail, metro, and tramway track length in cell (m) | `0.0` |
| 32 | `pipeline_and_power_length_m` | Layer 0 (OSM Network) | `DOUBLE` | Total high-voltage power line and utility corridor length in cell (m) | `0.0` |
| 33 | `total_poi_count` | Layer 0 (OSM Amenity) | `BIGINT` | Total Points of Interest (amenities, shops, offices, tourism) | `0` |
| 34 | `market_and_retail_count` | Layer 0 (OSM Amenity) | `BIGINT` | Supermarkets, marketplaces, department stores, and retail shops | `0` |
| 35 | `food_and_beverage_count` | Layer 0 (OSM Amenity) | `BIGINT` | Restaurants, cafés, bakeries, and bars within cell | `0` |
| 36 | `education_and_health_count` | Layer 0 (OSM Amenity) | `BIGINT` | Schools, universities, hospitals, clinics, and pharmacies in cell | `0` |
| 37 | `utility_and_infrastructure_nodes` | Layer 0 (OSM Amenity) | `BIGINT` | Power substations, telecom towers, water works, municipal nodes | `0` |

</details>

---

## 🏗️ System Architecture — Frugal Disaggregation

<div align="center">
<img src="https://github.com/user-attachments/assets/80b3cc4e-9e37-4e0c-a38e-c9d06b7c3c35" alt="Nexus-H3 resource-frugal disaggregated architecture" width="70%"/>
</div>

To sustain a continuous urban observatory without enterprise cloud funding, Nexus-H3 decouples coordination state, object storage, continuous ingestion, and analytical compilation across strictly bounded execution tiers:

| Execution Tier | Technology Stack | Resource Envelope / Role |
|:---|:---|:---|
| **Edge Ingestion Worker** | Single ARM64 VM (Azure `Standard_B2pts_v2`, 2 vCPU) polling IDFM, Vélib', Open-Meteo | **1.0 GB RAM** + 64 GB NVMe |
| **Streaming Transformation** | `ijson` (`yajl2_c`) + `Polars` + `DuckDB Spatial` (H3 Res-9 & WGS84/L93 projection) | Bounded sub-GB memory envelope |
| **Control Plane & Lineage** | FastAPI + Serverless Neon PostgreSQL 16 (`batch_leases`, `parquet_lineage`) | Atomic upsert locks + SHA-256 catalog |
| **Zero-Egress Object Store** | Google Drive API v3 (OAuth), ZSTD Parquet, 24-slice daily compaction | Zero egress fees (55–60 GB/year) |
| **Edge Security Gateway** | Nginx reverse proxy + `HMAC-SHA256` worker signing + replay protection | 5 req/s/IP, 20 max conns, 5 MB cap |
| **Compiler — Tier 1 (Browser)** | `DuckDB-Wasm` in the researcher's browser via HTTP byte-range streaming | Zero server compute; 5-row live preview |
| **Compiler — Tier 2 (Server)** | Co-located FastAPI + DuckDB server compiler with out-of-core disk spill | `max_memory = '512MB'`, `threads = 2` |
| **Compiler — Tier 3 (Offload)** | Deterministic Python/DuckDB script synthesis on free-tier Google Colab | **10.0 GB RAM**; burn-on-claim session keys |

### 1. Constant-Memory Streaming Extraction
The IDFM SIRI Lite API broadcasts regional vehicle departures as an **82.11 MB** decompressed JSON payload carrying **~19,500 active vehicle journeys** (~30,000 departure events) every 3 minutes. Standard `json.load()` deserialization constructs the full AST in memory, generating **411.12 MB** of traced Python heap allocations (~575 MB including raw byte and UTF-8 string buffers) and triggering OOM crashes on a 1 GB host. Nexus-H3 initiates an asynchronous prefetch 45 seconds before each window boundary and streams records from disk via C-backed `ijson` into bounded `Polars` frames — reducing traced Python heap allocations to **0.77 MB (535.7× lower)** and cutting critical-path latency from **212.0 s to 3.3 s (64× faster)**.

### 2. Duplicate-Safe Lease State Machine & Cryptographic Lineage
Every 3-minute observational window is governed by an atomic PostgreSQL upsert (`INSERT ... ON CONFLICT DO UPDATE` on `batch_leases`) with four persisted states: `PENDING`, `COMPLETED`, `EMPTY`, and `ORPHANED`. A granted lease remains `PENDING`, distinguished only by its `worker_id` and 3600-second TTL expiry — enforcing liveness by time rather than a persisted in-progress lock. Expired grants are swept to `ORPHANED` and atomically re-claimed, while `COMPLETED` is terminal, guaranteeing that worker restarts or Kafka rebalances never produce duplicate Parquet row groups. Every committed slice is recorded in `parquet_lineage` with its `sha256_hash`, `row_count`, `file_size_bytes`, and `[window_start, window_end]` bounds.

### 3. Progressive Compute Escalation & Star-Schema Compilation
Instead of pre-materializing >400 GB/year of denormalized tables in a cloud warehouse, Nexus-H3 compiles datasets on demand across three join topologies:
* **`telemetry_only` (Telemetry Only):** Exports dynamic Layer 1 observations only for lightweight time-series analysis and operational auditing.
* **`custom_static` (Custom Static Join):** Attaches the 24 Layer 0 static attributes strictly to H3 cells exhibiting active mobility updates within each window.
* **`complete_h3_grid` (Complete H3 Grid):** Executes a full Cartesian spatial-temporal cross-join ($\mathcal{T} \times N, N=8,277$) before left-joining dynamic facts and static layers, producing the fixed graph topology ($\mathbf{A} \in \mathbb{R}^{N \times N}$) commonly required by graph-based spatiotemporal workflows.

### 4. Strict Temporal Regime Separation
Continuously operating observatories accumulate data under evolving sensing regimes. Nexus-H3 enforces a strict non-overlapping separation ($\mathcal{D}_{\text{historical}} \cap \mathcal{D}_{\text{live}} = \emptyset$) at `2026-10-17T00:00:00Z`:
* **Mode B (Historical Backfill, pre-cutover):** Rate-limit-aware 6-hour grouped backfill of road-loop and meteorological archives; ephemeral transit delays and bike availability are structurally `NULL` and **never synthetically imputed**.
* **Mode A (Live Multi-Modal Telemetry, post-cutover):** Continuous 3-minute multi-modal observations across all 37 columns.

---

## 📊 Comparison With Representative Benchmarks

| Dataset | Continuous Updates | Multi-Modal | Weather Fused | 3D Morphology | Infrastructure | Spatial Indexing | Public Access |
|:---|:---:|:---:|:---:|:---:|:---:|:---|:---:|
| **METR-LA** | ✗ (Static Archive) | ✗ (Traffic Only) | ✗ | ✗ | ✗ | Point / Graph | ✓ |
| **PeMS-BAY** | ✗ (Static Archive) | ✗ (Traffic Only) | ✗ | ✗ | ✗ | Point / Graph | ✓ |
| **NYC Taxi** | ✗ (Monthly Batches) | ✗ (Taxis Only) | ✗ | ✗ | ✗ | Custom Taxi Zones | ✓ |
| **CitiBike (NYC)** | ✗ (Monthly Batches) | ✗ (Bikes Only) | ✗ | ✗ | ✗ | Lat/Lon Points | ✓ |
| **Nexus-H3 (Ours)** | **✓ (3-min Telemetry)** | **✓ (Transit, Bikes, Traffic)** | **✓ (Open-Meteo)** | **✓ (OSM Buildings)** | **✓ (Stops, 5G)** | **Uber H3 (Res-9)** | **✓ (Zero-Egress)** |

---

## 📈 Systems Evaluation & Dataset Utility Validation

### 1. Systems Benchmarks

<div align="center">

| Systems Dimension | Benchmark Result |
|:---|:---|
| **SIRI Lite Deserialization Heap** (82.11 MB, 19,500 journeys) | **535.7× reduction** in traced AST heap (`411.12 MB` → `0.77 MB`) |
| **Critical-Path Ingestion Latency** (Prefetch + `ijson` stream) | **64× speedup** (`212.0 s` → `3.3 s` in-window commit) |
| **Key-Sorted ZSTD Compaction** (470 windows, 3,890,190 rows) | **40.9% storage reduction** (`287.46 MB` unitary Snappy → `170.02 MB` ZSTD, `1.69×`) |
| **24-Slice Daily Batch Commit** (`POST /leases/complete-batch`) | **7.4× faster** (`96.0 s/day` → `13.0 s/day`, 95.8% fewer Drive/DB transactions) |
| **Layer 0 Spatial Compilation** (240 MB OSM PBF → 8,277 cells) | Completes in **64 s** on Tier 3 (10 GB Colab + `Pyrosm`) vs. OOM on Tier 1/Tier 2 |
| **Initial 7-Day Live Deployment Uptime** (Oct 19–25, 2026) | **95.65%** (`3,214 / 3,360` windows captured; zero upstream OOM crashes) |

</div>

### 2. Total Cost of Ownership (1 TB Archive + 10 TB Monthly Scan Volume)

<div align="center">

| Architecture | Estimated Monthly Cost |
|:---|---:|
| **Nexus-H3 (Azure `Standard_B2pts_v2` Micro-VM + Disaggregated Free Tiers)** | **< €5.00** |
| GCP BigQuery (Storage + On-Demand Analytical Scan) | ~€95.00 |
| AWS S3 + Snowflake Compute Warehouse | ~€140.00 |

</div>

### 3. Dataset Utility Validation (v1.0 Weekday Snapshot: 470 Windows, 3,890,190 Rows)

*Note: These experiments serve as utility validations verifying schema consistency, chronological hygiene, and out-of-the-box usability across tabular, temporal, and graph-conditioned baselines (`h = 30 min` / 10 steps ahead across $N_{\text{active}} = 1,183$ cells, chronological 70/15/15 split, `SEED = 42`).*

<div align="center">

| Baseline Architecture | MAE ↓ | RMSE ↓ | MAPE (%) ↓ |
|:---|---:|---:|---:|
| Historical Average (HA) | 6.607 | 9.629 | 91.36% |
| Persistence (Last Observation Carried Forward, LOCF) | 2.505 | 5.776 | 22.53% |
| LightGBM (Dynamic $\mathbf{X}_t$ Only) | 2.151 | 4.487 | **20.78%** |
| Temporal Residual Baseline (No Graph) | 2.153 | 4.549 | 20.96% |
| LightGBM ($+\text{Static } \mathbf{S} + \text{H3 } 1\text{-Ring}$) | 2.147 | **4.477** | 21.06% |
| **H3-GNN (H3 $1$-Ring Graph $\mathbf{A} + \text{Static } \mathbf{S}$)** | **2.144** | 4.541 | 20.91% |

</div>

* **30-Minute Forecasting:** All four learned baselines reduce MAE by up to **14.4%** and RMSE by **22.5%** relative to Persistence under a strictly chronological split.
* **Zero-Shot Spatial Transfer (Paris Intra-Muros $N_{\text{train}}=814 \rightarrow$ Suburbs $N_{\text{test}}=369$):** Models trained exclusively on Intra-Muros transfer effectively to unseen suburban hexagons, reducing suburban RMSE by **38.8%** (`1.967` vs. `3.212` LOCF), while quantile-normalized static morphology ($\mathbf{S}$) improves relative percentage accuracy (`MAPE = 15.68%` vs. `15.94%`).

---

## 🚀 Getting & Compiling the Data

You can inspect and compile custom Star-Schema slices directly via the **[Interactive Compilation Workbench](https://nexus-h3.vercel.app)** or download the archival v1.0 release from **[Zenodo (DOI: 10.5281/zenodo.23700284)](https://doi.org/10.5281/zenodo.23700284)**:

1. **Interactive Configuration:** Select the temporal regime (*Mode A Live Telemetry* or *Mode B Historical Backfill*), cadence (`3m`, `15m`, or `1h`), spatial scope (*Grand Paris* $N=8,277$ or *Intra-Muros*), and join topology (`telemetry_only`, `custom_static`, or `complete_h3_grid`).
2. **In-Browser Inspection (Tier 1):** `DuckDB-Wasm` streams Parquet footers into your browser via HTTP range requests for instant schema verification and 5-row live sampling.
3. **Zero-Cost Cloud Compilation (Tier 3):** Click **Download with Colab** to mint a burn-on-claim session (`job_id` + ephemeral `secret_key`) and copy the synthesized Python bootstrap script into a free Google Colab runtime (10 GB RAM). The script streams authorized partitions, verifies SHA-256 checksums against `parquet_lineage`, executes the Star-Schema join, and exports a key-sorted ZSTD Parquet file.
4. **Reproduce Paper Benchmarks:** Run `notebooks/section8_benchmarks.ipynb` on the compiled snapshot (`live_cd_3m_26.10.21_22-26.10.22_21_full_join.parquet`) to deterministically reproduce all systems, forecasting, and spatial transfer tables (`SEED = 42`).

---

## 🗺️ Future Roadmap (v2.0 Architecture)

<div align="center">
<img src="https://github.com/user-attachments/assets/75400d33-7c58-4fd7-bce3-61c0f418d150" alt="Nexus-H3 v2.0 Fault-Isolated & Self-Cached Roadmap Architecture" width="70%"/>
</div>

To evolve Nexus-H3 from a foundational Grand Paris observatory into a fault-isolated, multi-season benchmark suite, five engineering extensions are underway:

- [x] **Duplicate-Safe Lease State Machine & `EMPTY` Nocturnal States** — atomic PostgreSQL upsert locks (`PENDING`, `COMPLETED`, `EMPTY`, `ORPHANED`) with SHA-256 lineage tracking.
- [ ] **Fault-Isolated 10 GB Local NVMe Spooling & Async Kafka Replay** — decouples *event-time capture* from *processing-time materialization* via a local write-ahead ring buffer (~66 hours of offline capacity at ~7.5 MB/window), ensuring near-100% telemetry capture across downstream OAuth or database maintenance windows.
- [ ] **Granular Failure-Reason Telemetry & Automated SLA Auditing** — extends `batch_leases` with structured diagnostic sub-codes distinguishing nocturnal transit pauses from upstream HTTP `5xx` outages, `429` quota exhaustion, and local OAuth expirations.
- [ ] **Standalone Cross-Platform Rust Compiler Executables & 5 GB Server Slice Cache** — zero-dependency native binaries (`.exe`, macOS, Linux) that execute Star-Schema joins directly on local workstations without Google Colab, paired with a **5 GB server-side NVMe read-through Parquet cache (24h TTL + LRU eviction)** to eliminate Google Drive `403`/`429` throttling across overlapping compilation jobs.
- [ ] **Automated Rolling Live Compaction** — extends the 24-slice compaction daemon to automatically roll up aging 3-minute Mode A live partitions into daily key-sorted ZSTD archives after a 48-hour grace period.
- [ ] **Longitudinal Multi-Season Cross-Regime Benchmark** — multi-season release evaluating deep domain adaptation and cross-regime transfer learning between the 12-month Mode B archive and continuous Mode A streams across autumn, winter, and spring commuting cycles.

---

## 📁 Repository Layout

```text
nexus-h3-observatory/
├── control-plane/       # FastAPI: lease state machine, SHA-256 lineage catalog, Tier 2/3 compiler broker
├── ingestion-engine/    # Polars/DuckDB workers: streaming ijson live 3-min loop, Kafka backfill, Layer 0 GIS
├── web-portal/          # React 18 + TypeScript + DuckDB-Wasm: interactive compilation workbench & admin console
├── notebooks/           # Reproducible systems, forecasting & spatial transfer benchmarks (SEED = 42)
├── reverse-proxy/       # Nginx edge gateway configuration (TLS termination, IP rate limits, payload caps)
└── LICENSE              # MIT License (software & SQL) + ODbL 1.0 notice (compiled data & OSM tables)
```

---

## 🔒 Privacy, Ethics & Open-Science Licensing

* **Privacy by Spatial Aggregation (Zero PII / GDPR Compliant):** Nexus-H3 ingests exclusively macroscopic infrastructure states and aggregate station/stop telemetry. No individual rider trajectories, smart-card validations, MAC addresses, or mobile subscriber identifiers are ever accessed or stored. Projecting observations onto Uber H3 Resolution-9 hexagons (~0.105 km² per cell) provides neighborhood-scale aggregation that precludes individual re-identification entirely.
* **Dual Open Licensing:**
  * **Source Code, Control Plane & SQL (`MIT License`):** All software and compilation workflows in this repository are released under the permissive [MIT License](LICENSE).
  * **Compiled Benchmark & Layer 0 Morphology Tables (`ODbL 1.0`):** Upstream mobility and infrastructure streams (IDFM, Smovengo, Paris Data, ANFR) are governed by the French **Licence Ouverte / Open Licence 2.0 (Etalab)**, meteorological series by **CC BY 4.0** (Open-Meteo), and building/network morphology by the **Open Database License (ODbL 1.0)** (OpenStreetMap). To uphold share-alike compliance, the compiled benchmark and derived Layer 0 tables are released under **ODbL 1.0**.

---

## 📜 Citation

If you use the Nexus-H3 observatory, compilation workbench, or benchmark snapshots in your research, please cite:

```bibtex
@misc{islam2026nexush3,
  title       = {Nexus-H3: A Resource-Frugal Urban Mobility Observatory for
                 Continuous Telemetry Preservation and Reproducible Benchmarking},
  author      = {Islam, Saiful},
  year        = {2026},
  institution = {Institut Polytechnique de Paris, Palaiseau, France},
  note        = {Preprint, October 2026},
  doi         = {10.5281/zenodo.23700284},
  url         = {https://github.com/SaifulShohag/nexus-h3-observatory}
}
```

## 🙏 Acknowledgements

Nexus-H3 is built upon public open-data feeds maintained by **Île-de-France Mobilités (IDFM)** (SIRI Lite & GTFS), **Smovengo / Vélib' Métropole** (GBFS), **Paris Open Data** (permanent road-loop counters), **Open-Meteo** (meteorological series), **ANFR** (commercial cellular antenna registries), and **OpenStreetMap** contributors.

<div align="center">

**NEXUS-H3 · Urban Mobility Observatory**

*Preserving the ephemeral, one 3-minute window at a time.*

</div>
