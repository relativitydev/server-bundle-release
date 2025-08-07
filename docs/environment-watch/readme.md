# Relativity Environment Watch Dashboard Post-Install Verification

![Post-Install Verification Banner](../../../resources/post-install-verification-images/Post-installation-verification.svg)

## Prerequisites

> [!IMPORTANT]
> After installation, wait 10–15 minutes before starting the verification process. This allows time for:
> - All system services to fully initialize and become available
> - Data collection agents to begin exporting telemetry to Elasticsearch
> - Dashboard visualizations to populate with accurate status information
> - Health indicators to show accurate statuses

## Overview

This documentation outlines the step-by-step procedures for verifying that the entire Relativity Environment Watch system is functioning properly after installation. The verification process confirms that the Elastic Stack cluster is healthy, all monitoring agents are exporting telemetry to Elasticsearch, and the integration with Relativity (e.g. Relativity Alerts) is working correctly.

> [!NOTE]
> All Kibana dashboards are designed and optimized for **1920x1080** screen resolution to ensure optimal viewing experience and proper layout formatting.

## Table of Contents

- **[Elastic Cluster Dashboard Post-Install Verification](post-install-verification/elasticsearch-cluster-health.md)**  
- **[Monitoring Agents Dashboard Post-Install Verification](post-install-verification/monitoring-agents.md)**  
- **[Alerts Post-Install Verification](post-install-verification/alerts.md)**
