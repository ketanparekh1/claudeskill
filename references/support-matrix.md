# Qlik Replicate Support Matrix (condensed)

Source: "Qlik Replicate Support Matrix", May 2026. Full original PDF has
more cloud-service variants (each usually "same as the underlying
endpoint" — check the original PDF for anything not listed here).

## Supported platforms
- **Windows**: Server 2016, 2019, 2022, 2025 (64-bit)
- **Linux**: RHEL 8.x, 9.x, 10.x; Amazon Linux 2023

Rule: once a major version is certified, minor versions in that family are
supported too (e.g. Oracle 19.x covers 19.0, 19.3, 19.5...).

## Relational database sources
| Endpoint | Supported versions | Windows | Linux |
|---|---|---|---|
| IBM DB2 for LUW | 11.5, 12.1 | Yes | Yes |
| IBM DB2 for z/OS | DB2 12.1/13.1, z/OS 2.5/3.1/3.2 | Yes | Yes |
| IBM DB2 for iSeries | 7.4, 7.5 | Yes | Yes |
| IBM Informix | 12.1, 14.10 | Yes | No |
| Microsoft SQL Server | 2016, 2017, 2019, 2022, 2025 | Yes | Yes |
| Microsoft SQL Server (MS-CDC) | 2016, 2017, 2019, 2022, 2025 | Yes | Yes |
| MySQL | 8.4 | Yes | Yes |
| MariaDB | 10.6–10.11, 11.4, 11.8 | Yes | Yes |
| **Oracle** | **19.x, 21c, Oracle AI Database 26ai** (26ai: Standard/Enterprise editions only; compatibility parameter must match a supported version) | Yes | Yes |
| PostgreSQL | 14.x–18.x | Yes | Yes |
| SAP Sybase ASE | 16.0, 16.1 | Yes | Yes |
| SAP HANA | 2.0 | Yes | Yes |

## Relational database targets
| Endpoint | Supported versions | Windows | Linux |
|---|---|---|---|
| IBM DB2 for z/OS | DB2 12/12.1/13.1, z/OS 2.5/3.1/3.2 | Yes | Yes |
| IBM DB2 for LUW | 11.5, 12.1 | Yes | Yes |
| Microsoft SQL Server | 2016, 2017, 2019, 2022, 2025 | Yes | Yes |
| MySQL | 8.4 | Yes | Yes |
| MariaDB | 10.6–10.11, 11.4, 11.8 | Yes | Yes |
| **Oracle** | **19.x, 21c, Oracle AI Database 26ai** (26ai: Standard/Enterprise editions only) | Yes | Yes |
| PostgreSQL | 14.x–18.x | Yes | Yes |
| SAP Sybase ASE | 16.0, 16.1 | Yes | Yes |
| SAP HANA | 2.0 | Yes | Yes |

## Other source categories (see original PDF for full detail)
- **Cloud-based** (Amazon RDS/Aurora, Google Cloud SQL, Azure, Snowflake, etc.) — almost all say "same as the underlying endpoint," with occasional version carve-outs (e.g. Amazon RDS for Oracle excludes 26ai; Amazon RDS for SQL Server excludes 2025).
- **ARC-based**: IBM IMS (z/OS 2.3–3.2, IMS 13–15.x), IBM VSAM Batch
- **File / File Channel**: N/A (no version)
- **Data warehouses**: Teradata Database 17.x, 20.x
- **ODBC / ODBC with CDC**: 3.0, 3.5, 3.8 — only for sources not otherwise in this matrix, and only via Professional Services approval
- **NoSQL**: MongoDB (Standard and Atlas) 7.x, 8.x
- **SAP** (Application, DB, Extractor, ODP, OData) — version support follows the underlying backend database; SAP ODP needs ODP API 2.0 (specific SAP basis levels — see original PDF)

## Other target categories (see original PDF for full detail)
- Streaming: Kafka 3.x/4.0.x/4.1/4.2; Confluent Cloud, Azure Event Hubs, Kinesis, Pub/Sub
- Hadoop-based: Cloudera 7.x, Hortonworks 3.x+, HDP 3.x+, CDP Private Cloud 7.x
- Data warehouses: Teradata 17.x/20.x, SingleStore 8.9.x/9.0.x
- Cloud storage/warehouse: Snowflake, Redshift, S3, BigQuery, Databricks (multiple LTS versions), Microsoft Fabric, ADLS
- ODBC: 3.0, 3.5, 3.8 (same Professional-Services-only caveat as source)

## Bidirectional replication support
Only a subset of endpoints support bidirectional tasks. Sources: Amazon
RDS (MySQL/PostgreSQL/SQL Server), Aurora PostgreSQL, File Channel,
Google Cloud SQL variants, IBM DB2 (all 3 variants), Azure SQL (MS-CDC and
Managed Instance), SQL Server, SQL Server (MS-CDC), MySQL, **Oracle**,
PostgreSQL, SAP Sybase ASE. Targets: File Channel, Google Cloud SQL
variants, IBM DB2 for z/OS, SQL Server, Azure SQL Database, MySQL, ODBC,
**Oracle**, PostgreSQL, SAP Sybase ASE.

## Supported browsers (for the EM/Replicate console)
Microsoft Edge, Mozilla Firefox, Google Chrome — all with automatic
updates on. Multi-vertical-monitor console display is not supported.
