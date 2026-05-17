# The Bare Metal Houses · Land of the Machines

> Brief for a Claude Code agent extending `os_genealogy.html` with a second tab.
> Subject: a sister chart showing the operating systems that *live where the metal is* — hypervisors, container substrates, unikernels. The user-facing OSes ride on top of these. They are the older dynasty, the one most users never see.

---

## 1. Brief

The current artifact is a chart of *who runs on the machine for a human to use*. This adds a sister chart of *who decides who runs on the machine* — the layer below.

The argument the chart makes, visually: VM/370 (1972) sits in the existing Mainframe house already, but it is **also** the founding ancestor of an entire parallel dynasty that the existing chart does not show. That dynasty runs CP-40 → CP-67 → VM/370 → through a long quiet period → Disco at Stanford (1997) → VMware ESX (2001), and in parallel Xen at Cambridge UK (2003) → KVM (2007) → and out into the management-plane OSes that wrap them (Proxmox, XCP-ng, oVirt, SmartOS). The remarkable historical fact the chart will surface: virtualization as a discipline was invented in **one IBM building in Cambridge, Massachusetts in the late 1960s** and then re-invented from scratch in **Cambridge, UK in 2003** by an independent team. Same name, different Cambridge, three decades apart.

Alongside the hypervisor dynasty: the **container-substrate dynasty** (CoreOS lineage, plus the K8s-host OSes that followed), and a small **unikernel** clade where the application *is* the OS.

The chart is **roughly 45 entries**, smaller than the user-facing chart (110). It is a tighter, more focused dynasty.

---

## 2. Naming

Same dual-register pattern as before:

- **Display title**: *The Bare Metal Houses*
- **Subtitle / scholarly form**: *Land of the Machines*

Tab labels in the UI:

```
[ Genealogy ]   [ Bare Metal ]
  the user-facing       land of the machines
  operating systems
```

(The italic muted second line in each tab is optional polish; the tab labels alone work.)

When the Bare Metal tab is active, the page chrome updates:

- Page title: *The Bare Metal Houses · Land of the Machines · 1967–2026*
- House label area on the left: shows the three Bare Metal houses instead of the nine user-facing ones.

---

## 3. UX architecture — the tab system

### Tabs
Add a tab strip at the top of the chart, above the existing control bar. Two tabs: **Genealogy** (the current chart, default) and **Bare Metal**.

Switching tabs:
- Re-filters `OSES` by the new `chart` field (see §4) and re-runs `packHouses()` / `computeHouseGeometry()` / the SVG render pipeline against the active chart's `HOUSES` map.
- Resets the detail rail (closes it if open) and clears any current selection.
- Preserves the search box value — it filters across both tabs but only displays matches in the active tab.
- Preserves the house-filter and layer toggle state *per-tab*. Each tab has its own filter state.

### State model
A single `activeChart` string ('genealogy' | 'baremetal'). All filter/selection state hangs off it. URL-hash routing optional (`#chart=baremetal`) so users can deep-link — would be a nice touch.

### Layout
Same chart canvas dimensions and the same horizontal time axis (1966–2027). The chart canvas is reused; only the data and house metadata change. Visually the tab change should feel like *descending a layer* — see §6 for the recommended visual register.

### Layer toggles per tab
- **Genealogy tab** carries the *Free Peoples* toggle (from `MOVEMENTS_LAYER.md`).
- **Bare Metal tab** carries one new toggle: ***Shared Ancestors***. Default ON. When ON, render "ghosted" cross-chart entries (VM/370, FreeBSD, Solaris) at the top of the chart as semi-transparent reference bars; clicking one switches to the Genealogy tab focused on that entry. When OFF, hide them — the chart shows only Bare Metal natives.

---

## 4. Data model changes

### a. New top-level field on entries

```js
{
  // existing fields ...
  chart: 'genealogy' | 'baremetal',   // which chart this entry belongs to
  // existing OSES entries default to 'genealogy' if the field is absent
}
```

Migration: a one-time pass adds `chart: 'genealogy'` to all existing entries, or simply treat absence-of-field as `'genealogy'`. Recommend the latter — fewer diffs in the existing data block.

### b. Cross-chart parent references

Bare Metal entries can declare parents that live in the Genealogy chart. Example: Proxmox's parents include `debian` (which lives in the Genealogy chart's `linux` house). The rendering pipeline must handle this:

