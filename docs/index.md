# GSoC 2025: Extension Support for pgexporter

**Project Title:** Extension Support for pgexporter  
**Contributor:** Bassam Adnan  
**Organization:** PostgreSQL  
**Mentors:** Jesper Pedersen (My-Wine-Cellar), Saurav Pal (resyfer)  
**Progress Report:** [https://github.com/pgexporter/pgexporter/discussions/221](https://github.com/pgexporter/pgexporter/discussions/221)  
**Proposal:** PostgreSQL extensions can offer detailed views of query performance, storage usage, and system health but this would need frequent manual setup to be used with monitoring systems. Our proposal improves upon pgexporter through automatic detection of supported extensions and the output of pre-configured metrics for vital extensions such as pg_stat_statements, pg_buffercache, and pg_stat_monitor. It allows easy observability, diminishes the role of manual setup, and helps for optimal data-driven database performance monitoring. Please find my detailed proposal [here](https://docs.google.com/document/d/1EPbXDyh524dscINv4Yv-B8Qo7lo7Cjkex9w6P38g7JE/edit?usp=sharing).

---

## Project Overview

pgexporter is a Prometheus exporter for PostgreSQL that provides database metrics. This project adds **extension support** to pgexporter, enabling automatic detection, version handling, and metric collection from popular PostgreSQL extensions. See **Discussion Forum:** [GSoC 2025 Extension Support #221](https://github.com/pgexporter/pgexporter/discussions/221) for an extensive discussion of how the project played out.

### The Problem

While pgexporter had extensive ammount of core queries and supported custom queries, it lacked extension support, this becomes important because the extension versioning may have different semantics (per extension itself that too). We wanted to add out-of-the box metric support for most popular Postgres extensions.

### What We Built

- **Extension Detection System**: Automatic discovery of installed extensions with version tracking
- **Semantic Versioning**: Unified comparison system supporting different versioning formats (e.g., `2.19.3` vs `1.10`)
- **Comprehensive Extension Library**: Metric files for 15+ popular PostgreSQL extensions
- **Configuration System**: Enable/disable extensions globally or per-server
- **Test Infrastructure**: HTTP-based test suite for validating metrics and server behavior

---

## Contributions

Here I discuss the contributions I have made whlie providing support for `pgexporter`.
### Extensions Added (with version support)

| Extension | Purpose | Versions Supported |
|-----------|---------|-------------------|
| **pg_stat_statements** | Query performance tracking | 1.8 - 1.11 |
| **TimescaleDB** | Time-series data | 2.10.0 - 2.20.0 |
| **PostGIS** (+ topology, raster) | Spatial/geographic objects | 3.0+ |
| **pg_buffercache** | Buffer cache statistics | 1.3+ |
| **vector** | Vector similarity search (AI/ML) | 0.8.0+ |
| **pgcrypto** | Cryptographic functions | 1.3+ |
| **pg_wait_sampling** | Wait event sampling | All versions |
| **Citus** | Distributed PostgreSQL | 10.2+ (10.2, 11.0, 12.0) |
| **pg_stat_kcache** | OS-level statistics | 2.1.3 - 2.3.1 |
| **pg_stat_monitor** | Enhanced query monitoring | 2.0 - 2.3 |
| **pgstattuple** | Tuple-level statistics | 1.5 (PG13-17) |

### Features added

**1. Semantic Versioning**
```c
// Supports comparing 2.19.3 with 2.19.2 AND 1.10 with 1.9, handles alphanumeric cases too. Atleast the ones we encountered :D
struct version {
    int major;
    int minor;
    int patch;
};
```

**2. Extension Detection on Startup**
Queries `pg_available_extensions` and stores metadata in AVL tree structures for version-based query selection.

**3. Extension Configuration Section**
```ini
[pgexporter]
extensions = pg_stat_statements, timescaledb  # Global enable

[primary]
extensions = citus  # Server-specific override
```

**4. View Evolution Tracking**
Created scripts to extract and diff extension views across PostgreSQL versions, documenting schema changes for compatibility. These are available [here](https://github.com/bassamadnan/pg_ext_tracker/). With verification scripts and ouputs apart from the view patches accross versions.

### Infrastructure Improvements

- **Duplicate Detection**: YAML/JSON validation using ART (Adaptive Radix Tree) to prevent metric name collisions
- **HTTP API Refactor**: Enhanced request/response handling with deque-based header lookups (ported to pgmoneta/pgagroal)
- **Test Suite**: Comprehensive test cases covering startup, HTTP endpoints, metric validation
- **Memory Leak Fixes**: Multiple rounds of valgrind-based leak detection and fixes
- **Documentation**: Updated Prometheus chapter, added `extra.info` for all projects

---

## Some Numbers

- **30+ PRs merged** across pgexporter, pgmoneta, and pgagroal
- **14 extensions** with comprehensive metric coverage (11 external + 3 PostGIS variants)
- **100+ metrics** added for extension monitoring alone, added metrics for PostgreSQL 18 as well.
- **PostgreSQL 13+** version support with ongoing compatibility work

---

## Learning Outcomes

Through this project, I had a great time developing my skills in C. Engaging with syscalls for the first time outside an academic project, I got to learn much more about how shared memory space works, transferring data via HTTP (by building my own HTTP suite!) and where exactly is encryption used in the network. Apart from that, PostgreSQL wire protocol was a very fun read. Abit of a nightmare to deal with was GitHub Actions, I still remember the time we had one failing test case only in FreeBSD systems while every other system seemed just okay.

Apart from that, one major thing which I will always hold on to from this project is engaging in constant discussions, reaching out to help and make good quality PRs so as to not waste the mentors time. To me, the most important thing i've learned out of this project is communication.

---

## Future Work

- **More Extensions**: pg_partman, pg_qualstats, hstore metrics
- **Monitoring Solutions**: Alternative to Grafana more fine tuned for pgexporter use cases.

---

## Acknowledgements

Huge thanks to my mentors **Jesper Pedersen** and **Saurav Pal** for their guidance, code reviews, and patience throughout this journey. Special thanks to **@Userfrom1995** and **@ashu3103** for engaging with me in discussions for test suite and the entire pgexporter community for their support.

---

## Links

- **Discussion Forum:** [GSoC 2025 Extension Support](https://github.com/pgexporter/pgexporter/discussions/221)
- **Project Repository:** [pgexporter/pgexporter](https://github.com/pgexporter/pgexporter)
- **Documentation:** [pgexporter.github.io](https://pgexporter.github.io/pgexporter/)
- **Releases:** Extension support integrated in pgexporter releases

---

*This project was completed as part of Google Summer of Code 2025 with pgexporter.*
