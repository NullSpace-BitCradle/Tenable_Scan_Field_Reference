# Tenable Scan Field Reference

A data dictionary for the **.nessus XML** scan export format. Documents every extractable field with its XML source location, SQL data type, and description, so you can parse .nessus files and import them into a relational database with the correct schema.

**Companion repo:** [Tenable_Nessus_Scripts](https://github.com/NullSpace-BitCradle/Tenable_Nessus_Scripts) -- Python scripts that extract these fields into CSV, Excel, and SQL Server.

## Quick Start

1. Export scan results from Tenable.io or Nessus in `.nessus` format (XML)
2. Use the field tables below to build your database schema (or use the [CREATE TABLE example](#sql-server-create-table))
3. Parse `ReportHost` elements for host properties, then `ReportItem` elements for vulnerability data
4. Store raw `plugin_output` in a MAX/TEXT column -- it can be very large

## Field Reference

Fields are organized by where they appear in the .nessus XML structure. The **SQL Type** column matches the schema used by the companion [Tenable_Nessus_Scripts](https://github.com/NullSpace-BitCradle/Tenable_Nessus_Scripts) SQL import script.

### Host Properties

Extracted from `ReportHost > HostProperties > tag[@name]` elements. One set of values per scanned host.

| Field | SQL Type | Description |
|-------|----------|-------------|
| **host_ip** | VARCHAR(200) | Primary IPv4/IPv6 address of the scanned asset. |
| **mac_address** | NVARCHAR(MAX) | MAC address(es) discovered for the host. Multiple addresses are comma-separated. |
| **virtual_mac_address** | NVARCHAR(1000) | MAC address of virtual adapter reported by virtualized hosts. |
| **hostname** | VARCHAR(100) | NetBIOS/short hostname gathered from the host (SMB, SSH, etc.). |
| **netbios_name** | VARCHAR(100) | NetBIOS host name obtained via SMB enumeration. |
| **host_fqdn** | VARCHAR(150) | Fully-qualified domain name of the target host, if resolved. |
| **host_rdns** | VARCHAR(150) | Reverse DNS lookup result for host_ip. |
| **system_type** | VARCHAR(25) | Asset role discovered (server, workstation, router, etc.). |
| **cpe** | VARCHAR(1000) | Host-level Common Platform Enumeration string(s) describing the platform. |
| **operating_system_id** | VARCHAR(150) | OS identifier string from the Tenable feed. |
| **operating_system** | VARCHAR(200) | OS string determined by local checks, banners, or fingerprinting. |
| **operating_system_conf** | VARCHAR(150) | Confidence level for operating system identification. |
| **operating_system_method** | VARCHAR(150) | Detection method used (local_checks, ssh_fingerprint, banner). |
| **os_identification** | VARCHAR(150) | Aggregated text summary of OS fingerprint information. |
| **sinfp_signature** | VARCHAR(250) | Signature string from SINFP TCP/IP stack fingerprint plugin. |
| **Credentialed_Scan** | VARCHAR(15) | Whether host assessment was performed with valid credentials (true/false). |
| **LastAuthenticatedResults** | VARCHAR(50) | Whether last credentialed attempt on the host succeeded (true/false). |
| **local_checks_proto** | VARCHAR(15) | Protocol used for local checks (SMB, SSH, WMI). |
| **smb_login_used** | VARCHAR(150) | Username that successfully authenticated for SMB local checks. |
| **wmi_domain** | VARCHAR(150) | Windows domain name used for WMI credentialed checks. |
| **policy_used** | VARCHAR(150) | Name of the scan policy/template executed. |
| **patch_summary_total_cves** | VARCHAR(150) | Number of CVEs addressed by patches (from patch summary plugins). |
| **host_start** | VARCHAR(150) | Human-readable timestamp when scanning of the host began. |
| **host_start_timestamp** | INT | Unix epoch (seconds) of host_start. |
| **host_end** | VARCHAR(150) | Human-readable timestamp when scanning of the host finished. |
| **host_end_timestamp** | INT | Unix epoch (seconds) of host_end. |

### Report Item Attributes

Extracted from `ReportItem` XML attributes. Present on every finding.

| Field | SQL Type | Description |
|-------|----------|-------------|
| **pluginID** | INT | Numeric identifier of the Nessus plugin. |
| **pluginName** | VARCHAR(MAX) | Human-readable name/title of the plugin. |
| **pluginFamily** | VARCHAR(150) | Plugin family category (e.g., Windows, Databases, CGI). |
| **port** | INT | Network port associated with the finding (0 for host-level issues). |
| **protocol** | VARCHAR(15) | Transport protocol (tcp, udp, icmp). |
| **svc_name** | VARCHAR(150) | Service name associated with the open port (from nmap-services). |
| **severity** | INT | Raw Nessus severity value (0-4). Not stored directly; used to compute severity_rating and severity_reversed. |

### Report Item Child Elements

Extracted from child elements within each `ReportItem`. Most are optional and may be empty for a given finding.

#### Vulnerability Details

| Field | SQL Type | Description |
|-------|----------|-------------|
| **synopsis** | VARCHAR(255) | One-sentence summary of the issue. |
| **description** | VARCHAR(MAX) | Full textual description of the vulnerability or plugin purpose. |
| **solution** | VARCHAR(MAX) | Vendor or Tenable remediation guidance. |
| **risk_factor** | VARCHAR(15) | Severity string (None, Low, Medium, High, Critical) from CVSS or Tenable override. |
| **plugin_output** | VARCHAR(MAX) | Verbose per-host output returned by the plugin (evidence, banners, file lists). |
| **plugin_type** | VARCHAR(15) | Plugin category: remote, local, compliance, patch-management. |
| **plugin_publication_date** | DATE | Date Tenable originally published the plugin. |
| **plugin_modification_date** | DATE | Date Tenable last modified the plugin in the feed. |
| **script_version** | VARCHAR(150) | Revision string of the plugin script. |
| **see_also** | VARCHAR(MAX) | URLs or document references for additional advisories or guides. |
| **age_of_vuln** | VARCHAR(100) | Days between original disclosure date and scan time. VPR key driver. |

#### CVSS v2 Scores

| Field | SQL Type | Description |
|-------|----------|-------------|
| **cvss_base_score** | VARCHAR(100) | NVD CVSS v2 Base Score (0.0-10.0). |
| **cvss_vector** | VARCHAR(150) | Complete CVSS v2 Base vector string (e.g., `AV:N/AC:L/...`). |
| **cvss_score_rationale** | VARCHAR(500) | Free-text explanation of Tenable's CVSS override, if applied. |
| **cvss_score_source** | VARCHAR(100) | Origin of the CVSS v2 score: NVD, Vendor, or Tenable. |
| **cvss_temporal_score** | VARCHAR(100) | CVSS v2 Temporal Score computed by Tenable. |
| **cvss_temporal_vector** | VARCHAR(100) | Vector string for CVSS v2 Temporal metrics. |

#### CVSS v3 Scores

| Field | SQL Type | Description |
|-------|----------|-------------|
| **cvss3_base_score** | VARCHAR(100) | NVD CVSS v3.x Base Score. |
| **cvss3_vector** | VARCHAR(150) | CVSS v3.x Base vector string. |
| **cvss3_score_source** | VARCHAR(100) | Origin of the CVSS v3 score. |
| **cvss3_temporal_score** | VARCHAR(100) | Tenable-calculated CVSS v3.x Temporal Score. |
| **cvss3_temporal_vector** | VARCHAR(100) | Vector string for v3 Temporal metrics. |
| **cvssV3_impactScore** | VARCHAR(100) | CVSS v3.x Impact sub-score from NVD. |
| **vpr_score** | VARCHAR(100) | Tenable Vulnerability Priority Rating (0.0-10.0) combining CVSS and threat intel. |

#### CVE and Security References

| Field | SQL Type | Description |
|-------|----------|-------------|
| **cve** | NVARCHAR(4000) | CVE identifier(s) tied to the plugin. Multiple CVEs are comma-separated. |
| **cwe** | VARCHAR(150) | Common Weakness Enumeration ID(s) (e.g., CWE-79). |
| **bid** | VARCHAR(150) | SecurityFocus BugTraq identifier(s). |
| **edb_id** | VARCHAR(150) | Exploit-DB reference number(s). |
| **cea_id** | VARCHAR(150) | CERT/CC Vulnerability Note or CISA KEV catalogue identifier. |
| **cert** | VARCHAR(150) | CERT vulnerability note number(s) (e.g., VU#12345). |
| **cisa_known_exploited** | VARCHAR(150) | Whether CVE is in CISA's Known Exploited Vulnerabilities catalogue (true/false). |
| **cisa_ncas** | VARCHAR(150) | CISA National Cyber Awareness System advisory reference. |
| **cisco_bug_id** | VARCHAR(150) | Cisco PSIRT bug ID(s) (e.g., CSCvx12345). |
| **cisco_sa** | VARCHAR(150) | Cisco Security Advisory ID. |
| **iava** | VARCHAR(150) | DISA Information Assurance Vulnerability Alert ID. |
| **iavb** | VARCHAR(150) | DISA Information Assurance Vulnerability Bulletin ID. |
| **iavt** | VARCHAR(150) | DISA Information Assurance Technical Advisory ID. |
| **icsa** | VARCHAR(150) | ICS-CERT advisory identifier (OT/ICS devices). |
| **msft** | VARCHAR(150) | Microsoft Security Advisory reference (MSFT-YYYY-NN). |
| **mskb** | VARCHAR(150) | Microsoft Knowledge Base article number. |
| **tra** | VARCHAR(150) | Tenable Research Advisory reference. |
| **vmsa** | VARCHAR(150) | VMware Security Advisory ID(s). |
| **secunia** | VARCHAR(150) | Secunia advisory reference number. |
| **stig_severity** | VARCHAR(100) | DISA STIG severity mapping (Cat I/II/III). |

#### Exploit Intelligence

| Field | SQL Type | Description |
|-------|----------|-------------|
| **exploit_available** | VARCHAR(15) | Whether public exploit code is available (true/false). |
| **exploitability_ease** | VARCHAR(150) | Qualitative rating: No Known, Theoretical, PoC, Functional, High. |
| **exploit_code_maturity** | VARCHAR(150) | VPR driver category: Unchecked, PoC, Functional, High. |
| **exploit_framework_canvas** | VARCHAR(15) | Exploit present in Immunity Canvas (true/false). |
| **canvas_package** | VARCHAR(150) | Name of the Canvas exploit module. |
| **exploit_framework_core** | VARCHAR(15) | Exploit present in Core Impact (true/false). |
| **exploit_framework_metasploit** | VARCHAR(15) | Exploit present in Metasploit Framework (true/false). |
| **metasploit_name** | VARCHAR(150) | Name of the associated Metasploit module(s). |
| **exploited_by_malware** | VARCHAR(15) | Whether vulnerability is exploited in active malware (true/false). |
| **exploited_by_nessus** | VARCHAR(15) | Nessus itself can exploit this issue for more info (true/false). |
| **in_the_news** | VARCHAR(15) | Tenable threat-intel flag for recent mainstream media coverage (true/false). |

#### Threat Metrics

| Field | SQL Type | Description |
|-------|----------|-------------|
| **threat_recency** | VARCHAR(150) | Ordinal bucket of how recently exploits were observed (Very Recent, Recent, etc.). |
| **threat_intensity_last_28** | VARCHAR(150) | Aggregate metric (0-100) of exploit activity in last 28 days. |
| **threat_sources_last_28** | VARCHAR(150) | Count of discrete threat sources (exploit kits, campaigns) in last 28 days. |
| **product_coverage** | VARCHAR(150) | Percentage of vendor-known vulnerable products covered by patches. VPR driver. |
| **unsupported_by_vendor** | VARCHAR(15) | Product is end-of-life and no longer receives patches (true/false). |

#### Asset and Inventory

| Field | SQL Type | Description |
|-------|----------|-------------|
| **cpe** | VARCHAR(1000) | Item-level Common Platform Enumeration string for the affected product. |
| **agent** | VARCHAR(150) | UUID or hostname of the Nessus Agent that performed the scan. |
| **always_run** | VARCHAR(150) | Plugin runs on every scan regardless of credential/port status (true/false). |
| **asset_inventory** | VARCHAR(150) | Plugin purpose is inventory enumeration, not vulnerability detection (true/false). |
| **asset_inventory_category** | VARCHAR(150) | Class of inventory info: Software, Hardware, Service. |
| **asset_categories** | VARCHAR(150) | Tenable-defined business/technical categories (e.g., Server, Workstation). |
| **hardware_inventory** | VARCHAR(150) | Result pertains to hardware identification (true/false). |
| **generated_plugin** | VARCHAR(150) | Plugin is auto-generated from feed data, not hand-authored (true/false). |
| **thorough_tests** | VARCHAR(150) | Whether "Enable thorough tests" was on (true/false). |
| **os_identification** | VARCHAR(150) | OS fingerprint information from item-level detection. |

#### Dates

| Field | SQL Type | Description |
|-------|----------|-------------|
| **patch_publication_date** | DATE | Date vendor released the official patch. |
| **vuln_publication_date** | DATE | Original public disclosure date of the vulnerability. |

### Derived/Computed Fields

These fields are computed during extraction and do not exist directly in the .nessus XML.

| Field | SQL Type | Description |
|-------|----------|-------------|
| **severity_rating** | VARCHAR(15) | Human-readable severity: Critical, High, Medium, Low, Informational. Computed from raw `severity` attribute. |
| **severity_reversed** | INT | Inverted severity (0=Critical, 4=Informational). Useful for ascending sort by severity. |
| **scan_date** | DATE | Date extracted from `host_start` timestamp (YYYY-MM-DD). |
| **file_name** | VARCHAR(255) | Basename of the source .nessus file. Added during extraction for traceability. |

### Additional .nessus Fields (Not Extracted by Scripts)

These fields exist in the .nessus XML format but are not currently extracted by the companion Tenable_Nessus_Scripts. They may appear in specialized scans or extended plugin output.

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
    host_ip VARCHAR(200),
    mac_address NVARCHAR(MAX),
    virtual_mac_address NVARCHAR(1000),
    hostname VARCHAR(100),
    netbios_name VARCHAR(100),
    host_fqdn VARCHAR(150),
    host_rdns VARCHAR(150),
    system_type VARCHAR(25),
    cpe VARCHAR(1000),
    operating_system_id VARCHAR(150),
    operating_system VARCHAR(200),
    severity_rating VARCHAR(15),
    risk_factor VARCHAR(15),
    severity_reversed INT,
    pluginName VARCHAR(MAX),
    synopsis VARCHAR(255),
    description VARCHAR(MAX),
    solution VARCHAR(MAX),
    port INT,
    protocol VARCHAR(15),
    svc_name VARCHAR(150),
    plugin_output VARCHAR(MAX),
    plugin_type VARCHAR(15),
    plugin_publication_date DATE,
    plugin_modification_date DATE,
    stig_severity VARCHAR(100),
    cvss_base_score VARCHAR(100),
    cvss_vector VARCHAR(150),
    cvss_score_rationale VARCHAR(500),
    cvss_score_source VARCHAR(100),
    cvss_temporal_score VARCHAR(100),
    cvss_temporal_vector VARCHAR(100),
    cvss3_base_score VARCHAR(100),
    cvss3_vector VARCHAR(150),
    cvss3_score_source VARCHAR(100),
    cvss3_temporal_score VARCHAR(100),
    cvss3_temporal_vector VARCHAR(100),
    cvssV3_impactScore VARCHAR(100),
    vpr_score VARCHAR(100),
    patch_publication_date DATE,
    patch_summary_total_cves VARCHAR(150),
    vuln_publication_date DATE,
    age_of_vuln VARCHAR(100),
    product_coverage VARCHAR(150),
    exploit_available VARCHAR(15),
    exploitability_ease VARCHAR(150),
    exploit_code_maturity VARCHAR(150),
    exploit_framework_canvas VARCHAR(15),
    canvas_package VARCHAR(150),
    exploit_framework_core VARCHAR(15),
    exploit_framework_metasploit VARCHAR(15),
    metasploit_name VARCHAR(150),
    exploited_by_malware VARCHAR(15),
    exploited_by_nessus VARCHAR(15),
    edb_id VARCHAR(150),
    threat_recency VARCHAR(150),
    threat_intensity_last_28 VARCHAR(150),
    threat_sources_last_28 VARCHAR(150),
    local_checks_proto VARCHAR(15),
    smb_login_used VARCHAR(150),
    wmi_domain VARCHAR(150),
    thorough_tests VARCHAR(150),
    Credentialed_Scan VARCHAR(15),
    LastAuthenticatedResults VARCHAR(50),
    policy_used VARCHAR(150),
    host_start VARCHAR(150),
    host_start_timestamp INT,
    host_end VARCHAR(150),
    host_end_timestamp INT,
    os_identification VARCHAR(150),
    operating_system_conf VARCHAR(150),
    operating_system_method VARCHAR(150),
    sinfp_signature VARCHAR(250),
    pluginID INT,
    pluginFamily VARCHAR(150),
    script_version VARCHAR(150),
    agent VARCHAR(150),
    always_run VARCHAR(150),
    asset_inventory VARCHAR(150),
    asset_inventory_category VARCHAR(150),
    asset_categories VARCHAR(150),
    hardware_inventory VARCHAR(150),
    bid VARCHAR(150),
    cea_id VARCHAR(150),
    cert VARCHAR(150),
    cisa_known_exploited VARCHAR(150),
    cisa_ncas VARCHAR(150),
    cisco_bug_id VARCHAR(150),
    cisco_sa VARCHAR(150),
    cve NVARCHAR(4000),
    cwe VARCHAR(150),
    iava VARCHAR(150),
    iavb VARCHAR(150),
    iavt VARCHAR(150),
    icsa VARCHAR(150),
    msft VARCHAR(150),
    mskb VARCHAR(150),
    tra VARCHAR(150),
    vmsa VARCHAR(150),
    secunia VARCHAR(150),
    unsupported_by_vendor VARCHAR(15),
    see_also VARCHAR(MAX),
    in_the_news VARCHAR(15),
    generated_plugin VARCHAR(150),
    file_path VARCHAR(255),
    scan_date DATE
);
```

## Data Type Reference

| SQL Type | When to Use |
|----------|-------------|
| VARCHAR(n) | Short text: IDs, names, enumerations, boolean flags (true/false) |
| VARCHAR(MAX) | Unbounded text: descriptions, plugin output, solutions |
| NVARCHAR(n/MAX) | Text that may contain Unicode characters (CVE lists, MAC addresses) |
| INT | Whole numbers: plugin IDs, port numbers, Unix timestamps, severity integers |
| DATE | Calendar dates (YYYY-MM-DD): publication dates, scan dates |

## Conventions

- **Booleans**: Stored as VARCHAR. Normalize to `true`/`false` (case-insensitive). Raw XML may use `True/False`, `Yes/No`, or `0/1`.
- **Timestamps**: `host_start`/`host_end` are human-readable strings; the corresponding `*_timestamp` fields are Unix epoch (seconds). Prefer epoch for joins and sorting.
- **Multi-value fields**: `cve` contains comma-separated CVE IDs. `mac_address` may contain multiple addresses. Split on `, ` when normalizing.
- **Severities**: `risk_factor` is categorical text (None/Low/Medium/High/Critical). `severity_reversed` is an integer (0=Critical through 4=Informational). `vpr_score` is a decimal string (0.0-10.0).

## License

MIT License. See [LICENSE](LICENSE) for details.