- When rendering the **Bare Metal tab**: parent refs that resolve to a `chart: 'genealogy'` entry are drawn as connections to *ghosted ancestor bars* in the Shared Ancestors zone at the top of the chart.
- When rendering the **Genealogy tab**: parent refs from `chart: 'baremetal'` entries are *not* drawn (the Genealogy chart shouldn't be cluttered with the bare-metal layer).

### c. New `HOUSES` map for Bare Metal

```js
const HOUSES_BAREMETAL = {
  hypervisor: {
    name: 'The Hypervisor House',
    short: 'HYP',
    color: '#a16207',           // bronze
    motto: 'Sovereigns of the metal · grantors of audience'
  },
  substrate: {
    name: 'The Substrate House',
    short: 'SUB',
    color: '#0e7490',           // deep steel-cyan
    motto: 'Hosts of the containers · no shell, no purpose but to host'
  },
  unicorn: {
    name: 'The Unicorns',
    short: 'UNI',
    color: '#7c3aed',           // violet
    motto: 'Where the application is the operating system'
  }
};

const HOUSE_ORDER_BAREMETAL = ['hypervisor', 'substrate', 'unicorn'];
```

If the agent decides to render Shared Ancestors as a fourth pseudo-house at the top of the chart, add:

```js
  ancestors: {
    name: 'Shared Ancestors',
    short: '∴',
    color: '#8a8373',           // muted, matches existing --faint
    motto: 'These live in the Genealogy chart; shown here for lineage'
  }
```

…and put it first in `HOUSE_ORDER_BAREMETAL`.

### d. Clade tags (sub-grouping within houses)

The Hypervisor House has internal clade structure worth surfacing. Use the same `clade` field already introduced in the Movements layer brief:

```js
const CLADES_BAREMETAL = {
  // within hypervisor house:
  'cambridge-line':   { name: 'The Cambridge Line',     short: 'CAM',  accent: '#fbbf24' },
  'commercial-stack': { name: 'The Commercial Stack',   short: 'COM',  accent: '#94a3b8' },
  'silicon-bound':    { name: 'The Silicon-Bound',      short: 'SIL',  accent: '#f87171' },

  // within substrate house:
  'coreos-line':      { name: 'The CoreOS Line',        short: 'CORE', accent: '#5eead4' },
  'vendor-substrate': { name: 'The Vendor Substrates',  short: 'VND',  accent: '#67e8f9' },
  'kubernetes-only':  { name: 'The Kubernetes-Only',    short: 'K8S',  accent: '#86efac' }
};
```

Cambridge-Line is the open-source lineage from IBM Cambridge MA → Cambridge UK Xen → KVM. Commercial-Stack is VMware + Microsoft. Silicon-Bound is the hyperscaler/hardware-coupled wave (Nitro, Apple Virtualization, Nutanix AHV).

Clade tags are display-only for the legend and detail panel; they do not affect lane packing.

---

## 5. The dataset

### 5.1 Shared Ancestors (cross-chart references)

These already exist in the Genealogy chart. Do **not** duplicate the entries. Instead, in the Bare Metal renderer, draw ghost-bars referencing these IDs. Recommended ghost-bar treatment: 35% opacity fill, dashed outline, "↗ Genealogy" badge on hover.

| Genealogy ID | Why it appears as a Bare Metal ancestor |
|---|---|
| `vm370` | The founding ancestor of all virtualization. Already in `mainframe`. |
| `freebsd` | Parent technology of `jails` (2000) — the first production container-like isolation. |
| `solaris` | Parent technology of Zones (2004) — the first commercial OS-level virtualization. |
| `linux` | Host kernel for KVM, LXC, cgroups. Cambridge-Line ancestor. |
| `fedora` | Base of Project Atomic, Fedora CoreOS, RHEL CoreOS. |
| `debian` | Base of Proxmox VE, SteamOS 1, several container substrates. |
| `arch` | Base of XCP-ng's optional dom0, SteamOS 3. |
| `suse` | Base of Harvester. |

### 5.2 Foundational technologies (Bare Metal natives, but tech-not-OS)

These are *technologies* rather than shipping operating systems. Render them as smaller-height bars (e.g. 14px vs 22px for the OS bars) to mark their distinct status. They have parents like OS entries do.

```js
{ id: 'cp40', name: 'CP-40', house: 'hypervisor', born: 1967, ended: 1972,
  parents: [], company: 'IBM Cambridge Scientific Center',
  chart: 'baremetal', clade: 'cambridge-line', kind: 'os',
  note: "The first virtual machine system. IBM Cambridge MA, on a modified S/360 Model 40. Direct ancestor of CP-67 and VM/370." }

{ id: 'cp67', name: 'CP-67 / CMS', house: 'hypervisor', born: 1968, ended: 1979,
  parents: [{ id: 'cp40', rel: 'derived' }],
  company: 'IBM Cambridge Scientific Center',
  chart: 'baremetal', clade: 'cambridge-line', kind: 'os',
  note: "Second-generation virtualization on the S/360 Model 67. Each user got a complete virtual S/360. Productised as VM/370 in 1972." }

{ id: 'chroot', name: 'chroot', house: 'substrate', born: 1979, ended: null,
  parents: [{ id: 'unixv7', rel: 'derived' }],
  company: 'Bell Labs',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "A Unix V7 syscall. The first filesystem-isolation primitive. Direct conceptual ancestor of every container — though it would be twenty years before anyone built containers around it." }

{ id: 'jails', name: 'FreeBSD jails', house: 'substrate', born: 2000, ended: null,
  parents: [{ id: 'freebsd', rel: 'derived' }, { id: 'chroot', rel: 'inspired' }],
  company: 'FreeBSD · Poul-Henning Kamp',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "First production-grade OS-level virtualization. PHK's design. Influenced everything downstream." }

{ id: 'linux-vserver', name: 'Linux-VServer', house: 'substrate', born: 2001, ended: 2018,
  parents: [{ id: 'linux', rel: 'derived' }, { id: 'jails', rel: 'inspired' }],
  company: 'community',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "First serious attempt at container-like isolation on Linux. Kernel patches; never mainlined. Mostly displaced by OpenVZ and then LXC." }

{ id: 'zones', name: 'Solaris Zones', house: 'substrate', born: 2004, ended: null,
  parents: [{ id: 'solaris', rel: 'derived' }, { id: 'jails', rel: 'inspired' }],
  company: 'Sun Microsystems',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "Sun's commercial OS-level virtualization. Shipped in Solaris 10. The Bryan Cantrill design — sophisticated, observable, the standard reference for years." }

{ id: 'openvz', name: 'OpenVZ', house: 'substrate', born: 2005, ended: null,
  parents: [{ id: 'linux', rel: 'derived' }, { id: 'linux-vserver', rel: 'inspired' }],
  company: 'SWsoft → Parallels → Virtuozzo',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "Production Linux containers before LXC existed. Patched kernel required. Still used in low-cost VPS hosting." }

{ id: 'cgroups', name: 'cgroups', house: 'substrate', born: 2008, ended: null,
  parents: [{ id: 'linux', rel: 'derived' }],
  company: 'Google · Paul Menage, Rohit Seth',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "Linux kernel resource accounting and limiting. Merged in 2.6.24. Originated at Google for their internal Borg system. Without cgroups, no Docker." }

{ id: 'lxc', name: 'LXC', house: 'substrate', born: 2008, ended: null,
  parents: [{ id: 'cgroups', rel: 'derived' }, { id: 'openvz', rel: 'inspired' }],
  company: 'IBM · later LinuxContainers.org',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "The first containers built on mainline kernel primitives (cgroups + namespaces). Daniel Lezcano and Serge Hallyn at IBM. The substrate Docker initially used." }

{ id: 'docker', name: 'Docker', house: 'substrate', born: 2013, ended: null,
  parents: [{ id: 'lxc', rel: 'derived' }],
  company: 'dotCloud → Docker Inc.',
  chart: 'baremetal', clade: 'coreos-line', kind: 'tech',
  note: "Not an OS. The image format, the tooling, the cultural moment that made containers ubiquitous. Initially shipped on LXC, later moved to its own libcontainer (now runc/containerd)." }
```

> **Schema note**: the `kind` field above is new — `'os'` (default) or `'tech'`. The renderer uses it to pick bar height. Adding this field to existing user-facing entries is unnecessary; absence defaults to `'os'`.

### 5.3 The Hypervisor House

#### 5.3.1 The Cambridge Line (open-source / Xen / KVM lineage)

```js
{ id: 'xen', name: 'Xen', house: 'hypervisor', born: 2003, ended: null,
  parents: [{ id: 'vm370', rel: 'inspired' }, { id: 'linux', rel: 'derived' }],
  company: 'University of Cambridge · Ian Pratt et al. → XenSource → Citrix → Linux Foundation',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Paravirtualization on x86. SOSP 2003 paper 'Xen and the Art of Virtualization' is the founding document. Originally required guest kernel modifications; later supported HVM. Now under the Linux Foundation." }

{ id: 'kvm', name: 'KVM', house: 'hypervisor', born: 2007, ended: null,
  parents: [{ id: 'linux', rel: 'derived' }, { id: 'xen', rel: 'inspired' }],
  company: 'Qumranet · Avi Kivity → Red Hat (acquired 2008)',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Kernel-based Virtual Machine. Avi Kivity's insight: with Intel VT-x and AMD-V, Linux itself can be a hypervisor — no separate kernel needed. Merged into Linux 2.6.20 (Feb 2007). Substrate of almost every modern Linux-based hypervisor." }

{ id: 'xenserver', name: 'XenServer · Citrix Hypervisor', house: 'hypervisor', born: 2007, ended: null,
  parents: [{ id: 'xen', rel: 'derived' }],
  company: 'XenSource → Citrix (acquired 2007)',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Commercial product wrapping Xen with a management plane. Branding flip-flopped between XenServer and Citrix Hypervisor. Open-sourced 2013." }

{ id: 'xcp-ng', name: 'XCP-ng', house: 'hypervisor', born: 2018, ended: null,
  parents: [{ id: 'xenserver', rel: 'forked' }],
  company: 'Vates · France',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Community fork of XenServer/Citrix Hypervisor after Citrix moved features behind paywalls. Vates maintains it from France. Free, no feature gates." }

{ id: 'proxmox', name: 'Proxmox VE', house: 'hypervisor', born: 2008, ended: null,
  parents: [{ id: 'debian', rel: 'derived' }, { id: 'kvm', rel: 'derived' }, { id: 'lxc', rel: 'derived' }],
  company: 'Proxmox Server Solutions · Vienna',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Debian + KVM + LXC + Perl-and-JavaScript web UI. The most consequential open-source hypervisor for small-to-mid datacenters and homelabs. Wraps two virtualization stacks at once." }

{ id: 'rhev', name: 'Red Hat Virtualization · RHEV', house: 'hypervisor', born: 2010, ended: 2024,
  parents: [{ id: 'kvm', rel: 'derived' }, { id: 'rhel', rel: 'derived' }],
  company: 'Red Hat',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Red Hat's commercial KVM management platform. Discontinued — Red Hat consolidated to OpenShift Virtualization (Kubernetes-managed VMs). End of an era." }

{ id: 'ovirt', name: 'oVirt', house: 'hypervisor', born: 2011, ended: 2024,
  parents: [{ id: 'kvm', rel: 'derived' }, { id: 'fedora', rel: 'derived' }],
  company: 'Red Hat (open source upstream)',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Open-source upstream of RHEV. Reached end-of-life alongside RHEV. Some community continuation efforts." }

{ id: 'harvester', name: 'Harvester', house: 'hypervisor', born: 2021, ended: null,
  parents: [{ id: 'suse', rel: 'derived' }, { id: 'kvm', rel: 'derived' }],
  company: 'SUSE · Rancher',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Hyperconverged infrastructure built on Kubernetes managing KubeVirt managing KVM. The most architecturally ambitious recent hypervisor — Kubernetes all the way down." }
```

#### 5.3.2 The Commercial Stack (VMware / Microsoft / proprietary lineage)

```js
{ id: 'disco', name: 'Disco', house: 'hypervisor', born: 1997, ended: 1999,
  parents: [{ id: 'vm370', rel: 'inspired' }],
  company: 'Stanford · Bugnion, Devine, Govil, Rosenblum',
  chart: 'baremetal', clade: 'commercial-stack', kind: 'tech',
  note: "Stanford research project that revived virtualization for commodity x86. The paper that founded VMware. Mendel Rosenblum and Edouard Bugnion left Stanford to co-found VMware in 1998." }

{ id: 'vmware-esx', name: 'VMware ESX', house: 'hypervisor', born: 2001, ended: 2010,
  parents: [{ id: 'disco', rel: 'derived' }],
  company: 'VMware',
  chart: 'baremetal', clade: 'commercial-stack',
  note: "First commercial bare-metal hypervisor for x86. Console OS was a stripped Linux. Discontinued in favor of ESXi (smaller, no console OS)." }

{ id: 'vmware-esxi', name: 'VMware ESXi', house: 'hypervisor', born: 2007, ended: null,
  parents: [{ id: 'vmware-esx', rel: 'derived' }],
  company: 'VMware · → Broadcom (acquired 2023)',
  chart: 'baremetal', clade: 'commercial-stack',
  note: "ESX without the Linux console — custom microkernel-style VMkernel. The dominant enterprise hypervisor for 15+ years. Broadcom's 2023 acquisition and licensing changes created the largest exodus in datacenter history." }

{ id: 'hyperv', name: 'Hyper-V', house: 'hypervisor', born: 2008, ended: null,
  parents: [{ id: 'vms', rel: 'inspired' }],
  company: 'Microsoft',
  chart: 'baremetal', clade: 'commercial-stack',
  note: "Microsoft's Type 1 hypervisor, shipped with Windows Server 2008. Architecturally similar to Xen (microkernel + parent partition). Also ships as the standalone Hyper-V Server." }
```

#### 5.3.3 The BSD/Illumos Branch

```js
{ id: 'bhyve', name: 'bhyve', house: 'hypervisor', born: 2011, ended: null,
  parents: [{ id: 'freebsd', rel: 'derived' }],
  company: 'NetApp → FreeBSD',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "FreeBSD's modern Type 2 hypervisor (counts as Type 1 in some configurations). Originally Peter Grehan at NetApp. Now substrate of FreeNAS/TrueNAS VM features and several appliances." }

{ id: 'smartos', name: 'SmartOS', house: 'hypervisor', born: 2011, ended: null,
  parents: [{ id: 'solaris', rel: 'derived' }, { id: 'kvm', rel: 'derived' }],
  company: 'Joyent · Bryan Cantrill et al. → Samsung',
  chart: 'baremetal', clade: 'cambridge-line',
  note: "Illumos (the OpenSolaris fork) with KVM ported in by Cantrill's team, plus Zones, ZFS, DTrace. Lives in memory, runs from USB, persists nothing locally. The most architecturally idiosyncratic hypervisor on the chart." }
```

#### 5.3.4 The Silicon-Bound (hyperscaler / hardware-coupled, mostly proprietary)

```js
{ id: 'nutanix-ahv', name: 'Nutanix AHV', house: 'hypervisor', born: 2015, ended: null,
  parents: [{ id: 'kvm', rel: 'derived' }, { id: 'centos', rel: 'derived' }],
  company: 'Nutanix',
  chart: 'baremetal', clade: 'silicon-bound',
  note: "KVM-based hypervisor packaged with Nutanix's hyperconverged platform. Sold as appliance, not standalone. The most successful KVM commercialization aimed at displacing VMware." }

{ id: 'nitro', name: 'AWS Nitro', house: 'hypervisor', born: 2017, ended: null,
  parents: [{ id: 'kvm', rel: 'derived' }],
  company: 'Amazon · Annapurna Labs',
  chart: 'baremetal', clade: 'silicon-bound',
  movementNote: "First deployed with the C5 instance family in late 2017. Built on KVM but with most of the hypervisor stack offloaded to custom Annapurna-designed silicon. AWS migrated all instances off Xen onto Nitro between 2017 and 2019.",
  note: "Lightweight KVM-derived hypervisor with the network/storage offloaded to dedicated Nitro Cards. The hypervisor that runs the world's largest cloud — and almost nobody outside AWS has ever logged into one." }

{ id: 'apple-virt', name: 'Apple Virtualization', house: 'hypervisor', born: 2020, ended: null,
  parents: [{ id: 'darwin', rel: 'derived' }],
  company: 'Apple',
  chart: 'baremetal', clade: 'silicon-bound',
  note: "Apple's hypervisor framework — Hypervisor.framework dates earlier, but the Apple Silicon era (2020+) made it a serious bare-metal proposition. Substrate of UTM, Parallels, VMware Fusion on M-series, and the official Linux/macOS VM tooling." }
```

### 5.4 The Substrate House

#### 5.4.1 The CoreOS Line (and its Red Hat continuation)

```js
{ id: 'coreos', name: 'CoreOS Container Linux', house: 'substrate', born: 2013, ended: 2020,
  parents: [{ id: 'chromeos', rel: 'inspired' }, { id: 'gentoo', rel: 'derived' }],
  company: 'CoreOS Inc. → Red Hat (acquired 2018) → IBM',
  chart: 'baremetal', clade: 'coreos-line',
  note: "The founding entry of the clade. Alex Polvi + Brandon Philips, 2013. Took the immutable-root idea from ChromeOS and built it for servers. Acquired by Red Hat in January 2018 ($250M). Discontinued May 2020 in favor of Fedora CoreOS." }

{ id: 'atomic', name: 'Project Atomic', house: 'substrate', born: 2014, ended: 2020,
  parents: [{ id: 'fedora', rel: 'derived' }],
  company: 'Red Hat',
  chart: 'baremetal', clade: 'coreos-line',
  note: "Red Hat's pre-acquisition answer to CoreOS. Used rpm-ostree, which became the foundation of Silverblue, Fedora CoreOS, and RHEL CoreOS. Discontinued 2020 — merged with CoreOS Container Linux into Fedora CoreOS." }

{ id: 'fedora-coreos', name: 'Fedora CoreOS', house: 'substrate', born: 2020, ended: null,
  parents: [{ id: 'coreos', rel: 'derived' }, { id: 'atomic', rel: 'derived' }],
  company: 'Red Hat · Fedora Project',
  chart: 'baremetal', clade: 'coreos-line',
  note: "The merger of CoreOS Container Linux and Project Atomic. Auto-updating, immutable, container-only. The community upstream of RHEL CoreOS." }

{ id: 'rhcos', name: 'RHEL CoreOS', house: 'substrate', born: 2018, ended: null,
  parents: [{ id: 'rhel', rel: 'derived' }, { id: 'atomic', rel: 'derived' }],
  company: 'Red Hat',
  chart: 'baremetal', clade: 'coreos-line',
  note: "The host OS underneath OpenShift. Not user-installable as a standalone product; ships only as part of OpenShift. The most-deployed substrate OS in enterprise Kubernetes." }

{ id: 'flatcar', name: 'Flatcar Container Linux', house: 'substrate', born: 2018, ended: null,
  parents: [{ id: 'coreos', rel: 'forked' }],
  company: 'Kinvolk → Microsoft (acquired 2021)',
  chart: 'baremetal', clade: 'coreos-line',
  note: "Friendly fork of CoreOS Container Linux, started when Red Hat's plans for CoreOS became uncertain. Kept the original architecture and update channels. Acquired by Microsoft in 2021 — and now lives inside Azure." }

{ id: 'rancheros', name: 'RancherOS', house: 'substrate', born: 2015, ended: 2022,
  parents: [{ id: 'coreos', rel: 'inspired' }],
  company: 'Rancher Labs → SUSE (acquired 2020)',
  chart: 'baremetal', clade: 'coreos-line',
  note: "Everything in containers — including PID 1. The most architecturally radical of the early substrate OSes. Discontinued in favor of k3OS and later Elemental." }
```

#### 5.4.2 The Vendor Substrates

```js
{ id: 'photonos', name: 'Photon OS', house: 'substrate', born: 2015, ended: null,
  parents: [{ id: 'centos', rel: 'inspired' }],
  company: 'VMware → Broadcom',
  chart: 'baremetal', clade: 'vendor-substrate',
  note: "VMware's minimal Linux for running containers on vSphere. Also serves as the appliance OS for some VMware management products. Cross-clade with the commercial-stack lineage." }

{ id: 'bottlerocket', name: 'Bottlerocket', house: 'substrate', born: 2020, ended: null,
  parents: [{ id: 'coreos', rel: 'inspired' }],
  company: 'Amazon Web Services',
  chart: 'baremetal', clade: 'vendor-substrate',
  note: "AWS's container OS. Rust-heavy, A/B partition updates, no shell by default (admin container accessed separately). Optimised for EKS/ECS; runs elsewhere but with strong AWS gravity." }
```

#### 5.4.3 The Kubernetes-Only

```js
{ id: 'talos', name: 'Talos Linux', house: 'substrate', born: 2018, ended: null,
  parents: [{ id: 'coreos', rel: 'inspired' }],
  company: 'Sidero Labs',
  chart: 'baremetal', clade: 'kubernetes-only',
  movementNote: "Pre-release 2018, stable releases following. 12 binaries total, no SSH, no shell — all management via gRPC API (talosctl). The most radical minimalism in the clade.",
  note: "Linux without the Linux operational pattern: no shell, no SSH, API-only. If you cannot do it via talosctl, you cannot do it. Built to run Kubernetes and nothing else." }

{ id: 'k3os', name: 'k3OS', house: 'substrate', born: 2019, ended: 2022,
  parents: [{ id: 'rancheros', rel: 'derived' }],
  company: 'Rancher Labs → SUSE',
  chart: 'baremetal', clade: 'kubernetes-only',
  note: "Substrate for K3s (lightweight Kubernetes). Discontinued. Replaced by Elemental in the SUSE lineup." }

{ id: 'kairos', name: 'Kairos', house: 'substrate', born: 2022, ended: null,
  parents: [{ id: 'k3os', rel: 'inspired' }],
  company: 'community · originally c3os',
  chart: 'baremetal', clade: 'kubernetes-only',
  note: "Immutable container-based OS focused on edge K3s/K8s deployments. The continuation of the k3OS idea after Rancher dropped it." }
```

### 5.5 The Unicorns (unikernels)

```js
{ id: 'mirageos', name: 'MirageOS', house: 'unicorn', born: 2013, ended: null,
  parents: [{ id: 'xen', rel: 'derived' }],
  company: 'University of Cambridge · OCaml Labs',
  chart: 'baremetal',
  note: "The reference unikernel: OCaml all the way down, Xen as substrate. Each application compiles into its own unikernel image with no OS underneath. Anil Madhavapeddy's project. The most academically rigorous unikernel." }

{ id: 'osv', name: 'OSv', house: 'unicorn', born: 2013, ended: null,
  parents: [],
  company: 'Cloudius Systems · → ScyllaDB',
  chart: 'baremetal',
  note: "Unikernel for unmodified Linux applications — runs a single JVM, single Node, or single anything in its own OS image. Project survived its founders' pivot to ScyllaDB." }

{ id: 'includeos', name: 'IncludeOS', house: 'unicorn', born: 2015, ended: null,
  parents: [],
  company: 'IncludeOS · Norway',
  chart: 'baremetal',
  note: "C++ unikernel — `#include <os>` and your code becomes a bootable image. The most syntactically charming entry in this house." }

{ id: 'hermit', name: 'HermitCore · Hermit', house: 'unicorn', born: 2016, ended: null,
  parents: [],
  company: 'RWTH Aachen · later Hermit Project',
  chart: 'baremetal',
  note: "HPC-focused unikernel, more recently rewritten in Rust as plain 'Hermit'. Used in some scientific computing contexts." }

{ id: 'unikraft', name: 'Unikraft', house: 'unicorn', born: 2017, ended: null,
  parents: [{ id: 'mirageos', rel: 'inspired' }],
  company: 'EU H2020 research project → Unikraft.io',
  chart: 'baremetal',
  note: "Build-system-driven unikernel framework — compose your unikernel from a library catalogue. The most commercially serious unikernel effort." }

{ id: 'nanos', name: 'Nanos', house: 'unicorn', born: 2018, ended: null,
  parents: [],
  company: 'NanoVMs',
  chart: 'baremetal',
  note: "Unikernel optimized for running a single Linux ELF binary. Commercial focus. The most pragmatic of the unikernels — accept the Linux ABI rather than rewrite the world." }
```

---

## 6. Visual register

The user-facing chart's palette is warm-paper-dark: amber and teal accents on warm near-black. The Bare Metal chart should *feel a layer deeper* — denser, colder, more industrial. Suggested treatment:

- **House colours**: bronze (#a16207) for Hypervisor, deep steel-cyan (#0e7490) for Substrate, violet (#7c3aed) for Unicorn. Darker and more saturated than the user-facing palette.
- **Background**: same `var(--bg)` `#0c0a07` — keep the artifact's overall aesthetic continuity.
- **Bar treatment**: same rectangle shape, but slightly taller (24px vs 22px) to convey weight. Or *unchanged shape* with a 1px dark border at the bottom of the bar implying the metal underneath. Pick one. Recommend: unchanged shape, but **a subtle 2px serif tick on the left edge of each bar** — a typographic notch — that the user-facing chart doesn't have. This becomes the visual signature of the Bare Metal chart.
- **Lineage lines**: same Bézier curves as the user-facing chart, but with a slightly higher base opacity (1.0 vs 0.7) — the relationships in this chart are *load-bearing*, not narrative-illustrative.
- **Foundational technology bars** (entries with `kind: 'tech'`): half-height (14px), distinct fill (e.g. a hatched or striped pattern via SVG `<pattern>`). This visually distinguishes "this is a technology, not a shipping OS."
- **Ghosted ancestor bars** (cross-chart refs): 30% opacity, dashed outline, with a small "↗ Genealogy" badge on hover or persistent margin marker. Clicking them switches to the Genealogy tab and selects that entry.

The tab switch itself should be **noticeable but not flashy**. A 250ms cross-fade is enough. Avoid sliding panes or zoom — the chart canvas is the same canvas, only the data differs.

---

## 7. Implementation plan

Suggested order:

1. **Schema migration**: introduce the `chart` field. Default existing entries to `'genealogy'` (treat absence as that). Add the `kind` field (default `'os'`).
2. **HOUSES split**: rename the existing `HOUSES` to `HOUSES_GENEALOGY`, add `HOUSES_BAREMETAL`. Same for `HOUSE_ORDER`. Introduce an `activeChart` state variable, default `'genealogy'`.
3. **Render pipeline refactor**: parameterise `packHouses()`, `computeHouseGeometry()`, and the SVG render path on the active chart's houses and filtered entries. Existing behavior on Genealogy must be visually identical.
4. **Tab UI**: add tab strip above the existing control bar. Wire it to `activeChart`. Trigger a full re-render on switch.
5. **Add the Bare Metal dataset**: insert §5.2 through §5.5 entries. Verify dataset integrity (no missing parent refs, including cross-chart refs).
6. **Cross-chart ghost rendering**: when rendering Bare Metal, for any parent ref pointing to a `chart: 'genealogy'` entry not already shown, render a ghost bar in a Shared Ancestors lane at the top of the chart.
7. **Bare Metal visual treatment**: apply the palette and the `kind: 'tech'` half-height treatment.
8. **Per-tab layer toggles**: wire the Free Peoples toggle to Genealogy only; add the Shared Ancestors toggle to Bare Metal only.
9. **Cross-chart navigation**: clicking a ghost ancestor in Bare Metal switches to Genealogy and selects the canonical entry.
10. **Detail rail**: when displaying a Bare Metal entry, the panel should note its chart and clade. When displaying a Genealogy entry that has Bare Metal descendants (e.g. Debian → Proxmox), show those in a separate "Bare Metal descendants" section, distinct from the regular "Begat".

---

## 8. Open questions for Gerald

1. **Tab style**: prominent (like browser tabs at the top) or subtle (like sub-nav)? Recommend prominent — these are co-equal views.
2. **URL routing**: deep-link tabs via `#chart=baremetal`? Nice-to-have.
3. **Shared Ancestors as a fourth pseudo-house** (drawn at the top of the Bare Metal chart) **vs. inline ghosts** scattered among the houses? Recommend fourth pseudo-house — cleaner separation, easier to toggle off.
4. **The `kind: 'tech'` distinction** — worth surfacing? Or treat chroot/jails/zones/cgroups/LXC/Docker as full OS entries despite being technologies? Recommend keep the distinction; it teaches the reader something.
5. **Should `vm370` be visually duplicated** at the top of the Bare Metal chart (in its own bronze colour) **or** ghosted-from-genealogy (in muted grey with the `↗ Genealogy` indicator)? Recommend ghosted — it's one entry with two roles, not two entries.
6. **Clade colour scheme**: §4.d uses six clade accents. They should harmonise with the three house colours. Worth a pass.
7. **Hyper-V's parent**: I gave it `vms` (OpenVMS) as `inspired` — the architectural pedigree runs through the VMS team Microsoft hired in the 1980s (Dave Cutler et al.) and into the NT kernel, of which Hyper-V is essentially a microkernel-style refactoring. Defensible but arguable. Could also parent to NT directly.
8. **AWS Nitro's visibility**: it's a proprietary internal hypervisor most people will never see. Worth a full bar, or a footnote? Recommend full bar — the chart is partly an argument that the most consequential hypervisor of the 2020s is one nobody outside AWS has logged into.
9. **VMware's acquisition by Broadcom**: end-date ESXi? The product still ships. I left `ended: null` but noted the acquisition. The chart's house-stops-being-active rule for entries like RHEV could apply here within a few years.

---

## 9. Dead ends — things already ruled out

So the agent doesn't re-derive them:

- **No appliance/router/NAS OSes** (OpenWrt, pfSense, OPNsense, VyOS, TrueNAS, Unraid). These deserve their own chart — *The Specialists* or *The Appliance Houses* — not a forced fit into Bare Metal. Considered and explicitly deferred.
- **No embedded RTOSes** (VxWorks, FreeRTOS, Zephyr, μITRON). QNX is the edge case (already in Genealogy's `unix` house). Embedded is its own world with its own dynamics; potentially a fourth chart later.
- **No VMware Workstation, GSX Server, or other Type-2 / hosted hypervisors as standalone bars**. The chart is *Bare Metal*. Workstation is mentioned in ESX's note as the lineage origin but doesn't get a bar.
- **No OpenStack, vSphere, vCenter, or other management products that aren't OSes.** OpenStack runs *on* an OS, it doesn't *replace* the OS. Same for vCenter, Foreman, MAAS.
- **No Docker, Kubernetes, or container runtimes as full OS entries.** Docker gets a `kind: 'tech'` bar because it was historically pivotal; Kubernetes does not get a bar — it is an orchestrator, not an OS.
- **No Solaris-derived hypervisor lineage as a separate clade.** SmartOS exists, but the Illumos branch is small enough to fold into the Cambridge Line (via KVM port) rather than spawn its own clade.
- **No "pre-Disco quiet period" placeholder bar.** The 1972–1997 gap is meaningful — the chart shows it as actual empty space between VM/370 ending its dominance and Disco beginning the revival. Don't fill it with a dummy entry. The gap is the point.
- **Do not try to fold the Bare Metal entries into the existing Genealogy chart as a third layer toggle.** This would compress the genuinely separate dynastic story into a footnote. The tab structure is the correct primitive.
- **Do not assume Proxmox and ESXi are peers visually.** They are peers commercially, but Proxmox is a Debian-derivative management plane wrapping KVM, while ESXi is its own kernel. Their bars are both in the Hypervisor House but their parent lines tell very different stories.

---

## 10. Sources

**Foundational and Cambridge MA (mainframe-era virtualization)**
- IBM CP-40 history: https://en.wikipedia.org/wiki/CP/CMS — Cambridge Scientific Center, 1967
- CP-67 and VM/370 history: Melinda Varian's *VM and the VM Community: Past, Present, and Future* (SHARE 1997) is the canonical history. Available at http://www.leeandmelindavarian.com/Melinda/

**Stanford / VMware**
- Disco paper: "Disco: Running Commodity Operating Systems on Scalable Multiprocessors", Bugnion, Devine, Govil, Rosenblum, SOSP 1997. https://web.stanford.edu/~ouster/cgi-bin/papers/disco.pdf
- VMware history: https://www.vmware.com/about.html and Mendel Rosenblum interviews
- Broadcom acquisition (Nov 2023): widely covered; the operational fallout is the more interesting genealogical event

**Cambridge UK / Xen**
- Xen paper: "Xen and the Art of Virtualization", Barham et al., SOSP 2003. https://www.cl.cam.ac.uk/research/srg/netos/papers/2003-xensosp.pdf
- Xen Project home: https://xenproject.org/
- XenServer/XCP-ng split: https://xcp-ng.org/

**KVM**
- KVM merge: Linux kernel 2.6.20, February 2007. Avi Kivity, Qumranet (acquired by Red Hat in 2008).
- KVM history: https://www.linux-kvm.org/page/Main_Page

**Proxmox, oVirt, RHEV, Harvester**
- Proxmox VE: https://www.proxmox.com/en/proxmox-virtual-environment/overview — Vienna, founded 2005, VE product 2008
- oVirt: https://www.ovirt.org/ — discontinued late 2024
- Harvester: https://harvesterhci.io/

**Hyper-V**
- Microsoft documentation: https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/
- Released with Windows Server 2008 (June 2008)

**BSD / Illumos**
- bhyve: https://wiki.freebsd.org/bhyve
- SmartOS: https://smartos.org/ — Joyent, 2011; Bryan Cantrill's blog has the architectural origin story

**AWS Nitro**
- AWS announcement: https://aws.amazon.com/ec2/nitro/ — production with C5 family at re:Invent 2017
- Anthony Liguori's re:Invent 2017 talk is the primary technical source
- Annapurna Labs acquisition: January 2015 ($350M reported)

**Apple Virtualization**
- Apple developer docs: https://developer.apple.com/documentation/virtualization
- Hypervisor.framework predates Apple Silicon; the Virtualization framework on Apple Silicon (2020+) is the bare-metal-relevant successor

**Container substrate lineage**
- chroot: Unix V7, 1979 (Bell Labs)
- FreeBSD jails: 2000, Poul-Henning Kamp. Original paper: https://docs.freebsd.org/en/articles/jails/
- Solaris Zones: introduced in Solaris 10, 2004
- LXC: https://linuxcontainers.org/ — first release August 2008
- cgroups: merged Linux 2.6.24, January 2008. Originated at Google (Paul Menage, Rohit Seth)
- Docker: https://www.docker.com/ — public release March 2013

**CoreOS lineage**
- CoreOS Container Linux: https://en.wikipedia.org/wiki/Container_Linux — founded 2013 (Polvi, Philips); acquired by Red Hat January 2018; discontinued May 2020
- Project Atomic: https://www.projectatomic.io/ — Red Hat, 2014; merged into Fedora CoreOS 2020
- Fedora CoreOS: https://fedoraproject.org/coreos/ — merger product, 2020
- RHEL CoreOS: ships with OpenShift, 2018
- Flatcar: https://www.flatcar.org/ — Kinvolk 2018; acquired by Microsoft 2021
- Rancher OS: https://github.com/rancher/os — 2015; discontinued in favor of k3OS and later Elemental

**Vendor substrates**
- Photon OS: https://vmware.github.io/photon/ — VMware, 2015
- Bottlerocket: https://aws.amazon.com/bottlerocket/ — AWS, GA August 2020

**Kubernetes-only**
- Talos Linux: https://www.talos.dev/ — Sidero Labs, pre-release 2018, stable 1.0 mid-2022
- k3OS: https://github.com/rancher/k3os — Rancher, 2019; archived 2022
- Kairos: https://kairos.io/ — formerly c3os, 2022

**Unikernels**
- MirageOS: https://mirage.io/ — Cambridge OCaml Labs, 2013
- OSv: https://osv.io/ — Cloudius Systems, 2013
- IncludeOS: https://www.includeos.org/ — Norway, 2015
- Hermit: https://github.com/hermit-os/hermit-rs — RWTH Aachen, 2016
- Unikraft: https://unikraft.org/ — EU H2020 project, 2017
- Nanos: https://nanos.org/ — NanoVMs, 2018

---

## 11. Acceptance criteria

The Bare Metal tab is done when:

1. The artifact loads on the Genealogy tab and is **visually identical** to the pre-existing artifact.
2. Clicking the Bare Metal tab reveals a chart with three houses (Hypervisor, Substrate, Unicorn) plus a Shared Ancestors zone, roughly 45 bars total.
3. The chart spans the same time axis as Genealogy (1966–2027), with the earliest bar being CP-40 (1967) and the most recent being Kairos (2022) or similar.
4. Selecting Proxmox shows its bloodline correctly: Debian (ghosted from Genealogy) + KVM + LXC as parents; visible descendants none. The detail rail shows clade `cambridge-line`.
5. Selecting VM/370 from either tab works: from Genealogy it shows up in the Mainframe house; from Bare Metal it appears in Shared Ancestors with a "↗ Genealogy" indicator.
6. Switching tabs preserves search box content; resets selection.
7. The Shared Ancestors toggle on Bare Metal hides/shows the ghost-bars without reflowing the rest of the chart.
8. Foundational technologies (chroot, jails, cgroups, LXC, Docker) render visually distinct from OS entries (half-height bars or hatched fill).
9. The lineage from Disco → VMware ESX → ESXi is traceable on hover.
10. The Cambridge-MA → Cambridge-UK conceptual arc — VM/370 ghost-bar → Xen — is renderable as a single inspired-line bridging across decades.

---

*End of brief.*
