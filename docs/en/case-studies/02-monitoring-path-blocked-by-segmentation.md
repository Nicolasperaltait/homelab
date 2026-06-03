# Case Study 02 - Monitoring Path Blocked by Segmentation

## Context

The lab separates services and infrastructure into functional zones. That segmentation is valuable, but monitoring sometimes requires carefully justified cross-zone traffic.

## Symptom

An observability path failed even though the monitored service was healthy. The issue was not the dashboard; it was the network path between monitoring and the infrastructure target.

## Decision

The resolution pattern was:

- keep the segmentation model intact
- allow only the specific monitoring flow required
- avoid opening an entire zone for convenience
- document the exception and why it exists
- validate that dashboards reflect real metrics again

## Validation

The public validation pattern is:

- verify the exporter or metric source is healthy
- verify the monitoring collector can reach it
- verify the dashboard is backed by fresh data
- document the exception as a deliberate flow

## Lesson learned

Segmentation is not about blocking everything. It is about making allowed flows explicit, minimal and explainable.
