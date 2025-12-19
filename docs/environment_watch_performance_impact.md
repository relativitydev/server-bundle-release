# Environment Watch Performance Impact

## Overview

This document provides transparent information about the performance overhead Environment Watch introduces to standard Relativity workloads, based on comprehensive testing in a production-like environment.

## Performance Impact on Relativity Workloads

Environment Watch has been rigorously tested to ensure minimal impact on your Relativity operations. Here's what you can expect:

### Performance Results Summary

| Workload Category | Impact | Summary |
|------------------|--------|------------------|
| **Processing** | TBD |TBD|
| **Review (Conversion)** | **+5% faster** | Review operations saw a modest 5% improvement, providing slightly faster document conversion without any workflow disruption. |
| **Imaging & Production** | **Stable (±4%)** | Imaging and production performance remained stable, with changes within a ±4% range, resulting in no meaningful impact to customer workflows. |
| **Data Transfer** | TBD| TBD |

## Test Environment Specifications

### Server Configuration Summary

| Server Role | Quantity | Specs |
|-------------|----------|-------|
| Web Servers | 3 | Standard D8s v5 (8 vCPUs, 32 GiB RAM) |
| Core Agent Servers | 6 | Standard D8s v5 (8 vCPUs, 32 GiB RAM) |
| Processing Workers | 4 | Standard D16ls v6 (16 vCPUs, 32 GiB RAM) |
| Data Grid Servers | 3 | Standard D8s v4 (8 vCPUs, 32 GiB RAM) |
| SQL Primary | 1 | Standard D8as v5 (8 vcpus, 32 GiB RAM) |
| SQL Invariant | 1 | Standard DS13 v2 (8 vcpus, 56 GiB RAM) |
| SQL Distributed | 1 | Standard DS14-8 v2 (8 vcpus, 112 GiB RAM) |
| Analytics Server | 1 | Standard D8s v4 (8 vCPUs, 32 GiB RAM) |
| Conversion Agent | 1 | Standard D8s v5 (8 vCPUs, 32 GiB RAM) |
| DtSearch Agent | 1 | Standard D8ls v5 (8 vCPUs, 16 GiB RAM) |
| RabbitMQ Server | 1 | Standard D8ls v5 (8 vCPUs, 16 GiB RAM) |
| PDF Server | 1 | Standard D8s v5 (8 vCPUs, 32 GiB RAM) |

This comprehensive test environment, ranging from Small to Medium scale, mirrors typical production Relativity deployments and ensures our performance results are representative of real-world customer workloads.

## Conclusion(TBD)

