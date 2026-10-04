# Case Study 04 - Storage Migration, Observability and Controlled Reboot

## Context

A small production infrastructure needed to recover storage capacity without breaking
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
- retire local rollback only after separate confirmation and functional tests;
- reuse the freed disk with a clear role, separating virtual machine disks,
  backups, installation images and NAS storage;
- validate observability with specific metrics, not broad green states;
- order virtual machine autostart by dependency;
- test behavior with the primary DNS service down;
- confirm a real reboot through boot time and active kernel version.

## Validation

The public validation pattern was:

- confirm that the new storage is mounted at the expected path;
- verify that dependent services remain active;
- check that dashboards and metrics query the right resource;
- run critical automations ahead of schedule and validate recoverable artifacts;
- confirm that no active disks or snapshots still retain the previous storage;
- review base disks, Cloud-Init disks and snapshots before deleting files that
  appear duplicated;
- verify final usage by storage and confirm that virtual machines remain in the
  expected state;
- temporarily stop the primary DNS service and validate fallback;
- reboot the hypervisor and verify virtual machine autostart;
- confirm the active kernel, not only that the remote session disconnected.

## Lesson Learned

A storage change does not end when files have been copied. It ends when
automation, dashboards, critical services, rollback and post-reboot startup have
all been validated with evidence.

A practical rule also came out of the event: an SSH disconnect is not proof of a
reboot. Reboot is confirmed with boot time, uptime and active kernel version.

Another lesson was not to assume that two disk files are duplicates. A base disk
can be removed only after checking references and backing files; a small
Cloud-Init disk may be actively referenced and should be retained.

## Outcome

The environment remained operational, storage was reclaimed in a controlled way,
observability was corrected and automatic startup was validated. The previous
storage was first retained as rollback, then retired after functional validation
and separate confirmation.

The freed disk was then formatted and reused as the primary virtual machine
storage. Local storage was kept for minimal critical components, while backups,
installation images and NAS storage were separated into a support storage tier.
