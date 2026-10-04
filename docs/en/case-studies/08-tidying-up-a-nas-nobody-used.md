# Case 08 - Tidying Up a NAS Nobody Used

## Context

The lab has a virtualized NAS that receives the workstation backups. It worked: backups
ran every night and the metrics were green. But it had accumulated months of layers:

- accounts from a previous job that no longer applied;
- shares pointing to folders that **no longer existed**;
- scheduled tasks notifying a messaging channel that had been retired weeks earlier;
- network discovery services nobody used;
- dozens of pending updates;
- and network routes to the security zone that **vanished** every time the system
  restarted its network stack.

Above all, **it had no visible use for its owner**. It was a backup warehouse nobody
opened.

The session had two goals: **tidy it up under least privilege**, and **give it real use**
without compromising what it already did well.

## The working rule

Nothing was applied without a written plan with rollback. Each batch was a script that:

1. backs up what it is about to touch;
2. applies the change;
3. **tests the effect, not the action**, and where possible **tests that what must fail
   does fail**;
4. and if a check does not pass, stops and says where the backup is.

The AI assistant prepares and verifies in read-only mode; **every privileged change is run
by the operator**, with their password, in their terminal.

## 1. Routes that did not survive the network

Routes to the internal zones were created by a one-shot service at boot. It covered boot
and nothing else. An update to a base system library restarted the network manager and
wiped them; the SIEM agent on that host stayed disconnected for an hour and a half, and
the symptom showed up on **another** system.

**Fix:** routes move into the declarative network configuration, in a dedicated file the
appliance manager neither generates nor deletes.

**Test**, at three increasing levels of severity:

| Test | Result |
|---|---|
| Restart the network manager (what the update did) | routes back on their own |
| Redeploy networking from the appliance panel | file still there, routes present |
| **Reboot the machine** | routes back on their own |

If any failed, the script rolled back on its own.

## 2. Accounts by function

| Role | Shell | Privilege | Shares |
|---|---|---|---|
| Internal administrator | no | full | no |
| Human operator | yes, key-based | **only one with elevation** | deliberately excluded |
| NAS access | **no shell** | none | read/write |
| Read-only NAS access | no shell | none | read |
| AI agent | only from the jump VM | scoped read | no |

Two accounts were deleted **after** verifying they had no processes or files. One that
looked expendable **was kept**: two automated workstation tasks used it. Its shell was
removed, because those tasks do not need one.

**Criterion:** before retiring an identity, ask what else it opened. Before keeping it,
ask what it actually needs.

## 3. Hardening

- dead scheduled tasks **moved, not deleted**, and the old channel's config file moved
  **without reading it** (it could contain a secret);
- network discovery and unused services turned off;
- minimum file-sharing protocol raised to the modern version;
- access from outside home only through the remote-access mesh, **never exposed to the
  internet**;
- all updates applied with a **prior VM snapshot**, a reboot, and a post-check that looks
  at **connectivity**, not just services: routes, reachability of internal zones and the
  internet, and the SIEM agent connected **according to its own state**, not the service
  manager's;
- the snapshot was **deleted** once verified: on a copy-on-write virtual disk a forgotten
  snapshot retains blocks, and had already filled storage once;
- system auditing shipped to the SIEM, and audit rules made **immutable** until next boot;
- a new alert: **SIEM agent disconnected**. The metric existed; the rule did not.

## 4. Giving it a purpose: one folder, three doors

A single exchange folder was created with three ways in:

| Door | What for |
|---|---|
| Web explorer | upload and download from any device, at home or over the remote mesh |
| File synchronization | what is dropped on the PC shows up on the NAS and the phone |
| File share | as one more network drive |

**The web file explorer runs in a container that only mounts that folder.** Isolation was
tested with a witness file: the container sees it, and **does not see** any backup folder.

**And room was made before inviting use.** The folder lives on the same disk as the
backups, whose nightly job **does not run** if free space drops below a threshold. The
margin was about 6 %. The virtual disk was grown online, after checking there were no live
snapshots, and the margin went above 40 %.

## What went wrong, told plainly

### The check that failed on a correct change

The minimum-protocol check compared against a literal string. The tool reports the same
value with a version suffix. **The change was right; the check failed**, and cut short the
verification that followed, which had to be finished by hand.

### The open port that was not open

The web explorer's port was opened and from the network it timed out, while **from the host
itself it answered fine**. The container engine publishes the port by rewriting the
destination: the connection travels through the firewall's **forward** chain, not the
**input** chain. The added rule was never evaluated.

A good side effect: the explorer's admin account still had **the factory password**, and
because of that mistake it **was never reachable**. The lesson still stands: the right
order was to change the password **before** opening, and to test the port **from another
machine**.

### A tunnel the host's own hardening forbade

To change that password with the port closed, a tunnel over secure remote access was
proposed. **The same host's hardening forbids tunnels.** Check the effective configuration
before proposing a path.

