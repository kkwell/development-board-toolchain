# GUI Baseline 2026-04-14

This document records the current local development baseline that was verified before archival.

## App Identity

- bundle name: `DBT-Agent.app`
- in-app title: `Development Board Toolchain`

## Verified Boards

- `TaishanPi`
- `ColorEasyPICO2`

## Baseline Behavior

### TaishanPi

- overview reads status from local `dbt-agentd`
- device IP is clickable
- SSH status is clickable
- reboot uses job-based execution
- update / environment / plugin install flows are agent-driven

### ColorEasyPICO2

- overview reflects unified status output
- BOOTSEL and runtime are derived from unified `status --json`
- firmware page only keeps:
  - flash initial program
  - save flash
- save flash prompts for destination path
- initial UF2 path is not shown inline; it is available through hover help

## UI Cleanup Included In This Baseline

- removed deprecated right-corner hover indicator dots from clickable cards
- plugin list no longer auto-jumps back into detail from list view
- ColorEasyPICO2 no longer shows release/development workspace toggles
- SSH terminal launch no longer clears the terminal first
- notification detail view resets back to list when switching tabs

## Control Plane Baseline

- local GUI status should rely on local `dbt-agentd`
- underlying hardware state should come from unified `dbtctl status --json`

## Explicit Non-Goals

- this repository does not bundle runtime tools
- this repository does not include private release orchestration
- this repository is not the source of truth for `dbt-agentd` or `dbtctl`
