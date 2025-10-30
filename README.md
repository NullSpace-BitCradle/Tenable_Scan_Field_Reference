# Tenable Scan Field Reference

## Overview

This document lists the headers that appear in a Nessus‑generated CSV/XML export or in the Tenable Security Center / Vulnerability Management data stream, along with a concise description of each field and suggested data type. Use it as a quick lookup when parsing or analyzing scan data.

## Quick Start

- Extract the report (CSV or XML) from Tenable.
- Use the Field Reference table to map columns/tags to types.
- Normalize booleans and timestamps according to Conventions below.
- Keep the raw `plugin_output` separately if you need verbose evidence.

## Field Reference

| Field | Type | Description |
|---|---|---|
| **age_of_vuln** | INT | Number of days between the vulnerability’s original disclosure date (from the NVD) and the time of the scan. Typically used as a VPR key driver. |
| **agent** | VARCHAR | UUID or hostname of the Nessus Agent that performed the local scan. Empty for network‑based scanner results. |
| **always_run** | BOOLEAN | Boolean flag indicating that the plugin is executed on every scan regardless of credential status or port/service discovery. |
| **asset_categories** | VARCHAR | Comma‑separated list of Tenable‑defined business/technical categories (e.g., _Server_, _Workstation_, _Network Device_). Derived from tag rules in Tenable VM. |
| **asset_inventory** | BOOLEAN | Boolean/flag plugins whose purpose is inventory enumeration rather than vulnerability detection (software/hardware listing, configuration capture). |
| **asset_inventory_category** | VARCHAR | High‑level class of inventory information returned by the plugin (e.g., _Software_, _Hardware_, _Service_). |
| **bid** | INT | SecurityFocus BugTraq identifier(s) associated with the vulnerability (e.g., `BID‑12345`). |
| **canvas_package** | VARCHAR | Name of the exploit module in Immunity Canvas that can exploit this vulnerability. Present when an exploit exists. |
| **cea_id** | VARCHAR | CERT Coordination Center (CERT/CC) Vulnerability Note identifier, or CISA Known Exploited Vulnerabilities (KEV) Catalogue entry, when applicable. |
| **cert** | VARCHAR | CERT vulnerability note number(s) linked to the finding (e.g., `VU#12345`). |
| **cisa_known_exploited** | BOOLEAN | true/false flag indicating the CVE is in CISA’s Known Exploited Vulnerabilities catalogue. |
| **cisa_ncas** | VARCHAR | Link or reference number to a CISA National Cyber Awareness System (NCAS) advisory related to the finding. |
| **cisco_bug_id** | VARCHAR | Cisco PSIRT bug ID(s) if Cisco has assigned one (e.g., `CSCvx12345`). |
| **cisco_sa** | VARCHAR | Cisco Security Advisory ID associated with the CVE. |
| **cpe** | VARCHAR | Common Platform Enumeration string(s) that describe the affected product and version. |
| **Credentialed_Scan** | BOOLEAN | true/false — whether host assessment was performed with valid credentials (local checks). |
| **cve** | VARCHAR | CVE identifier(s) tied to the plugin (e.g., `CVE‑2024‑12345`). Multiple CVEs are pipe‑delimited. |
| **cvss_base_score** | FLOAT | NVD CVSS v2 Base Score (0.0–10.0). |
| **cvss_score_rationale** | NVARCHAR | Free‑text explanation of Tenable’s CVSS override, if applied. |
| **cvss_score_source** | VARCHAR | Origin of the CVSS v2 score displayed: NVD, Vendor, or Tenable (manual). |
| **cvss_temporal_score** | FLOAT | CVSS v2 Temporal Score computed by Tenable. |
| **cvss_temporal_vector** | VARCHAR | Vector string for the CVSS v2 Temporal metrics. |
| **cvss_vector** | VARCHAR | Complete CVSS v2 Base vector (e.g., `AV:N/AC:L/...`). |
| **cvss3_base_score** | FLOAT | NVD CVSS v3.x Base Score. |
| **cvss3_score_source** | VARCHAR | Origin of the CVSS v3 score (similar to cvss_score_source). |
| **cvss3_temporal_score** | FLOAT | Tenable‑calculated CVSS v3.x Temporal Score. |
| **cvss3_temporal_vector** | VARCHAR | Vector string for the v3 Temporal metrics. |
| **cvss3_vector** | VARCHAR | CVSS v3.x Base vector string. |
| **cvssV3_impactScore** | FLOAT | CVSS v3.x Impact sub‑score provided by NVD. |
| **cwe** | VARCHAR | Common Weakness Enumeration ID(s) (e.g., `CWE‑79`). |
| **d2_elliot_name** | VARCHAR | Name of the exploit module in the D2 Elliot framework relevant to the vulnerability. |
| **description** | NVARCHAR | Full textual description of the vulnerability or plugin purpose. |
| **edb_id** | INT | Exploit‑DB reference number(s) where proof‑of‑concept code exists. |
| **exploit_available** | BOOLEAN | true/false — Tenable’s assessment that public exploit code is available. |
| **exploit_code_maturity** | VARCHAR | VPR driver category (e.g., Unchecked, PoC, Functional, High). |
| **exploit_framework_canvas** | BOOLEAN | true/false — exploit present in Immunity Canvas. |
| **exploit_framework_core** | BOOLEAN | true/false — exploit present in Rapid7 Core Impact. |
| **exploit_framework_d2_elliot** | BOOLEAN | true/false — exploit present in D2 Elliot. |
| **exploit_framework_exploithub** | BOOLEAN | true/false — exploit present in ExploitHub marketplace. |
| **exploit_framework_metasploit** | BOOLEAN | true/false — exploit present in Metasploit Framework. |
| **exploitability_ease** | VARCHAR | Tenable’s qualitative rating of how easy exploitation is (No Known, Theoretical, PoC, Functional, High). |
| **exploited_by_malware** | BOOLEAN | true/false — whether the vulnerability is known to be exploited in active malware. |
| **exploited_by_nessus** | BOOLEAN | true/false — Nessus itself can exploit this issue to obtain more info (e.g., weak credentials). |
| **Family** | VARCHAR | Plugin family container tag in the XML report. |
| **FamilyItem** | VARCHAR | Repetition element that lists a single plugin family in XML. |
| **FamilyName** | VARCHAR | String name of the plugin family (e.g., Windows, CGI, Databases). |
| **FamilySelection** | VARCHAR | Scan policy setting indicating which families were enabled/disabled. |
| **file_path** | VARCHAR | Path of a detected file (seen in compliance / malware / inventory plugins). |
| **fullName** | VARCHAR | Fully‑qualified internal name of the plugin in Tenable’s feed (shortName plus ID). |
| **generated_plugin** | BOOLEAN | true/false — whether the plugin is auto‑generated from feed data rather than hand‑authored. |
| **hardware_inventory** | BOOLEAN | Indicates result pertains to hardware identification (serial, vendor, model). |
| **host_end** | DATETIME | Human‑readable timestamp when scanning of the host finished. |
| **host_end_timestamp** | INT | Unix epoch of host_end. |
| **host_fqdn** | VARCHAR | Fully‑qualified domain name of the target host, if resolved. |
| **host_ip** | VARCHAR | Primary IPv4/IPv6 address of the asset as scanned. |
| **host_rdns** | VARCHAR | Reverse DNS lookup result for host_ip. |
| **host_start** | DATETIME | Human‑readable timestamp when scanning of the host began. |
| **host_start_timestamp** | INT | Unix epoch of host_start. |
| **hostname** | VARCHAR | NetBIOS/short hostname gathered from the host itself (SMB, SSH, etc.). |
| **iava / iavb / iavt** | VARCHAR | DISA Information Assurance Vulnerability Alert/Bulletin/Technical Advisory IDs. |
| **icsa** | VARCHAR | ICS‑CERT advisory identifier, relevant for OT/ICS devices. |
| **in_the_news** | BOOLEAN | true/false — Tenable threat‑intelligence flag that issue has recent mainstream coverage. |
| **IndividualPluginSelection** | VARCHAR | Policy element listing a specific plugin toggled on/off. |
| **item** | VARCHAR | Generic repeating XML element wrapper (context‑specific in Nessus schema). |
| **LastAuthenticatedResults** | BOOLEAN | true/false — whether last credentialed attempt on the host succeeded. |
| **local_checks_proto** | VARCHAR | Protocol used for local checks (SMB, SSH, WMI). |
| **mac_address** | VARCHAR | MAC address discovered for the host; multiple addresses are pipe‑delimited. |
| **malware** | BOOLEAN | true/false — Tenable classification that plugin result corresponds to known malware. |
| **metasploit_name** | VARCHAR | Name of the associated Metasploit module(s). |
| **msft / mskb** | VARCHAR | Microsoft advisory (MSFT‑YYYY‑NN) or Knowledge Base article reference. |
| **name** | VARCHAR | Generic attribute for a variety of XML elements (policy name, preference name, etc.). |
| **NessusClientData_v2** | VARCHAR | Root element of the Nessus XML schema (version 2). |
| **netbios_name** | VARCHAR | NetBIOS host name obtained via SMB enumeration. |
| **operating_system** | VARCHAR | OS string determined by local checks, banners, or fingerprinting. |
| **operating_system_conf** | INT | Confidence score (0–100) for operating_system identification. |
| **operating_system_id** | INT | Tenable internal numeric identifier for the OS in the feed. |
| **operating_system_method** | VARCHAR | Detection method (local_checks, ssh_fingerprint, banner). |
| **os_identification** | NVARCHAR | Aggregated text summary of OS fingerprint information. |
| **patch_publication_date** | DATE | Date vendor released the official patch/remediation. |
| **patch_summary_total_cves** | INT | For Japanese vulnerability data (JVN) or Tenable patch summary plugins: number of CVEs addressed by the patch. |
| **plugin_modification_date** | DATE | Date Tenable last modified the plugin in the feed. |
| **plugin_output** | NVARCHAR | Verbose per‑host output returned by the plugin (evidence, banners, file lists). |
| **plugin_publication_date** | DATE | Date Tenable originally published the plugin. |
| **plugin_type** | VARCHAR | Plugin category such as remote, local, compliance, patch‑management. |
| **pluginFamily** | VARCHAR | Same as FamilyName but in CSV/JSON exports. |
| **pluginID / pluginId** | INT | Numeric identifier of the plugin (duplicate fields differ only in case). |
| **PluginItem** | VARCHAR | XML element that encloses a single plugin result. |
| **pluginName** | VARCHAR | Human‑readable name/title of the plugin. |
| **PluginsPreferences** | VARCHAR | XML section listing all plugin‑specific preference settings. |
| **policy_used** | VARCHAR | Name of the scan policy/template executed. |
| **policyComments** | VARCHAR | Free‑form comments saved within the policy. |
| **policyName** | VARCHAR | Name attribute within the policy XML. |
| **port** | INT | Network port number associated with the finding (0 for host‑level issues). |
| **preference / preferenceName** | VARCHAR | Policy option name within Preferences section. |
| **Preferences** | VARCHAR | XML block containing all policy preferences (global + plugin‑specific). |
| **preferenceType** | VARCHAR | Enumeration indicating preference scope (global, plugin, credential). |
| **preferenceValues** | VARCHAR | Serialized set of possible/selected values for the preference. |
| **product_coverage** | FLOAT | Percentage of vendor‑known vulnerable products covered by patches. Part of VPR drivers. |
| **protocol** | VARCHAR | Transport protocol (tcp, udp, icmp) linked to port. |
| **Report / ReportHost / ReportItem** | VARCHAR | Top‑level, host‑level, and item‑level containers in Nessus XML. |
| **risk_factor** | VARCHAR | Severity string (None, Low, Medium, High, Critical) mapped from CVSS or Tenable override. |
| **scan_date** | DATE | Date the scan finished (report generation time). |
| **script_version** | VARCHAR | Revision string of the plugin script. |
| **secunia** | VARCHAR | Secunia advisory reference numbers. |
| **see_also** | VARCHAR | URLs or document references for additional vendor advisories or best‑practice guides. |
| **selectedValue** | VARCHAR | Value chosen for a preference in the policy. |
| **ServerPreferences** | VARCHAR | XML section inside the report describing server‑side settings (proxy, advanced). |
| **severity_rating** | FLOAT | Numeric severity (0–10) or categorical rating assigned to the finding. |
| **severity_reversed** | FLOAT | Inverted severity scoring (useful for sorting ascending). |
| **sinfp_signature** | VARCHAR | Signature string from SINFP TCP/IP stack fingerprint plugin. |
| **smb_login_used** | VARCHAR | Username that successfully authenticated for SMB local checks. |
| **solution** | VARCHAR | Vendor or Tenable remediation guidance. |
| **Status** | VARCHAR | Status element in Nessus namespace (e.g., current, completed). |
| **stig_severity** | VARCHAR | DISA STIG severity mapping (cat I/II/III). |
| **svc_name** | VARCHAR | Service name associated with the open port (from nmap‑services). |
| **synopsis** | VARCHAR | One‑sentence summary of the issue. |
| **system_type** | VARCHAR | Asset role discovered (server, workstation, router, etc.). |
| **thorough_tests** | BOOLEAN | true/false — whether Enable thorough tests was enabled. |
| **threat_intensity_last_28** | INT | Aggregate metric of exploit activity (0–100) seen in last 28 days for this CVE. |
| **threat_recency** | VARCHAR | Ordinal bucket of how recently exploits were observed (Very Recent, Recent, etc.). |
| **threat_sources_last_28** | INT | Count of discrete threat sources (exploit kits, campaigns) in last 28 days. |
| **tra** | VARCHAR | Red Hat Bugzilla tracking advisory reference (RHSA / RHTA). |
| **unsupported_by_vendor** | BOOLEAN | true/false — product is end‑of‑life and no longer receives patches. |
| **virtual_mac_address** | VARCHAR | MAC address of the virtual adapter reported by virtualized hosts. |
| **vmsa** | VARCHAR | VMware Security Advisory ID(s). |
| **vpr_score** | FLOAT | Tenable Vulnerability Priority Rating (0.0–10.0) combining CVSS and threat intel. |
| **vuln_publication_date** | DATE | Original public disclosure date of the vulnerability. |
| **wmi_domain** | VARCHAR | Windows domain name used for WMI credentialed checks. |

## Data Types

- **VARCHAR**: short text values (IDs, names, enumerations)
- **NVARCHAR**: long or free‑text values (descriptions, outputs)
- **INT**: whole numbers (IDs, counts, epochs)
- **FLOAT**: decimal/scalar scores (CVSS, VPR)
- **DATE**: calendar dates (YYYY‑MM‑DD)
- **DATETIME**: date + time strings from reports
- **BOOLEAN**: true/false flags

## Conventions

- **Booleans**: Normalize to `true`/`false` (case‑insensitive). CSV may use `True/False`, `Yes/No`, or `0/1`.
- **Timestamps**: `host_start`/`host_end` are human‑readable; the corresponding `*_timestamp` is Unix epoch (seconds). Prefer epoch for joins/sorting.
- **Lists**: Some fields contain multiple values separated by pipes `|` (e.g., `cve`, `mac_address`). Split on `|` when normalizing.
- **IDs vs Text**: Use numeric types where values are guaranteed numeric (`pluginId`, `port`, `edb_id`) and text for mixed formats (`cisco_bug_id`, `cve`).
- **Severities**: `risk_factor` is categorical; `severity_rating`/`vpr_score` are numeric.
