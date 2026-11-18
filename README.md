# Tenable Scan Field Reference

A data dictionary for the **.nessus XML** scan export format. Documents every extractable field with its XML source location, SQL data type, and description, so you can parse .nessus files and import them into a relational database with the correct schema.

**Companion repo:** [Tenable_Nessus_Scripts](https://github.com/NullSpace-BitCradle/Tenable_Nessus_Scripts) -- Python scripts that extract these fields into CSV, Excel, and SQL Server.

## Quick Start

1. Export scan results from Tenable.io or Nessus in `.nessus` format (XML)
2. Use the field tables below to build your database schema (or use the [CREATE TABLE example](#sql-server-create-table))
3. Parse `ReportHost` elements for host properties, then `ReportItem` elements for vulnerability data
4. Store raw `plugin_output` in a MAX/TEXT column -- it can be very large

## Field Reference

Fields are organized by where they appear in the .nessus XML structure. The **XML name** column is the tag, attribute or element name the companion scripts read. Tenable's [Nessus v2 file format document](https://static.tenable.com/documentation/nessus_v2_file_format.pdf) (2016) shows some of them (for example `HOST_START`, `HOST_END`, `operating-system`, `mac-address`, `netbios-name`, `synopsis`, `see_also` and `exploitability_ease`). Newer fields, such as the threat, CISA and Cisco ones, are not in it. No name here was checked against a real .nessus file. The bucket and value descriptions for the VPR fields come from Tenable's [VPR drivers documentation](https://developer.tenable.com/docs/vpr-drivers), and the exact text written into a .nessus file may differ from it. The **SQL Type** column matches the schema used by the companion [Tenable_Nessus_Scripts](https://github.com/NullSpace-BitCradle/Tenable_Nessus_Scripts) SQL import script.

### Host Properties

Extracted from `ReportHost > HostProperties > tag[@name]` elements. One set of values per scanned host.

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **host_ip** | `host-ip` | NVARCHAR(255) | Primary IPv4/IPv6 address of the scanned asset. |
| **mac_address** | `mac-address` | NVARCHAR(MAX) | MAC address(es) discovered for the host. Multiple addresses are comma-separated. |
| **virtual_mac_address** | `virtual-mac-address` | NVARCHAR(MAX) | MAC address of virtual adapter reported by virtualized hosts. |
| **hostname** | `hostname` | NVARCHAR(MAX) | Short hostname gathered from the host (SMB, SSH, etc.). See netbios_name for the NetBIOS name. |
| **netbios_name** | `netbios-name` | NVARCHAR(MAX) | NetBIOS host name obtained via SMB enumeration. |
| **host_fqdn** | `host-fqdn` | NVARCHAR(MAX) | Fully-qualified domain name of the target host, if resolved. |
| **host_rdns** | `host-rdns` | NVARCHAR(MAX) | Reverse DNS lookup result for host_ip. |
| **system_type** | `system-type` | NVARCHAR(MAX) | Asset role discovered (server, workstation, router, etc.). |
| **operating_system_id** | `os` | NVARCHAR(MAX) | OS identifier string from the Tenable feed. |
| **operating_system** | `operating-system` | NVARCHAR(MAX) | OS string determined by local checks, banners, or fingerprinting. |
| **operating_system_conf** | `operating-system-conf` | NVARCHAR(MAX) | Confidence level for operating system identification. |
| **operating_system_method** | `operating-system-method` | NVARCHAR(MAX) | Detection method used (local_checks, ssh_fingerprint, banner). |
| **sinfp_signature** | `sinfp-signature` | NVARCHAR(MAX) | Signature string from SINFP TCP/IP stack fingerprint plugin. |
| **Credentialed_Scan** | `Credentialed_Scan` | NVARCHAR(100) | Whether host assessment was performed with valid credentials (true/false). |
| **LastAuthenticatedResults** | `LastAuthenticatedResults` | NVARCHAR(MAX) | Whether last credentialed attempt on the host succeeded (true/false). |
| **local_checks_proto** | `local-checks-proto` | NVARCHAR(100) | Protocol used for local checks (SMB, SSH, WMI). |
| **smb_login_used** | `smb-login-used` | NVARCHAR(MAX) | Username that successfully authenticated for SMB local checks. |
| **wmi_domain** | `wmi-domain` | NVARCHAR(MAX) | Windows domain name used for WMI credentialed checks. |
| **policy_used** | `policy-used` | NVARCHAR(MAX) | Name of the scan policy/template executed. |
| **patch_summary_total_cves** | `patch-summary-total-cves` | NVARCHAR(MAX) | Number of CVEs addressed by patches (from patch summary plugins). |
| **host_start** | `HOST_START` | NVARCHAR(MAX) | Human-readable timestamp when scanning of the host began. |
| **host_start_timestamp** | `HOST_START_TIMESTAMP` | BIGINT | Unix epoch (seconds) of host_start. |
| **host_end** | `HOST_END` | NVARCHAR(MAX) | Human-readable timestamp when scanning of the host finished. |
| **host_end_timestamp** | `HOST_END_TIMESTAMP` | BIGINT | Unix epoch (seconds) of host_end. |

### Report Item Attributes

Extracted from `ReportItem` XML attributes. Present on every finding.

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **pluginID** | `pluginID` | INT | Numeric identifier of the Nessus plugin. |
| **pluginName** | `pluginName` | NVARCHAR(MAX) | Human-readable name/title of the plugin. |
| **pluginFamily** | `pluginFamily` | NVARCHAR(MAX) | Plugin family category (e.g., Windows, Databases, CGI). |
| **port** | `port` | INT | Network port associated with the finding (0 for host-level issues). |
| **protocol** | `protocol` | NVARCHAR(100) | Transport protocol (tcp, udp, icmp). |
| **svc_name** | `svc_name` | NVARCHAR(MAX) | Service name associated with the open port (from nmap-services). |
| **severity** | `severity` | INT | Raw Nessus severity value (0-4). Not stored directly; used to compute severity_rating and severity_reversed. |

### Report Item Child Elements

Extracted from child elements within each `ReportItem`. Most are optional and may be empty for a given finding.

#### Vulnerability Details

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **synopsis** | `synopsis` | NVARCHAR(MAX) | One-sentence summary of the issue. |
| **description** | `description` | NVARCHAR(MAX) | Full textual description of the vulnerability or plugin purpose. |
| **solution** | `solution` | NVARCHAR(MAX) | Vendor or Tenable remediation guidance. |
| **risk_factor** | `risk_factor` | NVARCHAR(100) | Severity string (None, Low, Medium, High, Critical) from CVSS or Tenable override. |
| **plugin_output** | `plugin_output` | NVARCHAR(MAX) | Verbose per-host output returned by the plugin (evidence, banners, file lists). |
| **plugin_type** | `plugin_type` | NVARCHAR(100) | Plugin category: remote, local, compliance, patch-management. |
| **plugin_publication_date** | `plugin_publication_date` | DATE | Date Tenable originally published the plugin. |
| **plugin_modification_date** | `plugin_modification_date` | DATE | Date Tenable last modified the plugin in the feed. |
| **script_version** | `script_version` | NVARCHAR(MAX) | Revision string of the plugin script. |
| **see_also** | `see_also` | NVARCHAR(MAX) | URLs or document references for additional advisories or guides. |
| **age_of_vuln** | `age_of_vuln` | NVARCHAR(MAX) | Range of days since the NVD published the vulnerability, as a bucket (Tenable's VPR docs list 0-7, 7-30, 30-60, 60-180, 180-365, 365-730 and more than 730 days). Not an exact day count. VPR key driver. |

#### CVSS v2 Scores

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cvss_base_score** | `cvss_base_score` | NVARCHAR(MAX) | NVD CVSS v2 Base Score (0.0-10.0). |
| **cvss_vector** | `cvss_vector` | NVARCHAR(MAX) | Complete CVSS v2 Base vector string (e.g., `AV:N/AC:L/...`). |
| **cvss_score_rationale** | `cvss_score_rationale` | NVARCHAR(MAX) | Free-text explanation of Tenable's CVSS override, if applied. |
| **cvss_score_source** | `cvss_score_source` | NVARCHAR(MAX) | Origin of the CVSS v2 score: NVD, Vendor, or Tenable. |
| **cvss_temporal_score** | `cvss_temporal_score` | NVARCHAR(MAX) | CVSS v2 Temporal Score computed by Tenable. |
| **cvss_temporal_vector** | `cvss_temporal_vector` | NVARCHAR(MAX) | Vector string for CVSS v2 Temporal metrics. |

#### CVSS v3 Scores

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cvss3_base_score** | `cvss3_base_score` | NVARCHAR(MAX) | NVD CVSS v3.x Base Score. |
| **cvss3_vector** | `cvss3_vector` | NVARCHAR(MAX) | CVSS v3.x Base vector string. |
| **cvss3_score_source** | `cvss3_score_source` | NVARCHAR(MAX) | Origin of the CVSS v3 score. |
| **cvss3_temporal_score** | `cvss3_temporal_score` | NVARCHAR(MAX) | Tenable-calculated CVSS v3.x Temporal Score. |
| **cvss3_temporal_vector** | `cvss3_temporal_vector` | NVARCHAR(MAX) | Vector string for v3 Temporal metrics. |
| **cvssV3_impactScore** | `cvssV3_impactScore` | NVARCHAR(MAX) | CVSS v3.x Impact sub-score from NVD. |
| **vpr_score** | `vpr_score` | NVARCHAR(MAX) | Tenable Vulnerability Priority Rating (0.0-10.0) combining CVSS and threat intel. |

#### CVE and Security References

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cve** | `cve` | NVARCHAR(MAX) | CVE identifier(s) tied to the plugin. Multiple CVEs are comma-separated. |
| **cwe** | `cwe` | NVARCHAR(MAX) | Common Weakness Enumeration ID(s) (e.g., CWE-79). |
| **bid** | `bid` | NVARCHAR(MAX) | SecurityFocus BugTraq identifier(s). |
| **edb_id** | `edb-id` | NVARCHAR(MAX) | Exploit-DB reference number(s). |
| **cea_id** | `cea-id` | NVARCHAR(MAX) | CERT/CC Vulnerability Note or CISA KEV catalogue identifier. |
| **cert** | `cert` | NVARCHAR(MAX) | CERT vulnerability note number(s) (e.g., VU#12345). |
| **cisa_known_exploited** | `cisa-known-exploited` | NVARCHAR(100) | Whether CVE is in CISA's Known Exploited Vulnerabilities catalogue (true/false). |
| **cisa_ncas** | `cisa-ncas` | NVARCHAR(MAX) | CISA National Cyber Awareness System advisory reference. |
| **cisco_bug_id** | `cisco-bug-id` | NVARCHAR(MAX) | Cisco PSIRT bug ID(s) (e.g., CSCvx12345). |
| **cisco_sa** | `cisco-sa` | NVARCHAR(MAX) | Cisco Security Advisory ID. |
| **iava** | `iava` | NVARCHAR(MAX) | DISA Information Assurance Vulnerability Alert ID. |
| **iavb** | `iavb` | NVARCHAR(MAX) | DISA Information Assurance Vulnerability Bulletin ID. |
| **iavt** | `iavt` | NVARCHAR(MAX) | DISA Information Assurance Technical Advisory ID. |
| **icsa** | `icsa` | NVARCHAR(MAX) | ICS-CERT advisory identifier (OT/ICS devices). |
| **msft** | `msft` | NVARCHAR(MAX) | Microsoft Security Advisory reference (MSFT-YYYY-NN). |
| **mskb** | `mskb` | NVARCHAR(MAX) | Microsoft Knowledge Base article number. |
| **tra** | `tra` | NVARCHAR(MAX) | Tenable Research Advisory reference. |
| **vmsa** | `vmsa` | NVARCHAR(MAX) | VMware Security Advisory ID(s). |
| **secunia** | `secunia` | NVARCHAR(MAX) | Secunia advisory reference number. |
| **stig_severity** | `stig_severity` | NVARCHAR(MAX) | DISA STIG severity mapping (Cat I/II/III). |

#### Exploit Intelligence

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **exploit_available** | `exploit_available` | NVARCHAR(100) | Whether public exploit code is available (true/false). |
| **exploitability_ease** | `exploitability_ease` | NVARCHAR(MAX) | How easy the issue is to exploit. Tenable's Nessus docs list the values: Exploits are available, No exploit is required, No known exploits are available. |
| **exploit_code_maturity** | `exploit_code_maturity` | NVARCHAR(MAX) | VPR driver: relative maturity of possible exploit code. Values: High, Functional, PoC, Unproven. |
| **exploit_framework_canvas** | `exploit_framework_canvas` | NVARCHAR(100) | Exploit present in Immunity Canvas (true/false). |
| **canvas_package** | `canvas_package` | NVARCHAR(MAX) | Name of the Canvas exploit module. |
| **exploit_framework_core** | `exploit_framework_core` | NVARCHAR(100) | Exploit present in Core Impact (true/false). |
| **exploit_framework_metasploit** | `exploit_framework_metasploit` | NVARCHAR(100) | Exploit present in Metasploit Framework (true/false). |
| **metasploit_name** | `metasploit_name` | NVARCHAR(MAX) | Name of the associated Metasploit module(s). |
| **exploited_by_malware** | `exploited_by_malware` | NVARCHAR(100) | Whether vulnerability is exploited in active malware (true/false). |
| **exploited_by_nessus** | `exploited_by_nessus` | NVARCHAR(100) | Whether Nessus can exploit this issue itself (true/false). |
| **in_the_news** | `in_the_news` | NVARCHAR(100) | Tenable threat-intel flag for recent mainstream media coverage (true/false). |

#### Threat Metrics

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **threat_recency** | `threat_recency` | NVARCHAR(MAX) | VPR driver: bucket of days since a threat event was seen (Tenable's VPR docs list 0-7, 7-30, 30-120, 120-365 and more than 365 days). |
| **threat_intensity_last_28** | `threat_intensity_last_28` | NVARCHAR(MAX) | VPR driver: relative intensity of recently observed threat events, as a bucket: Very Low, Low, Medium, High, Very High. Not a number. |
| **threat_sources_last_28** | `threat_sources_last_28` | NVARCHAR(MAX) | VPR driver: the sources where threat events were seen in the last 28 days (for example social media or the dark web), as a list. Not a count. |
| **product_coverage** | `product_coverage` | NVARCHAR(MAX) | VPR driver: relative number of unique products affected by the vulnerability, as a bucket: Low, Medium, High, Very High. Not a percentage. |
| **unsupported_by_vendor** | `unsupported_by_vendor` | NVARCHAR(100) | Product is end-of-life and no longer receives patches (true/false). |

#### Asset and Inventory

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cpe** | `cpe` | NVARCHAR(MAX) | Item-level Common Platform Enumeration string for the affected product. |
| **agent** | `agent` | NVARCHAR(100) | UUID or hostname of the Nessus Agent that performed the scan. |
| **always_run** | `always_run` | NVARCHAR(100) | Plugin runs on every scan regardless of credential/port status (true/false). |
| **asset_inventory** | `asset_inventory` | NVARCHAR(MAX) | Plugin purpose is inventory enumeration, not vulnerability detection (true/false). |
| **asset_inventory_category** | `asset_inventory_category` | NVARCHAR(MAX) | Class of inventory info: Software, Hardware, Service. |
| **asset_categories** | `asset_categories` | NVARCHAR(MAX) | Tenable-defined business/technical categories (e.g., Server, Workstation). |
| **hardware_inventory** | `hardware_inventory` | NVARCHAR(MAX) | Result pertains to hardware identification (true/false). |
| **generated_plugin** | `generated_plugin` | NVARCHAR(MAX) | Plugin is auto-generated from feed data, not hand-authored (true/false). |
| **thorough_tests** | `thorough_tests` | NVARCHAR(100) | Whether "Enable thorough tests" was on (true/false). |
| **os_identification** | `os_identification` | NVARCHAR(MAX) | OS fingerprint information from item-level detection. |

#### Dates

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **patch_publication_date** | `patch_publication_date` | DATE | Date vendor released the official patch. |
| **vuln_publication_date** | `vuln_publication_date` | DATE | Original public disclosure date of the vulnerability. |

### Derived/Computed Fields

These fields are computed during extraction and do not exist directly in the .nessus XML.

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **severity_rating** | (computed) | NVARCHAR(100) | Human-readable severity: Critical, High, Medium, Low, Informational. Computed from raw `severity` attribute. |
| **severity_reversed** | (computed) | INT | Inverted severity (0=Critical, 4=Informational). Useful for ascending sort by severity. |
| **scan_date** | (computed) | DATE | Date extracted from `host_start` timestamp (YYYY-MM-DD). |
| **file_name** | (computed) | NVARCHAR(255) | Basename of the source .nessus file. Added during extraction for traceability. Named `file_name` in the CSV and Excel output and `file_path` in the SQL table below (it still holds the basename, not a path). |

### Additional .nessus Fields (Not Extracted by Scripts)

The source of this list is not recorded. The companion Tenable_Nessus_Scripts do not extract them, and their element names have not been checked against Tenable's documentation or a real file. They may appear in specialized scans or extended plugin output.

| Field | Description |
|-------|-------------|
| **d2_elliot_name** | Name of the exploit module in the D2 Elliot framework. |
| **exploit_framework_d2_elliot** | Exploit present in D2 Elliot (true/false). |
| **exploit_framework_exploithub** | Exploit present in ExploitHub marketplace (true/false). |
| **malware** | Tenable classification that plugin result corresponds to known malware. |

## SQL Server CREATE TABLE

Ready-to-use schema matching the companion scripts. Adjust `YourTableName` as needed.

```sql
CREATE TABLE [YourTableName] (
    host_ip NVARCHAR(255),
    mac_address NVARCHAR(MAX),
    virtual_mac_address NVARCHAR(MAX),
    hostname NVARCHAR(MAX),
    netbios_name NVARCHAR(MAX),
    host_fqdn NVARCHAR(MAX),
    host_rdns NVARCHAR(MAX),
    system_type NVARCHAR(MAX),
    cpe NVARCHAR(MAX),
    operating_system_id NVARCHAR(MAX),
    operating_system NVARCHAR(MAX),
    severity_rating NVARCHAR(100),
    risk_factor NVARCHAR(100),
    severity_reversed INT,
    pluginName NVARCHAR(MAX),
    synopsis NVARCHAR(MAX),
    description NVARCHAR(MAX),
    solution NVARCHAR(MAX),
    port INT,
    protocol NVARCHAR(100),
    svc_name NVARCHAR(MAX),
    plugin_output NVARCHAR(MAX),
    plugin_type NVARCHAR(100),
    plugin_publication_date DATE,
    plugin_modification_date DATE,
    stig_severity NVARCHAR(MAX),
    cvss_base_score NVARCHAR(MAX),
    cvss_vector NVARCHAR(MAX),
    cvss_score_rationale NVARCHAR(MAX),
    cvss_score_source NVARCHAR(MAX),
    cvss_temporal_score NVARCHAR(MAX),
    cvss_temporal_vector NVARCHAR(MAX),
    cvss3_base_score NVARCHAR(MAX),
    cvss3_vector NVARCHAR(MAX),
    cvss3_score_source NVARCHAR(MAX),
    cvss3_temporal_score NVARCHAR(MAX),
    cvss3_temporal_vector NVARCHAR(MAX),
    cvssV3_impactScore NVARCHAR(MAX),
    vpr_score NVARCHAR(MAX),
    patch_publication_date DATE,
    patch_summary_total_cves NVARCHAR(MAX),
    vuln_publication_date DATE,
    age_of_vuln NVARCHAR(MAX),
    product_coverage NVARCHAR(MAX),
    exploit_available NVARCHAR(100),
    exploitability_ease NVARCHAR(MAX),
    exploit_code_maturity NVARCHAR(MAX),
    exploit_framework_canvas NVARCHAR(100),
    canvas_package NVARCHAR(MAX),
    exploit_framework_core NVARCHAR(100),
    exploit_framework_metasploit NVARCHAR(100),
    metasploit_name NVARCHAR(MAX),
    exploited_by_malware NVARCHAR(100),
    exploited_by_nessus NVARCHAR(100),
    edb_id NVARCHAR(MAX),
    threat_recency NVARCHAR(MAX),
    threat_intensity_last_28 NVARCHAR(MAX),
    threat_sources_last_28 NVARCHAR(MAX),
    local_checks_proto NVARCHAR(100),
    smb_login_used NVARCHAR(MAX),
    wmi_domain NVARCHAR(MAX),
    thorough_tests NVARCHAR(100),
    Credentialed_Scan NVARCHAR(100),
    LastAuthenticatedResults NVARCHAR(MAX),
    policy_used NVARCHAR(MAX),
    host_start NVARCHAR(MAX),
    host_start_timestamp BIGINT,
    host_end NVARCHAR(MAX),
    host_end_timestamp BIGINT,
    os_identification NVARCHAR(MAX),
    operating_system_conf NVARCHAR(MAX),
    operating_system_method NVARCHAR(MAX),
    sinfp_signature NVARCHAR(MAX),
    pluginID INT,
    pluginFamily NVARCHAR(MAX),
    script_version NVARCHAR(MAX),
    agent NVARCHAR(100),
    always_run NVARCHAR(100),
    asset_inventory NVARCHAR(MAX),
    asset_inventory_category NVARCHAR(MAX),
    asset_categories NVARCHAR(MAX),
    hardware_inventory NVARCHAR(MAX),
    bid NVARCHAR(MAX),
    cea_id NVARCHAR(MAX),
    cert NVARCHAR(MAX),
    cisa_known_exploited NVARCHAR(100),
    cisa_ncas NVARCHAR(MAX),
    cisco_bug_id NVARCHAR(MAX),
    cisco_sa NVARCHAR(MAX),
    cve NVARCHAR(MAX),
    cwe NVARCHAR(MAX),
    iava NVARCHAR(MAX),
    iavb NVARCHAR(MAX),
    iavt NVARCHAR(MAX),
    icsa NVARCHAR(MAX),
    msft NVARCHAR(MAX),
    mskb NVARCHAR(MAX),
    tra NVARCHAR(MAX),
    vmsa NVARCHAR(MAX),
    secunia NVARCHAR(MAX),
    unsupported_by_vendor NVARCHAR(100),
    see_also NVARCHAR(MAX),
    in_the_news NVARCHAR(100),
    generated_plugin NVARCHAR(MAX),
    file_path NVARCHAR(255),
    scan_date DATE
);
```

### Notes on the table

- **Columns:** 106, one per field above except `severity` (the raw value is not stored; `severity_rating` and `severity_reversed` replace it). The table has one `cpe` and one `os_identification` column. Both hold the item-level value, because that is what the companion scripts store; the host-level `cpe` tag is not kept.
- **Column name:** `file_path` is the SQL name for the `file_name` field, and it holds the file's basename.
- **No key or index.** One row is one finding on one host in one scan file. Do not put a unique key on (`host_ip`, `pluginID`, `port`, `protocol`, `scan_date`, `file_path`) or any other short column list: findings that differ only in their text share all of those values (compliance checks under one plugin ID, for example), and a unique index would silently drop all but one of them. To avoid loading the same file twice, record each loaded file's name and SHA-256 and skip files already recorded; the companion `nessus_to_sql.py` does this in a `<table>_import_log` table. If you add a key anyway, a hash of all 106 column values is the only choice that never merges two different findings.
- **Text types:** the companion script uses `NVARCHAR` throughout, and `NVARCHAR(MAX)` for every free-text and multi-value column (`synopsis` and the joined `cwe`, `bid`, `mskb`, `cve` lists included), because Tenable does not document length limits and I have no scan corpus to size them from. Only `host_ip` and `file_path` (255) and `protocol`, the true/false flags and the severity and risk labels (100) use a fixed length. If you copy an older version of this table with `VARCHAR(150)` reference columns, a finding with several references can fail with "String or binary data would be truncated"; widen those columns to `NVARCHAR(MAX)`.
- **Numbers and dates:** `port`, `pluginID` and `severity_reversed` are INT, the two `*_timestamp` columns are BIGINT, and the five date columns are DATE. Load empty values as NULL, not as an empty string, and parse the dates yourself (Nessus writes `2024/03/05`) so the server's language setting cannot misread them.
- **Provenance:** the field list is what the companion Tenable_Nessus_Scripts extract. It was not derived from a specific Nessus or Tenable version, and it was not checked against a real .nessus file.

## Data Type Reference

| SQL Type | When to Use |
|----------|-------------|
| NVARCHAR(n) | Short fixed-vocabulary text: boolean flags (true/false), severity and risk labels, the file name |
| NVARCHAR(MAX) | Everything else: names, descriptions, plugin output, solutions, and every field that can hold several joined values (CVE, CWE, BID, MSKB and the other reference IDs, MAC addresses) |
| INT | Whole numbers: plugin IDs, port numbers, severity integers |
| BIGINT | Unix timestamps (epoch seconds) |
| DATE | Calendar dates (YYYY-MM-DD): publication dates, scan dates |

## Conventions

- **Booleans**: Stored as NVARCHAR. Normalize to `true`/`false` (case-insensitive). Raw XML may use `True/False`, `Yes/No`, or `0/1`.
- **Timestamps**: `host_start`/`host_end` are human-readable strings; the corresponding `*_timestamp` fields are Unix epoch (seconds). Prefer epoch for joins and sorting.
- **Multi-value fields**: when an item lists the same reference more than once, the companion scripts keep all of them, joined with `, `. This covers `cve`, `cwe`, `bid`, `edb_id`, `cea_id`, `cert`, `cisa_ncas`, `cisco_bug_id`, `cisco_sa`, `iava`, `iavb`, `iavt`, `icsa`, `msft`, `mskb`, `tra`, `vmsa` and `secunia`. `mac_address` may also contain multiple addresses. Split on `, ` when normalizing.
- **Severities**: `risk_factor` is categorical text (None/Low/Medium/High/Critical). `severity_reversed` is an integer (0=Critical through 4=Informational). `vpr_score` is a decimal string (0.0-10.0).

## License

MIT License. See [LICENSE](LICENSE) for details.
