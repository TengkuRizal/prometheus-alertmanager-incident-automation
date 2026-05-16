# Interview Explanation

## Project Summary

This project demonstrates a production-style monitoring and incident response workflow using Prometheus, Alertmanager, Kubernetes, and GitHub Issues.

The goal is to detect host or service downtime, route the alert through Alertmanager, and simulate incident ticket creation using a SOAR or webhook workflow.

---

## What I Built

I created a Prometheus alert rule using `PrometheusRule` inside a Kubernetes cluster running kube-prometheus-stack.

The first implemented alert is `HostDown`, which fires when Prometheus cannot scrape `node_exporter` for more than 2 minutes.

The alert is then routed to Alertmanager, where it can be grouped, silenced, and forwarded to a webhook or SOAR workflow.

---

## Alert Flow

```text
Kubernetes Node / Host
        ↓
node_exporter
        ↓
Prometheus scrape target
        ↓
PrometheusRule: HostDown
        ↓
Alertmanager
        ↓
Webhook / SOAR workflow
        ↓
GitHub Issue incident ticket