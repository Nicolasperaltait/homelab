# Case 05 - AI Agent Access Under Least Privilege

## Context

An AI assistant took part in operating a small production-grade lab platform:
diagnosis, change preparation, configuration review and documentation. To do
that, it needed to reach the hosts.

The initial approach was the obvious one: one credential per host, with a narrow
permission list. It worked, and it had a problem that stays invisible until you
name it.

## Problem

**An agent whose credentials live on the operator's workstation is, in practice,
the operator.** Same disk, same origin, same reach. The distinction between "the
person did it" and "the assistant did it" rests on trust, not on design.

Three questions the previous model could not answer:

- How do you cut the agent's access **without cutting the operator's**?
- How does the evidence tell them apart?
- What happens if the agent's material is copied outside the controlled
  environment?

## Decision

Direct access was replaced by **a single path with a physical off switch**.

| Decision | Reason |
|---|---|
| A dedicated jump host, and no direct access | The agent enters through one place or not at all |
| The jump host **does not start on its own**: the operator powers it on | The switch lives on the hypervisor, outside the agent's reach |
| Credentials for the rest of the estate **live only inside** the jump host | If they are copied out, powering the host off stops meaning anything |
| Source restriction on every destination | Even if leaked, the credential is useless from elsewhere |
| **No port or agent forwarding** on the jump host | With forwarding, the credential is effectively back on the operator's workstation and the switch is pointless |
| An evidence-collection agent inside the jump host | Events reach the SIEM immediately, per account |
| **No emergency path** for the agent | Explicit decision: if the jump host is off, ask for it to be powered on, do not look for another route |

The fourth row is what holds the whole thing up. **Without it, the first three
are decorative.**

## Privileged reading without write privilege

The model had a practical cost: almost all of the assistant's useful work is
**reading** -effective configuration, service state, logs- and a permission list
scoped to specific commands produces constant round trips over harmless things.

This was solved with **a read-only wrapper**: it reads almost anything on the
host with privilege, and **denies the critical material** -the password file,
private keys, environment files, tunnel state, the password manager's database,
SIEM agent keys-.

Two details make it real:

- **It normalises the path before deciding**, so `..` and symbolic links do not
  get around it.
- **It inspects content**, not just names: a file with an innocent name that
  contains a private key is denied anyway.

And a design rule that is not obvious: **it is not implemented as "allow the read
command"**. Authorising a generic reader with privilege is equivalent to full
access, because it can read the password file. The wrapper is its own program
with its own deny list.

## The uncomfortable finding

Measuring the finished model surfaced something the documentation did not say:
**inside the jump host, the agents held unrestricted privilege.** Not the narrow
list they had everywhere else: full access.

The written rule said the assistant must not use it. And it did not.

**But a rule the agent follows voluntarily is not a control.**

The blast radius was bounded -leaving the jump host still uses limited
credentials, and the switch lives outside the machine- but there was a concrete
consequence: **the monitoring of that host could be disabled from inside it**,
and one agent could read the other's material.

It was corrected the same day. **The allowed-command list was not invented: it
came from reading the privileged invocations actually recorded.** The result:
almost all real use was reading -covered by the wrapper-, some queries had never
needed privilege at all, and two concrete actions remained.

**The unrestricted permission had never been needed.**

## Validation

Verified **by what now fails**, not by what works:

- a generic reader with privilege: **denied**
- a command interpreter with privilege: **denied**
- one agent reading the other's private material: **denied**
- the read-only wrapper: works
- access to every destination host: works

Before removing any permission, the replacement was installed and tested. **The
script that performs the change refuses to run if the replacement is not
working**, because access had already been lost twice in this environment by
removing a credential before its successor was proven.

## Result

| Before | After |
|---|---|
| Agent credentials on the operator's workstation | Only inside the jump host |
| Direct access to every host | One path, with a physical switch |
| Unrestricted privilege on the jump host | Narrow list derived from measured use |
| Agents indistinguishable from each other | Separated: neither reads the other's material |
| "The agent should not do X" | The agent **cannot** do X |

## Lessons

**A restriction that only holds when convenient is not a restriction.** During
this very work, measuring privileged use inside the jump host was required, and
the only available route was exactly the permission that was forbidden. It was
not used: the data was requested instead. **The restriction cost a real step, and
that is why it counts.**

**Size a permission by measuring, not by imagining.** The final list came from
reading what was actually invoked. Guessing would have left extra permissions
"just in case", or too few, silently breaking automation.

**The control belongs in the infrastructure, not in the agent's conduct.** The
difference between "must not" and "cannot" is the only one that survives a
mistake, a version change, or a different model.
