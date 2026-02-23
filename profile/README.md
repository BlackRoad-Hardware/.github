<div align="center">
  <h1>🔧 BlackRoad Hardware</h1>
  <p><strong>Pi fleet. IoT. Edge AI. Your silicon, your rules.</strong></p>
  <p>
    <img src="https://img.shields.io/badge/Pi%20Fleet-4%20Nodes-FF1D6C?style=for-the-badge"/>
    <img src="https://img.shields.io/badge/Hailo--8-52%20TOPS-9C27B0?style=for-the-badge"/>
    <img src="https://img.shields.io/badge/Agents-30K%20Capacity-2979FF?style=for-the-badge"/>
  </p>
</div>

## What Lives Here

Raspberry Pi fleet management, IoT automation, and edge AI deployment.

## Fleet

| Node | IP | Role | Capacity |
|------|----|------|----------|
| octavia (Pi 5) | 192.168.4.38 | Primary AI | 22,500 agents |
| lucidia (Pi 5) | 192.168.4.81 | Secondary AI | 7,500 agents |
| blackroad-pi | 192.168.4.64 | CF Tunnel | Services |
| alice | 192.168.4.49 | Tertiary | Backup |

**Network**: WireGuard hub-and-spoke (10.8.0.0/24)  
**Lucidia spec**: Hailo-8 x2 (52 TOPS) + 1TB NVMe

---
*© BlackRoad OS, Inc. All rights reserved.*