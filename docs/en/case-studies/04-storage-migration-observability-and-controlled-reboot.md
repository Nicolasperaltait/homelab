# Case Study 04 - Storage Migration, Observability and Controlled Reboot

## Context

A small production-style lab needed to recover storage capacity without breaking
automation, backups or observability. The environment had strong dependencies
between storage, DNS, dashboards, security services and virtual machine startup
order.

## Problem

The risk was not only moving data. The real operational risk was losing
continuity through one of these surfaces:

- stable paths used by automation;
- dashboards querying the wrong filesystem;
- security services with stale metrics;
- virtual machines without reliable autostart;
- untested dependency on the primary DNS service;
- hypervisor reboot without evidence of a real successful boot.

## Decision

The migration was treated as a controlled operational change:

- preserve the logical path used by dependent processes;
- keep rollback in place before retiring the previous storage;
- validate observability with specific metrics, not broad green states;
- order virtual machine autostart by dependency;
- test behavior with the primary DNS service down;
- confirm a real reboot through boot time and active kernel version.

## Validation

The public validation pattern was:

- confirm that the new storage is mounted at the expected path;
- verify that dependent services remain active;
- check that dashboards and metrics query the right resource;
- temporarily stop the primary DNS service and validate fallback;
- reboot the hypervisor and verify virtual machine autostart;
- confirm the active kernel, not only that the remote session disconnected.

## Lesson Learned

A storage change does not end when files have been copied. It ends when
automation, dashboards, critical services, rollback and post-reboot startup have
all been validated with evidence.

A practical rule also came out of the event: an SSH disconnect is not proof of a
reboot. Reboot is confirmed with boot time, uptime and active kernel version.

## Outcome

The environment remained operational, storage was reclaimed in a controlled way,
observability was corrected and automatic startup was validated. The previous
storage was not deleted immediately; it was retained as rollback through an
observation window.
