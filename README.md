<div align="center">

<img src="https://github.com/user-attachments/assets/284380e8-ac2d-4d22-9f67-56ab45ea6cdf" alt="Nexus-H3 - Urban Mobility Observatory" width="520"/>

# NEXUS-H3

**A Resource-Frugal Urban Mobility Observatory for Continuous Telemetry Preservation and Reproducible Benchmarking**

*Continuously preserving Grand Paris' ephemeral mobility telemetry as an open, reproducible spatiotemporal benchmark - for less than EUR 5 a month.*

[![Cadence](https://img.shields.io/badge/ingestion-3%20min%20cadence-ff69b4)](#the-benchmark) [![Spatial Index](https://img.shields.io/badge/spatial%20index-Uber%20H3%20Res--9-blue)](#the-benchmark) [![Cells](https://img.shields.io/badge/benchmark%20cells-8277-9cf)](#the-benchmark) [![Features](https://img.shields.io/badge/schema-up%20to%2053%20features-success)](#complete-v30-schema) [![Cost](https://img.shields.io/badge/operating%20cost-%3CEUR%205%2Fmonth-brightgreen)](#total-cost-of-ownership) [![License: MIT](https://img.shields.io/badge/code%20license-MIT-yellow)](#privacy--licensing) [![Data: ODbL](https://img.shields.io/badge/data%20license-ODbL%201.0-orange)](#privacy--licensing) [![Workbench](https://img.shields.io/badge/live%20workbench-nexus--h3.vercel.app-8A2BE2)](https://nexus-h3.vercel.app)

</div>

---

> **TL;DR** - Real-time municipal mobility feeds are ephemeral: Île-de-France Mobilités (IDFM) SIRI Lite departures and Smovengo Vélib' dock states are broadcast live and not maintained as public historical archives. **Nexus-H3** continuously polls, compacts with `polarduck`, and preserves these streams with Open-Meteo weather and RTE éCO2mix grid intensity onto an Uber H3 Resolution-9 grid across Grand Paris. The v3.0 Hybrid Star-Schema has up to 53 columns: 6 dual-CRS keys, 23 Layer 1 telemetry/traffic/CO2 features, and 24 Layer 0 static infrastructure and OSM morphology features. It runs on a 1 GB ARM64 VM, uses a 10 GB NVMe read-through cache, and sends 100% of analytical compilation to the browser, Google Colab, or the native `nexus-fetch` Rust CLI.

---

## Why Nexus-H3?

Modern metropolitan mobility research has three structural barriers:

| Structural Barrier | Reality | Nexus-H3 Solution |
|:---|:---|:---|
| **Ephemeral APIs** | IDFM and Vélib' are live operational feeds without public historical archives. | Continuous 3-minute polling with atomic PostgreSQL leases, SHA-256 lineage, NVMe dead-letter staging, and ZSTD compaction preserves observational windows. |
| **Static, single-modality benchmarks** | METR-LA, PeMS-BAY, NYC Taxi, and CitiBike lack multimodal transit delays, trajectories, grid-coupled CO2, 3D morphology, and infrastructure context. | A living 53-column Hybrid Star-Schema combines Eulerian scalars, mode-stratified delays, Dremel-shredded Lagrangian `raw_visits`, COPERT/VSP/RTE CO2, and 24 static layers. |
| **Cloud warehouse cost and compute** | Full denormalized 3-minute joins exceed about 1 TB/year and incur warehouse scan costs. | Normalized ZSTD slices are compactly stored and served through a 10 GB cache. Server compile routes are retired; analytical DuckDB joins run locally through `nexus-fetch`, Colab, or DuckDB-Wasm. |

---

## The Benchmark

<div align="center">
<img src="https://github.com/user-attachments/assets/6440f05f-a6b1-45f9-96ae-18583e65e3a0" alt="Grand Paris multi-modal benchmark on the H3 hexagonal grid" width="75%"/>
</div>

| Dimension | Specification |
|:---|:---|
| Spatial tessellation | Uber H3 Resolution-9, approximately 0.105 km2 per cell |
| Grand Paris core | 8,277 contiguous H3 cells across Departments 75, 92, 93, and 94 |
| Spatial composition | 7,670 building-footprint cells plus 607 park, river, and transit-corridor cells |
| Active Vélib' cells | 1,183: 814 Paris Intra-Muros and 369 suburbs |
| Compiled v3.0 schema | 53 columns: 6 keys, 23 Layer 1 dynamic/physics features, 24 Layer 0 static features |
| Native polling cadence | 3 minutes |
| Annual hybrid footprint | Approximately 165-185 GB/year with nested `raw_visits`, versus approximately 1 TB/year denormalized |
| Coordinate systems | WGS84 EPSG:4326 and Lambert-93 EPSG:2154 |
| Temporal cutover | `2026-10-17T00:00Z`; historical and live regimes remain disjoint |

During ingestion, Nexus-H3 captures approximately 9,800-12,000 regional Île-de-France H3 cells per 3-minute window to avoid boundary truncation. During compilation, local DuckDB spatially joins the data to the canonical 8,277-cell Grand Paris core.

### Complete v3.0 Schema

The v3.0 feature family is:

- **6 spatial/temporal keys:** `window_start`, `h3_index_r9`, WGS84 centroids, and Lambert-93 projected coordinates.
- **7 Layer 1 core facts:** Vélib' mechanical bikes, electric bikes, available docks, IDFM active departures, mean delay, temperature, and precipitation.
- **9 Tier 1 scalar features:** Additional mode-stratified and traffic context fields.
- **1 Tier 2 visit feature:** Dremel-shredded Lagrangian `raw_visits`.
- **6 Tier 3 physics features:** COPERT/VSP/RTE grid-coupled CO2e and related fields.
- **6 Layer 0 infrastructure features:** Transit stops, transit modal diversity, cellular antennas, 5G antennas, telecom operators, and monitored road arcs.
- **18 Layer 0 OSM 3D morphology features:** Building counts/footprints/heights/levels/type, land use and water areas, road/cycleway/rail/utility network lengths, and POI categories.

The live React page exposes all 53 columns with type, units, description, and zero-fill information through a searchable, filterable data dictionary.

---

## Production System Architecture

<div align="center">
<img src="https://github.com/user-attachments/assets/a91a9513-8383-480f-a20f-1a3d0bf9a370" alt="Nexus-H3 decoupled architecture" width="85%"/>
</div>

Nexus-H3 separates ingestion, coordination, caching, object storage, and user-side compilation into tightly bounded subsystems:

| Tier | Technology | Role |
|:---|:---|:---|
| **Edge ingestion** | Azure ARM64 VM, 1 GB RAM; 45-second IDFM prefetch; bounded streaming | Live Vélib', IDFM, weather, traffic, and historical windows |
| **Rust data plane** | [`polarduck`](https://github.com/Nexus-H3/PolarsDuck) v0.3.1 | Arrow/NDJSON streaming, native H3 indexing, schema-evolving ZSTD compaction, inline SHA-256, and USTAR streaming |
| **Control plane** | FastAPI, Neon PostgreSQL, Aiven Kafka | Lease state, lineage, catalog, backfill orchestration, session tickets, and audit downloads |
| **Object store** | Google Drive API v3 with ZSTD Parquet | Normalized partition storage with no server warehouse |
| **Read-through cache** | 10 GB NVMe `SliceCacheService` + Local DLQ | Cached slice delivery and resilient local staging |
| **Edge gateway** | Nginx HTTP/2 TLS reverse proxy | HMAC worker authentication, replay protection, and request limits |
| **Browser delivery** | DuckDB-Wasm | 5-row catalog preview and browser-side joining |
| **Native delivery** | [`nexus-fetch`](https://github.com/Nexus-H3/nexus-fetch) v0.1.2 | Cross-platform local streaming, embedded DuckDB compilation, static caching, and export packaging |

Server-side dataset compilation endpoints have been retired with `410 Gone`. Zero-cloud-compute exports run in the user's browser, Google Colab, or native workstation.

### Rust Data Plane and Compaction

Streaming extraction keeps traced heap use at **0.77 MB**, a **535.7x** reduction versus naive 411.12 MB AST deserialization. The 45-second prefetch and streaming ijson path reduce the critical ingestion path from 212 seconds to 3.3 seconds.

`polarduck` compacts 3-minute slices into key-sorted ZSTD blocks with bounded 8,192-row streaming passes. The 470-window validation set shrank from 287.46 MB of unitary Snappy files to 170.02 MB ZSTD, a 40.9% reduction.

### Native Local Compilation

Install the native CLI:

```bash
pip install -U nexus-fetch
nexus-fetch --job <JOB_UUID> --key <ONE_TIME_SECRET> --api https://<your-api-host>
```

Pre-built wheels are available for Windows, macOS, Linux x86_64, and Linux AArch64. The CLI supports parallel HTTP/1.1 downloads, optional HTTP/2, sub-day LZ4 intermediates, persistent static caching, local DuckDB compilation, ZSTD Parquet output, and stored-archive ZIP packaging.

### Lease, Cache, and Temporal State

- PostgreSQL atomically governs `PENDING`, `COMPLETED`, `EMPTY`, and `ORPHANED` 3-minute leases.
- Every committed slice is recorded with a SHA-256 digest and window bounds.
- Export tickets are single-use, scoped, and expire after five minutes.
- Mode B historical backfill and Mode A live telemetry are strictly separated at `2026-10-17T00:00Z`.
- Historical Vélib' and IDFM deviation fields remain SQL `NULL`; they are never synthetically imputed.

---

## Systems and Cost Results

### Systems Benchmarks

| Dimension | Result |
|:---|:---|
| SIRI Lite extraction | 411.12 MB naive heap -> 0.77 MB streaming heap, 535.7x lower |
| Critical ingestion path | 212.0 seconds -> 3.3 seconds, 64x faster |
| ZSTD compaction | 287.46 MB -> 170.02 MB, 40.9% smaller |
| Daily 24-slice commit | 96.0 seconds -> 13.0 seconds, 7.4x faster |
| Native 8-day scalar export | 3,128.91 MiB over 12 parallel HTTP/1.1 streams; 21.06 seconds local compile/merge |

### Total Cost of Ownership

Modeled monthly cost for a 1 TB cumulative workload and 10 TB of monthly analytical scan volume:

| Architecture | Estimated Monthly Cost |
|:---|---:|
| Nexus-H3: 1 GB Azure ARM64 + Neon + Google Drive + local/Colab/Wasm compilation | **< EUR 5.00** |
| GCP BigQuery | approximately EUR 95.00 |
| AWS S3 + Snowflake | approximately EUR 140.00 |

---

## Project Structure

```text
Nexus-H3/
├── .github/             # Organization profile and live landing/about documentation
├── mdl-web-view/        # React + Vite/TypeScript web workbench, admin console, and DuckDB-Wasm browser compiler
├── mdl-contron-plane/   # FastAPI control plane: leases, lineage, catalog, cache, Kafka and Drive orchestration
├── mdl-etl-engine/      # Live/backfill ingestion: ijson streaming, H3 aggregation, topology, and ZSTD compaction
├── nexus-fetch/         # Rust/Python native CLI: streaming downloads and local DuckDB compilation
├── PolarsDuck/          # Rust/Python acceleration: NDJSON streaming, H3 indexing, Parquet compaction, and SHA-256
└── nginx/               # Nginx reverse proxy and edge request policy
```

Primary repositories:

- [Web portal](https://github.com/Nexus-H3/mdl-web-view) - React/TypeScript workbench and about page
- [Control plane](https://github.com/Nexus-H3/mdl-contron-plane) - FastAPI and PostgreSQL orchestration
- [Ingestion engine](https://github.com/Nexus-H3/mdl-etl-engine) - live and backfill workers
- [nexus-fetch](https://github.com/Nexus-H3/nexus-fetch) - native Rust dataset compiler
- [PolarsDuck](https://github.com/Nexus-H3/PolarsDuck) - Rust data-plane acceleration
- [Nginx configuration](https://github.com/Nexus-H3/nginx) - public reverse-proxy layer

---

## Getting the Data

1. Open the [interactive compilation workbench](https://nexus-h3.vercel.app).
2. Select temporal regime, cadence, Grand Paris spatial scope, and join topology.
3. Preview the schema and a 5-row sample in the browser.
4. Download with `nexus-fetch`, Google Colab, or the browser route, according to the live workbench options.
5. Preserve the API version, interval, timezone, partition IDs, schema version, row counts, and SHA-256 checksums for reproducible research.

---

## Privacy, Ethics and Open-Science Licensing

- Nexus-H3 ingests macroscopic infrastructure and mobility states only. It does not access rider trajectories, subscriber identifiers, card validations, MAC addresses, or other PII.
- H3 Resolution-9 aggregation provides neighborhood-scale spatial context.
- **Source code and SQL:** MIT License.
- **Compiled benchmark and derived OSM tables:** ODbL 1.0.
- Upstream sources retain their applicable French open-license, CC BY, and OSM sharing terms.

---

## Citation

```bibtex
@misc{islam2026nexush3,
  title       = {Nexus-H3: A Resource-Frugal Urban Mobility Observatory for
                 Continuous Telemetry Preservation and Reproducible Benchmarking},
  author      = {Islam, Saiful},
  year        = {2026},
  month       = {oct},
  publisher   = {Zenodo},
  institution = {Telecom SudParis, Institut Polytechnique de Paris},
  doi         = {10.5281/zenodo.23749898},
  url         = {https://doi.org/10.5281/zenodo.23749898}
}
```

## Acknowledgements

Nexus-H3 builds on public feeds maintained by Île-de-France Mobilités (IDFM), Smovengo/Vélib' Métropole, Paris Open Data, Open-Meteo, RTE éCO2mix, ANFR, and OpenStreetMap contributors.

<div align="center">

**NEXUS-H3 - Urban Mobility Observatory**

*Preserving the ephemeral, one 3-minute window at a time.*

</div>
