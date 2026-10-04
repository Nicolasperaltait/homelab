# Case 07 - Workstation Migration and Backups That Lied

## Context

The main workstation used to operate the infrastructure needed to be reinstallable. Nobody dared, because
nobody knew what would be lost.

The question "what do I lose if I reinstall" was answered by measuring, not by
remembering. The result was worse than expected:

- **most working repositories had no remote at all.** Months of history lived on
  a single disk;
- **the backup meant to cover it had been frozen for almost four months**, while
  monitoring reported it healthy every day.

The goal was set as: reinstalling must lose nothing, and rebuilding the machine
must follow a document, not anyone's memory.

## Finding 1 - The backup that measured the wrong age

The backup chain had five links. The first one, a scheduled task on the
workstation, pointed to a script that no longer existed. It failed every day,
silently.

The later links kept running: storage compressed the frozen mirror every night,
the hash verified, and the metric reported a backup a few hours old.

**The metric measured the age of the archive, not the age of its content.** It
proved that the last step of the chain had run, not that the backup held new
data.

Fix: the chain was replaced with direct synchronization, and a single new metric
was added: **the age of the newest file inside the backup**, alerting when it
falls behind.

## Finding 2 - The offsite copy that failed silently

The offsite copy failed several days in a row without raising an alert.

The script used strict shell mode and an error trap. But **that trap does not fire
when the script itself ends with an explicit exit code**, which is exactly how it
aborted when a source folder was missing. The script reported failure to the
system, but the notification path never ran.

Fix: notification hooks into **process exit**, which always runs, and decides
based on the exit code. The optional folder no longer aborts the whole transfer.

## Finding 3 - The full disk that would not free space

Hypervisor storage reached one hundred percent. Deleting files inside the virtual
machine freed nothing.

The cause was a **forgotten snapshot**. While a snapshot exists, the virtual disk
keeps the old blocks: what is deleted inside still takes space outside.

Fix: retire expired snapshots and old backups, and only then propagate block
discard from the guest. Snapshots now get a retirement date when they are
created.

## The migration in phases

| Phase | What it did | Exit criterion |
|---|---|---|
| 0 | Save what had no copy, with key material encrypted separately | every archive **opened**, not just existed |
| 1 | Clean up, moving to quarantine instead of deleting | space measured and zero tasks pointing to missing paths |
| 2 | Self-hosted git remote | clone from another machine and compare history |
| 3 | Simple backup with a content metric | the metric alone confirms the backup is from today |
| 4 | Package the applications | deploy without manual steps |
| 5 | Real test on a clean machine | rebuild it reading only the document |

Two rules crossed every phase:

- **nothing is deleted directly.** Everything goes through a thirty-day
  quarantine with a source-to-destination manifest;
- **no destructive step runs unless its copy was verified by opening it.**

An example of why the second rule matters: a copy to quarantine was declared
complete and was missing thousands of files. It was caught because the deletion
step required a file-by-file comparison.

## The clean machine test

Phase 5 ran on a virtual machine built from scratch, **with the same two accounts
as the real machine**: an administrator and an unprivileged daily-use account.
That choice found defects a single-account machine would never have shown:

1. **A standard account cannot register scheduled tasks.** Restoration became
   three parts: administrator, user, then administrator again, registering the
   tasks on behalf of the daily account.
2. **A package installed from the store exists only for whoever installed it.**
   The daily account could not see the tool its own tasks needed. The classic
   installer is now forced, and the script looks for the tool in a path valid
   for any account.
3. **Restoration ran in the wrong profile.** The script now checks the account
   before starting and stops if it is not the right one.
4. **An installer failed once, transiently.** It is retried once before being
   declared failed.
5. **A bug of our own in the first-boot script** meant the package installer
   never ran, without showing any error. An explicit check was added at the end.

## The nightly backup with open files

Part of the configuration to back up belongs to tools that **never close**. The
first version silently accepted locked files and would, some day, have produced a
backup missing exactly what mattered.

Criterion, set by the machine's owner: **losing a few hours of work is
acceptable; restoring something months old while believing it is from yesterday
is not.**

- open files are read in shared mode;
- included databases are checked **after extracting them from the backup**, not
  at the source;
- if anything is incomplete, the archive is marked incomplete and **does not
  renew the success metric**. If that repeats, the alert fires on its own.

## Own remote: archiving is not deleting

With the self-hosted remote running, repositories were cleaned up:

- **only** one repository was deleted, one whose content was fully contained in
  another, **verified commit by commit** and not by similar names;
- repositories of retired projects were **archived**, not deleted. Their local
  copies were in quarantine: once it expired, the repository would be the only
  copy of that history. Deleting them freed no meaningful space and could not be
  undone.

A remote in the same house **is not an offsite copy**. That is why the remote is
included in the hypervisor backups, and a copy to external media remains pending.

## Common pattern

All three findings share the lesson of [Case 06](06-when-a-control-does-not-measure-what-it-claims.md):
**the success signal came from the wrong step.**

| Control | What it measured | What it had to measure |
|---|---|---|
| Backup metric | archive age | content age |
| Offsite alert | errors of one command | outcome of the whole process |
| Free space after deleting | what was deleted in the guest | what was freed on the host |

## Lesson

**A migration plan that was never tested is a hypothesis.** The only
authorization to reinstall is a machine that was never yours, rebuilt entirely
from the document. And a backup is only proven when someone opened what it
contains.