### Synchronization cut by an invisible field

When the exchange folder was accepted on the workstation, **synchronization of all the
backups** started dropping every few seconds, for 17 minutes.

The cause: a template carried an encryption password on the entry for **the device
itself**. The GUI only shows the other devices' entries, so on screen everything looked
empty. It was found by reading the raw configuration through the API.

**An almost identical case was already documented** from weeks earlier. This time the trap
was one step beyond what the interface lets you see. The fix included the template, so the
next folder does not inherit it.

### A silence read as evidence

It was concluded that a device "was not even trying to connect" because the log showed
nothing. **The querying account had no permission to read that log**: the answer was going
to be empty no matter what.

**An empty output is only evidence if the same query can see something when something
happens.**

### The button that would have undone the hardening

At the end, the appliance panel showed "pending changes to apply". Applying them **would
have turned back on** a service that had been disabled by hand: the configuration manager
re-enables it unless its own configuration says otherwise.

**Fix:** the disable moved into the manager's configuration, and only then was it applied,
verifying the service stayed off.

## A week later: the purpose that wasn't

Eight days later, the exchange folder **was empty**. Nobody had used it. Three doors to
one folder were not what was needed.

The question changed from "what can we add to the NAS" to **"what real problem does its
owner have"**. The answer was concrete: *someone asks me for a document and I can't get
to it*, and *I want to go back to an old version*. The usual answers were deliberately
ruled out: photo gallery, movie server, and a full cloud suite.

### One network drive, the same everywhere

The fix was the simplest one: **mount the whole NAS as one more drive** on each device,
always through the NAS address inside the remote-access mesh. It is the same address at
home and away, and traffic is encrypted.

| Device | How |
|---|---|
| Desktop PC | network drive, as before |
| Linux laptop | automount: connects when the folder is opened, releases when idle |
| Phone | file manager with a network client |

No local copy: the owner does not work offline, and with a mount there is nothing to
sync. A new folder shows up on all three on its own.

**Old versions already existed.** The backup mirror had been keeping versions for
weeks; what was missing was **a way to reach them**: the file share hides folders whose
names start with a dot, and that is where they live. From the laptop and phone they show.

A full cloud suite (web, apps, share links) was compared and ruled out on cost: a
database, a cache, more memory and major upgrades to maintain, for two needs a network
drive already covered. The door stays open: it can be mounted later over the same folders.

### What had been wrong the week before

**The web explorer "over the mesh" had never worked.** On the first day it was tested
only from home. Over the mesh the connection hung while the host firewall rule was
correct: the **mesh policy** did not allow that port, and since the packet never
reaches the host, its firewall logs nothing. The phone's sync away from home had the
same problem. The permission was added **along with a test in the policy itself**,
which makes any future change that closes it again get rejected.

**Synced files did not show up on the file share.** They were on disk and not on the
network drive. The folder permissions and the mask the sync service creates files with
were fixed. It works now, **with the exact cause unconfirmed**: the first hypothesis did
not match the permissions the file actually had. It was recorded that way, without
inventing an explanation.

### Own mistakes, told plainly

- A text replacement containing a Windows path read the backslashes as escape sequences
  and **broke two documents**. They were repaired by building the backslash outside the
  replacement.
- A block meant for the laptop **did not say which machine to run it on**, and it was
  run on the PC.
- Another block used a path format the operator's terminal does not understand.

### What remains open, said plainly

- What the owner creates directly on the NAS **has no backup**: backups copy fixed paths
  that come from the PC.
- The laptop stores a credential with write access to the whole NAS and **its disk is not
  encrypted**. It moved to high priority.

## Patterns

1. **Persist in the configuration that rules.** A route added by hand, a service masked
   outside the manager: they last until the next network restart or the next "apply".
2. **Test from where the user tests.** A port is tested from another machine; a route, by
   actually rebooting.
3. **Make room before inviting use.** The new feature shared a disk with the backups;
   growing it first prevented the folder's success from blocking the nightly backup.
4. **The interface is not the configuration.** When screen and symptom disagree, read the
   raw configuration.
5. **An empty result is not a "no".** First verify the query can see.
6. **Usefulness is measured in use, not features.** A folder empty after a week says more
   than three working doors. Ask about the problem, not the tool.
7. **From the mesh there are two filters.** A port meant for remote use needs the host
   rule **and** the mesh permission, and is tested against the mesh address.

## Outcome

| Before | After |
|---|---|
| routes lost on every network restart | declarative routes, tested with a real reboot |
| accounts from a previous job | five identities, one function each |
| shares pointing to non-existent folders | one share with separate read/write and read-only |
| dozens of pending updates | zero, with verified reboot |
| SIEM agent down with no warning | dedicated alert |
| ~6 % margin over the backup threshold | over 40 % margin |
| no use for its owner | exchange folder with browser, sync and network share |
| exchange folder empty after a week | the whole NAS as a drive on PC, laptop and phone, with versions within reach |
