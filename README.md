# sentinel-incident-enricher

> Automates the first ten minutes of incident triage: pull Microsoft Sentinel incidents, extract indicators, check them against threat intelligence, and rank incidents by real risk.

**Status:** 🚧 In progress. Built step by step while learning Python; see the roadmap below.

## Why

In a SOC, analysts copy IPs, domains and hashes out of every incident and look them up by hand in AbuseIPDB and VirusTotal before deciding what to escalate. This tool does that lookup automatically and hands the analyst a ranked list.

## How it works

```
Sentinel (KQL) ──┐
                 ├──> IOC extraction ──> Threat intel (AbuseIPDB, VirusTotal) ──> Scoring ──> Markdown + CSV report
Sample JSON ─────┘
```

## Quick start (sample mode, no Azure account needed)

```bash
pip install -r requirements.txt
cp .env.example .env        # add your free AbuseIPDB and VirusTotal API keys
python -m enricher.cli --mode sample --input sample_data/incidents.json
```

## Live mode

Runs this KQL against your own Log Analytics workspace:

```kql
SecurityIncident
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by IncidentNumber
| project IncidentNumber, Title, Severity, Status, Description, AlertIds
```

Authentication uses an Entra ID app registration with **read-only** access (Log Analytics Reader). No secrets are stored in this repo.

## Roadmap

- [ ] Week 1: read sample incidents, extract IPs, domains and SHA256 hashes (`ioc.py`), first unit tests
- [ ] Week 2: AbuseIPDB and VirusTotal lookups with caching and rate-limit handling
- [ ] Week 3: live Sentinel connector (`azure-identity`, `azure-monitor-query`)
- [ ] Week 4: scoring, Markdown/CSV report, CI, screenshots
- [ ] Later: Teams webhook alerts, MITRE ATT&CK tactic mapping

## Security considerations

- Least-privilege, read-only access to the workspace
- API keys loaded from environment variables, never committed
- Respects free-tier rate limits of each threat intel provider

## Sample data

`sample_data/incidents.json` contains fictional incidents. IP addresses use reserved documentation ranges (RFC 5737) and domains use `example.com`, so no real organisation's data is included.
