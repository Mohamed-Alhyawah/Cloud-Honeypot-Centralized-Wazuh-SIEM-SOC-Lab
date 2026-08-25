# 🛡️ Cloud Honeypot & Centralized Wazuh SIEM SOC Lab

A production-grade, distributed threat intelligence environment designed to capture, analyze, and visualize real-world SSH brute-force attacks and malware activity. This setup pairs an isolated Google Cloud Platform (GCP) honeypot with a local, self-hosted Wazuh SIEM stack linked over an encrypted zero-trust mesh network.

---

## 🗺️ Network Topology & Architecture

```mermaid
flowchart LR
    %% External Threat Layer
    subgraph Internet ["🌐 Public Internet"]
        Attacker["🥷 External Threat Actors / Botnets"]
    end

    %% Cloud Infrastructure
    subgraph GCP ["☁️ Google Cloud Platform (GCP VM)"]
        Cowrie["🍯 Cowrie Honeypot<br/>(Docker / Port 2222)"]
        WazuhAgent["🕵️ Wazuh Agent<br/>(Log Collector)"]
        
        Cowrie -->|"JSON Logs<br/>(/var/log/cowrie)"| WazuhAgent
    end

    %% Encrypted Transport
    subgraph Tunnel ["🔒 Zero-Trust Network"]
        Tailscale["⚡ Tailscale Mesh VPN<br/>(Encrypted WireGuard)" ]
    end

    %% Local SOC Stack
    subgraph LocalSOC ["🏠 Local Server (Ubuntu Linux)"]
        subgraph DockerStack ["🐳 Wazuh Docker Environment"]
            WazuhManager["⚙️ Wazuh Manager<br/>(Custom Detection Engine)"]
            Indexer["🔍 OpenSearch Indexer<br/>(Log Storage & Search)"]
            Dashboard["📊 Wazuh Dashboard<br/>(SOC Visualization UI)"]

            WazuhManager -->|"Ingested Alerts<br/>(Port 9200)"| Indexer
            Dashboard <-->|"API Queries<br/>(Port 5000 / 9200)"| Indexer
        end
    end

    %% Connections
    Attacker -->|"SSH Brute Force<br/>(TCP Port 2222)"| Cowrie
    WazuhAgent -->|"Encrypted Log Stream<br/>(TCP Port 1514)"| Tailscale
    Tailscale --> WazuhManager

    %% Subgraph Styling
    classDef cloud fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#01579b;
    classDef soc fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#1b5e20;
    classDef tunnel fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#e65100;
    classDef threat fill:#ffebee,stroke:#d32f2f,stroke-width:2px,color:#b71c1c;

    class GCP cloud;
    class LocalSOC soc;
    class Tunnel tunnel;
    class Internet threat;
