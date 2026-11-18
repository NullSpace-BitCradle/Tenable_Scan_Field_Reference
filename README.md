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
| **host_ip** | `host-ip` | VARCHAR(200) | Primary IPv4/IPv6 address of the scanned asset. |
| **mac_address** | `mac-address` | NVARCHAR(MAX) | MAC address(es) discovered for the host. Multiple addresses are comma-separated. |
| **virtual_mac_address** | `virtual-mac-address` | NVARCHAR(1000) | MAC address of virtual adapter reported by virtualized hosts. |
| **hostname** | `hostname` | VARCHAR(100) | Short hostname gathered from the host (SMB, SSH, etc.). See netbios_name for the NetBIOS name. |
| **netbios_name** | `netbios-name` | VARCHAR(100) | NetBIOS host name obtained via SMB enumeration. |
| **host_fqdn** | `host-fqdn` | VARCHAR(150) | Fully-qualified domain name of the target host, if resolved. |
| **host_rdns** | `host-rdns` | VARCHAR(150) | Reverse DNS lookup result for host_ip. |
| **system_type** | `system-type` | VARCHAR(25) | Asset role discovered (server, workstation, router, etc.). |
| **operating_system_id** | `os` | VARCHAR(150) | OS identifier string from the Tenable feed. |
| **operating_system** | `operating-system` | VARCHAR(200) | OS string determined by local checks, banners, or fingerprinting. |
| **operating_system_conf** | `operating-system-conf` | VARCHAR(150) | Confidence level for operating system identification. |
| **operating_system_method** | `operating-system-method` | VARCHAR(150) | Detection method used (local_checks, ssh_fingerprint, banner). |
| **sinfp_signature** | `sinfp-signature` | VARCHAR(250) | Signature string from SINFP TCP/IP stack fingerprint plugin. |
| **Credentialed_Scan** | `Credentialed_Scan` | VARCHAR(15) | Whether host assessment was performed with valid credentials (true/false). |
| **LastAuthenticatedResults** | `LastAuthenticatedResults` | VARCHAR(50) | Whether last credentialed attempt on the host succeeded (true/false). |
| **local_checks_proto** | `local-checks-proto` | VARCHAR(15) | Protocol used for local checks (SMB, SSH, WMI). |
| **smb_login_used** | `smb-login-used` | VARCHAR(150) | Username that successfully authenticated for SMB local checks. |
| **wmi_domain** | `wmi-domain` | VARCHAR(150) | Windows domain name used for WMI credentialed checks. |
| **policy_used** | `policy-used` | VARCHAR(150) | Name of the scan policy/template executed. |
| **patch_summary_total_cves** | `patch-summary-total-cves` | VARCHAR(150) | Number of CVEs addressed by patches (from patch summary plugins). |
| **host_start** | `HOST_START` | VARCHAR(150) | Human-readable timestamp when scanning of the host began. |
| **host_start_timestamp** | `HOST_START_TIMESTAMP` | INT | Unix epoch (seconds) of host_start. |
| **host_end** | `HOST_END` | VARCHAR(150) | Human-readable timestamp when scanning of the host finished. |
| **host_end_timestamp** | `HOST_END_TIMESTAMP` | INT | Unix epoch (seconds) of host_end. |

### Report Item Attributes

Extracted from `ReportItem` XML attributes. Present on every finding.

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **pluginID** | `pluginID` | INT | Numeric identifier of the Nessus plugin. |
| **pluginName** | `pluginName` | VARCHAR(MAX) | Human-readable name/title of the plugin. |
| **pluginFamily** | `pluginFamily` | VARCHAR(150) | Plugin family category (e.g., Windows, Databases, CGI). |
| **port** | `port` | INT | Network port associated with the finding (0 for host-level issues). |
| **protocol** | `protocol` | VARCHAR(15) | Transport protocol (tcp, udp, icmp). |
| **svc_name** | `svc_name` | VARCHAR(150) | Service name associated with the open port (from nmap-services). |
| **severity** | `severity` | INT | Raw Nessus severity value (0-4). Not stored directly; used to compute severity_rating and severity_reversed. |

### Report Item Child Elements

Extracted from child elements within each `ReportItem`. Most are optional and may be empty for a given finding.

#### Vulnerability Details

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **synopsis** | `synopsis` | VARCHAR(255) | One-sentence summary of the issue. |
| **description** | `description` | VARCHAR(MAX) | Full textual description of the vulnerability or plugin purpose. |
| **solution** | `solution` | VARCHAR(MAX) | Vendor or Tenable remediation guidance. |
| **risk_factor** | `risk_factor` | VARCHAR(15) | Severity string (None, Low, Medium, High, Critical) from CVSS or Tenable override. |
| **plugin_output** | `plugin_output` | VARCHAR(MAX) | Verbose per-host output returned by the plugin (evidence, banners, file lists). |
| **plugin_type** | `plugin_type` | VARCHAR(15) | Plugin category: remote, local, compliance, patch-management. |
| **plugin_publication_date** | `plugin_publication_date` | DATE | Date Tenable originally published the plugin. |
| **plugin_modification_date** | `plugin_modification_date` | DATE | Date Tenable last modified the plugin in the feed. |
| **script_version** | `script_version` | VARCHAR(150) | Revision string of the plugin script. |
| **see_also** | `see_also` | VARCHAR(MAX) | URLs or document references for additional advisories or guides. |
| **age_of_vuln** | `age_of_vuln` | VARCHAR(100) | Range of days since the NVD published the vulnerability, as a bucket (Tenable's VPR docs list 0-7, 7-30, 30-60, 60-180, 180-365, 365-730 and more than 730 days). Not an exact day count. VPR key driver. |

#### CVSS v2 Scores

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cvss_base_score** | `cvss_base_score` | VARCHAR(100) | NVD CVSS v2 Base Score (0.0-10.0). |
| **cvss_vector** | `cvss_vector` | VARCHAR(150) | Complete CVSS v2 Base vector string (e.g., `AV:N/AC:L/...`). |
| **cvss_score_rationale** | `cvss_score_rationale` | VARCHAR(500) | Free-text explanation of Tenable's CVSS override, if applied. |
| **cvss_score_source** | `cvss_score_source` | VARCHAR(100) | Origin of the CVSS v2 score: NVD, Vendor, or Tenable. |
| **cvss_temporal_score** | `cvss_temporal_score` | VARCHAR(100) | CVSS v2 Temporal Score computed by Tenable. |
| **cvss_temporal_vector** | `cvss_temporal_vector` | VARCHAR(100) | Vector string for CVSS v2 Temporal metrics. |

#### CVSS v3 Scores

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cvss3_base_score** | `cvss3_base_score` | VARCHAR(100) | NVD CVSS v3.x Base Score. |
| **cvss3_vector** | `cvss3_vector` | VARCHAR(150) | CVSS v3.x Base vector string. |
| **cvss3_score_source** | `cvss3_score_source` | VARCHAR(100) | Origin of the CVSS v3 score. |
| **cvss3_temporal_score** | `cvss3_temporal_score` | VARCHAR(100) | Tenable-calculated CVSS v3.x Temporal Score. |
| **cvss3_temporal_vector** | `cvss3_temporal_vector` | VARCHAR(100) | Vector string for v3 Temporal metrics. |
| **cvssV3_impactScore** | `cvssV3_impactScore` | VARCHAR(100) | CVSS v3.x Impact sub-score from NVD. |
| **vpr_score** | `vpr_score` | VARCHAR(100) | Tenable Vulnerability Priority Rating (0.0-10.0) combining CVSS and threat intel. |

#### CVE and Security References

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cve** | `cve` | NVARCHAR(4000) | CVE identifier(s) tied to the plugin. Multiple CVEs are comma-separated. |
| **cwe** | `cwe` | VARCHAR(150) | Common Weakness Enumeration ID(s) (e.g., CWE-79). |
| **bid** | `bid` | VARCHAR(150) | SecurityFocus BugTraq identifier(s). |
| **edb_id** | `edb-id` | VARCHAR(150) | Exploit-DB reference number(s). |
| **cea_id** | `cea-id` | VARCHAR(150) | CERT/CC Vulnerability Note or CISA KEV catalogue identifier. |
| **cert** | `cert` | VARCHAR(150) | CERT vulnerability note number(s) (e.g., VU#12345). |
| **cisa_known_exploited** | `cisa-known-exploited` | VARCHAR(150) | Whether CVE is in CISA's Known Exploited Vulnerabilities catalogue (true/false). |
| **cisa_ncas** | `cisa-ncas` | VARCHAR(150) | CISA National Cyber Awareness System advisory reference. |
| **cisco_bug_id** | `cisco-bug-id` | VARCHAR(150) | Cisco PSIRT bug ID(s) (e.g., CSCvx12345). |
| **cisco_sa** | `cisco-sa` | VARCHAR(150) | Cisco Security Advisory ID. |
| **iava** | `iava` | VARCHAR(150) | DISA Information Assurance Vulnerability Alert ID. |
| **iavb** | `iavb` | VARCHAR(150) | DISA Information Assurance Vulnerability Bulletin ID. |
| **iavt** | `iavt` | VARCHAR(150) | DISA Information Assurance Technical Advisory ID. |
| **icsa** | `icsa` | VARCHAR(150) | ICS-CERT advisory identifier (OT/ICS devices). |
| **msft** | `msft` | VARCHAR(150) | Microsoft Security Advisory reference (MSFT-YYYY-NN). |
| **mskb** | `mskb` | VARCHAR(150) | Microsoft Knowledge Base article number. |
| **tra** | `tra` | VARCHAR(150) | Tenable Research Advisory reference. |
| **vmsa** | `vmsa` | VARCHAR(150) | VMware Security Advisory ID(s). |
| **secunia** | `secunia` | VARCHAR(150) | Secunia advisory reference number. |
| **stig_severity** | `stig_severity` | VARCHAR(100) | DISA STIG severity mapping (Cat I/II/III). |

#### Exploit Intelligence

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **exploit_available** | `exploit_available` | VARCHAR(15) | Whether public exploit code is available (true/false). |
| **exploitability_ease** | `exploitability_ease` | VARCHAR(150) | How easy the issue is to exploit. Tenable's Nessus docs list the values: Exploits are available, No exploit is required, No known exploits are available. |
| **exploit_code_maturity** | `exploit_code_maturity` | VARCHAR(150) | VPR driver: relative maturity of possible exploit code. Values: High, Functional, PoC, Unproven. |
| **exploit_framework_canvas** | `exploit_framework_canvas` | VARCHAR(15) | Exploit present in Immunity Canvas (true/false). |
| **canvas_package** | `canvas_package` | VARCHAR(150) | Name of the Canvas exploit module. |
| **exploit_framework_core** | `exploit_framework_core` | VARCHAR(15) | Exploit present in Core Impact (true/false). |
| **exploit_framework_metasploit** | `exploit_framework_metasploit` | VARCHAR(15) | Exploit present in Metasploit Framework (true/false). |
| **metasploit_name** | `metasploit_name` | VARCHAR(150) | Name of the associated Metasploit module(s). |
| **exploited_by_malware** | `exploited_by_malware` | VARCHAR(15) | Whether vulnerability is exploited in active malware (true/false). |
| **exploited_by_nessus** | `exploited_by_nessus` | VARCHAR(15) | Whether Nessus can exploit this issue itself (true/false). |
| **in_the_news** | `in_the_news` | VARCHAR(15) | Tenable threat-intel flag for recent mainstream media coverage (true/false). |

#### Threat Metrics

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **threat_recency** | `threat_recency` | VARCHAR(150) | VPR driver: bucket of days since a threat event was seen (Tenable's VPR docs list 0-7, 7-30, 30-120, 120-365 and more than 365 days). |
| **threat_intensity_last_28** | `threat_intensity_last_28` | VARCHAR(150) | VPR driver: relative intensity of recently observed threat events, as a bucket: Very Low, Low, Medium, High, Very High. Not a number. |
| **threat_sources_last_28** | `threat_sources_last_28` | VARCHAR(150) | VPR driver: the sources where threat events were seen in the last 28 days (for example social media or the dark web), as a list. Not a count. |
| **product_coverage** | `product_coverage` | VARCHAR(150) | VPR driver: relative number of unique products affected by the vulnerability, as a bucket: Low, Medium, High, Very High. Not a percentage. |
| **unsupported_by_vendor** | `unsupported_by_vendor` | VARCHAR(15) | Product is end-of-life and no longer receives patches (true/false). |

#### Asset and Inventory

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **cpe** | `cpe` | VARCHAR(1000) | Item-level Common Platform Enumeration string for the affected product. |
| **agent** | `agent` | VARCHAR(150) | UUID or hostname of the Nessus Agent that performed the scan. |
| **always_run** | `always_run` | VARCHAR(150) | Plugin runs on every scan regardless of credential/port status (true/false). |
| **asset_inventory** | `asset_inventory` | VARCHAR(150) | Plugin purpose is inventory enumeration, not vulnerability detection (true/false). |
| **asset_inventory_category** | `asset_inventory_category` | VARCHAR(150) | Class of inventory info: Software, Hardware, Service. |
| **asset_categories** | `asset_categories` | VARCHAR(150) | Tenable-defined business/technical categories (e.g., Server, Workstation). |
| **hardware_inventory** | `hardware_inventory` | VARCHAR(150) | Result pertains to hardware identification (true/false). |
| **generated_plugin** | `generated_plugin` | VARCHAR(150) | Plugin is auto-generated from feed data, not hand-authored (true/false). |
| **thorough_tests** | `thorough_tests` | VARCHAR(150) | Whether "Enable thorough tests" was on (true/false). |
| **os_identification** | `os_identification` | VARCHAR(150) | OS fingerprint information from item-level detection. |

#### Dates

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **patch_publication_date** | `patch_publication_date` | DATE | Date vendor released the official patch. |
| **vuln_publication_date** | `vuln_publication_date` | DATE | Original public disclosure date of the vulnerability. |

### Derived/Computed Fields

These fields are computed during extraction and do not exist directly in the .nessus XML.

| Field | XML name | SQL Type | Description |
|-------|----------|----------|-------------|
| **severity_rating** | (computed) | VARCHAR(15) | Human-readable severity: Critical, High, Medium, Low, Informational. Computed from raw `severity` attribute. |
| **severity_reversed** | (computed) | INT | Inverted severity (0=Critical, 4=Informational). Useful for ascending sort by severity. |
| **scan_date** | (computed) | DATE | Date extracted from `host_start` timestamp (YYYY-MM-DD). |
| **file_name** | (computed) | VARCHAR(255) | Basename of the source .nessus file. Added during extraction for traceability. Named `file_name` in the CSV and Excel output and `file_path` in the SQL table below (it still holds the basename, not a path). |

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

### Notes on the table

- **Columns:** 106, one per field above except `severity` (the raw value is not stored; `severity_rating` and `severity_reversed` replace it). The table has one `cpe` and one `os_identification` column. Both hold the item-level value, because that is what the companion scripts store; the host-level `cpe` tag is not kept.
- **Column name:** `file_path` is the SQL name for the `file_name` field, and it holds the file's basename.
- **No key or index.** One row is one finding on one host in one scan file. If you need a key, (`host_ip`, `pluginID`, `port`, `protocol`, `scan_date`, `file_path`) is a reasonable candidate, but it has not been tested against real scan data.
- **`synopsis` is VARCHAR(255)** to match the companion script. Tenable does not document a length limit, and I have no scan corpus to measure. If an insert fails with "String or binary data would be truncated", widen it to VARCHAR(MAX).
- **Provenance:** the field list is what the companion Tenable_Nessus_Scripts extract. It was not derived from a specific Nessus or Tenable version, and it was not checked against a real .nessus file.

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
