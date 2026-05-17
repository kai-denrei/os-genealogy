# Open Source Movements — A Toggleable Layer

> Brief for a Claude Code agent extending `os_genealogy.html`.
> Subject: adding *The Free Peoples* — the layer of operating systems that refused vendor capture, sat outside the dynastic high table, or argued against the consensus from a position of code rather than market share.

---

## 1. Brief

The current chart depicts nine reigning houses and 110 operating systems. It is a chart of **who won**. This layer adds the projects that *lost on purpose*, *lost gracefully*, or are *still arguing*: the FSF orthodoxy, the Bell Labs apostates, the privacy purists, the anti-systemd schism, the mobile rebels, the from-scratch idealists, the microkernel holdouts, the immutable/functional wave, the market rebels, and a couple of eschatologists.

The layer must be **toggleable** (default OFF). When ON, ~45 new bars appear in their houses (with a distinct visual treatment), and several existing bars (already in the dataset) get a movement tag added that activates the same visual treatment. When OFF, the chart is exactly as it is today.

The layer is **a confederation, not a tenth house**. Members keep their natural house (Devuan stays in Linux, SerenityOS in altpc, etc.). The movement tag is orthogonal to house.

---

## 2. Naming

The toggle title has a dual register, mirroring the existing chart pattern (`Houses & Bloodlines` / `An Operating System Genealogy`):

- **Display title**: *The Free Peoples*
- **Subtitle / scholarly form**: *The Open Source Movements*

The display title carries the LotR/dynastic register and avoids calling the entries "rebels" (which would smuggle in a moral frame the rest of the chart doesn't have). The subtitle is the honest scholarly description.

In the toggle UI: render both. Suggested chip text:

```
○ The Free Peoples
  the open source movements
```

Where the second line is smaller, italic, muted-text colour. The toggle should sit visually adjacent to the house filter chips but be distinct — it is a layer, not a house filter.

---

## 3. Visual & interaction design

### When the layer is OFF (default)
Exactly the current chart. Movement-tagged bars and lines are hidden entirely (display: none on group, not opacity — they should not occupy lanes).

### When the layer is ON
- Movement-tagged bars appear with a **secondary visual treatment**: I suggest a paler fill (e.g. 35% opacity of the house colour), an outline stroke matching the house colour at 100%, and a small marginal glyph (a circle, a dot, or an italic letter) to mark the entry as movement. Reasoning: the layer should read as *added to* the main chart, not as competing with it.
- Lineage lines into/out of movement entries: use the layer's accent treatment — dashed-thin with a slightly different cap, or `stroke-dasharray: 4 4` against the main chart's `2 6` for "inspired" relations. Pick one consistent style.
- House lanes will need to repack to make room for newly visible entries. The existing `packHouses()` greedy lane-packer should handle this if movement entries are present in `OSES` and have a `movement` field; the toggle gates *visibility*, but the packer should plan around them so the chart doesn't reflow noticeably. Decision point: do you want stable lanes (pack with movement always included, hide bars when layer is off → leaves blank slots) or compact lanes (re-pack on toggle → chart height changes)? Recommend stable lanes — chart-height changes on toggle is jarring.
- Sub-clade colouring: the ten clades each get a faint marginal indicator. Optional and probably over-engineering; defer unless Gerald asks. If implemented: a single-pixel left border on the bar in the clade's accent colour, or a clade-letter glyph in the bar's left margin.

### Detail panel additions
When a movement-tagged OS is selected:
- Show a small movement badge under the name (`THE FREE PEOPLES · ${clade name}`).
- The "Begat" and "Begotten of" sections naturally pick up the new entries via the existing graph traversal.

### Search/filter
- Movement entries are excluded from search when the layer is OFF.
- Add a separate filter affordance: clicking the layer toggle while the option is depressed/right-clicked could enable "ONLY Free Peoples" mode. Optional — start with simple on/off.

---

## 4. Data model

Two changes to the data shape:

### a. Optional fields on existing `OS` entries

```js
{
  // ... existing fields ...
  movement: true,           // boolean — does this OS belong to the layer
  clade: 'fsf',             // string — which sub-clade. See clade list below.
  movementNote: '...'       // optional — note specifically about the movement context, distinct from the historical note. Use sparingly.
}
```

If `movement` is absent or false, behavior is unchanged.

### b. Clade enum

```js
const CLADES = {
  fsf:          { name: 'The Free Software Clade',     short: 'FSF',          accent: '#a3e635' },
  'bell-labs':  { name: 'The Bell Labs Apostates',     short: 'Bell Labs',    accent: '#fbbf24' },
  privacy:      { name: 'The Privacy Purists',         short: 'Privacy',      accent: '#f87171' },
  'anti-systemd': { name: 'The Anti-systemd Schism',   short: 'Anti-systemd', accent: '#fb923c' },
  'mobile-rebel': { name: 'The Mobile Rebels',         short: 'Mobile',       accent: '#f0abfc' },
  'from-scratch': { name: 'The From-Scratch Idealists',short: 'From scratch', accent: '#a78bfa' },
  microkernel:  { name: 'The Microkernel Holdouts',    short: 'Microkernel',  accent: '#67e8f9' },
  immutable:    { name: 'The Immutable Wave',          short: 'Immutable',    accent: '#5eead4' },
  'market-rebel': { name: 'The Market Rebels',         short: 'Market',       accent: '#c4b5fd' },
  eschaton:     { name: 'The Eschatologists',          short: 'Eschaton',     accent: '#8a8373' }
};
```

Accent colours are suggestions — adjust to taste against the existing dark palette. Keep them readable on `var(--bg)` `#0c0a07`.

---

## 5. The dataset

Below are the entries to add or tag. Format is the existing OS schema. Parents reference IDs that exist either in the current dataset or among these new entries. **Cross-reference every parent ID before inserting** — broken refs will silently fail the bloodline traversal.

> **Notation**: `→ [parent_id rel]` is shorthand for `parents: [{ id: 'parent_id', rel: 'rel' }]`. `rel` is one of `derived` (code descent), `inspired` (ideological/architectural influence, no code), `forked` (explicit fork, often hostile or schismatic).

### 5.1 Tag-only changes (existing entries get `movement: true` and a `clade`)

| id | clade | rationale |
|---|---|---|
| `plan9` | `bell-labs` | Already in unix house. The defining apostate entry. |
| `minix` | `microkernel` | Already in unix. Tanenbaum's microkernel teaching OS. |
| `freedos` | `from-scratch` | Already in dos. A community rewrite. Optional tag — weak case but defensible. |
| `reactos` | `from-scratch` | Already in altpc. Quintessential from-scratch project. |
| `haiku` | `from-scratch` | Already in altpc. BeOS continuation, all volunteer. |
| `firefoxos` | `mobile-rebel` | Already in mobile. Mozilla's mobile rebellion that Mozilla then killed. |
| `kaios` | `mobile-rebel` | Already in mobile. Firefox OS's unlikely afterlife. |
| `tizen` | `mobile-rebel` | Already in mobile. *Editorial call* — Samsung is corporate but Tizen descends ideologically from MeeGo. Borderline. |

### 5.2 New entries — The Free Software Clade (FSF orthodoxy)

```js
{ id: 'gnu', name: 'GNU', house: 'unix', born: 1983, ended: null,
  parents: [{ id: 'unixv7', rel: 'inspired' }],
  company: 'FSF · Stallman',
  movement: true, clade: 'fsf',
  note: "The 1983 manifesto: rebuild Unix as free software. Userland shipped — gcc, glibc, bash, coreutils became the substrate of every Linux distribution. The kernel did not." }

{ id: 'hurd', name: 'GNU Hurd', house: 'unix', born: 1990, ended: null,
  parents: [{ id: 'gnu', rel: 'derived' }, { id: 'mach', rel: 'inspired' }],
  company: 'FSF',
  movement: true, clade: 'fsf',
  note: "The unfinished prophecy. Pre-alpha for thirty-five years. The microkernel approach was right; the timing was not." }

{ id: 'trisquel', name: 'Trisquel', house: 'linux', born: 2004, ended: null,
  parents: [{ id: 'ubuntu', rel: 'derived' }],
  company: 'Trisquel Project · Galicia',
  movement: true, clade: 'fsf',
  note: "FSF-endorsed Ubuntu with the binary blobs scraped out. The most-installed of the FSF list." }

{ id: 'gnewsense', name: 'gNewSense', house: 'linux', born: 2006, ended: 2017,
  parents: [{ id: 'debian', rel: 'derived' }, { id: 'ubuntu', rel: 'derived' }],
  company: 'FSF · Shuttleworth (early funding)',
  movement: true, clade: 'fsf',
  note: "FSF-endorsed Debian/Ubuntu variant. Funded in part by Mark Shuttleworth — one of the few cases of a Linux founder funding the FSF-purist alternative to his own distro. Discontinued 2017." }

{ id: 'parabola', name: 'Parabola GNU/Linux-libre', house: 'linux', born: 2009, ended: null,
  parents: [{ id: 'arch', rel: 'derived' }],
  company: 'Parabola Project',
  movement: true, clade: 'fsf',
  note: "Arch with proprietary firmware removed and the Linux-libre kernel substituted. FSF-endorsed since 2011." }

{ id: 'hyperbola', name: 'Hyperbola', house: 'linux', born: 2017, ended: null,
  parents: [{ id: 'arch', rel: 'derived' }, { id: 'debian', rel: 'inspired' }],
  company: 'Hyperbola Project · Brazil',
  movement: true, clade: 'fsf',
  movementNote: "Began 2017 as a libre Arch derivative. In December 2019 announced it would cease to be Linux and become HyperbolaBSD, a hard fork of OpenBSD — rejecting Linux for HDCP/DRM, Rust, and systemd. The kernel transition is still in pre-alpha. Notable for being the only operating system to leave Linux for ideological reasons.",
  note: "FSF-endorsed since 2018. Now mid-migration from Linux to a libre fork of OpenBSD." }

{ id: 'guix', name: 'Guix System', house: 'linux', born: 2013, ended: null,
  parents: [{ id: 'gnu', rel: 'derived' }, { id: 'nixos', rel: 'inspired' }],
  company: 'GNU Project',
  movement: true, clade: 'fsf',
  note: "Functional, declarative, reproducible — Nix's ideas re-implemented in Guile Scheme under GNU's wing. The cleanest synthesis of FSF orthodoxy and the immutable wave." }

{ id: 'replicant', name: 'Replicant', house: 'mobile', born: 2010, ended: null,
  parents: [{ id: 'android', rel: 'derived' }],
  company: 'Replicant Project · FSF',
  movement: true, clade: 'fsf',
  note: "FSF's mobile flag-plant: Android with every blob removed. The trade-off is severe — most phones lose camera, modem, or Wi-Fi — and the project's reach is correspondingly small." }
```

### 5.3 New entries — The Bell Labs Apostates

```js
{ id: 'inferno', name: 'Inferno', house: 'unix', born: 1996, ended: null,
  parents: [{ id: 'plan9', rel: 'derived' }],
  company: 'Bell Labs · later Vita Nuova',
  movement: true, clade: 'bell-labs',
  note: "Plan 9's portable child: a virtual-machine OS (Limbo language, Dis VM) intended to run anywhere. Sold to Vita Nuova; later open-sourced." }

{ id: '9front', name: '9front', house: 'unix', born: 2011, ended: null,
  parents: [{ id: 'plan9', rel: 'forked' }],
  company: 'community',
  movement: true, clade: 'bell-labs',
  note: "The community fork that keeps Plan 9 alive after Bell Labs stopped. Active development, dadaist release notes, working modern hardware support." }
```

### 5.4 New entries — The Privacy Purists

```js
{ id: 'tails', name: 'Tails', house: 'linux', born: 2009, ended: null,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Tails Project · funded by Tor, EFF, others',
  movement: true, clade: 'privacy',
  note: "The Amnesic Incognito Live System. Boots from USB, routes through Tor, forgets everything on shutdown. Snowden's choice." }

{ id: 'qubes', name: 'Qubes OS', house: 'linux', born: 2012, ended: null,
  parents: [{ id: 'fedora', rel: 'derived' }],
  company: 'Invisible Things Lab · Rutkowska',
  movement: true, clade: 'privacy',
  note: "Security by compartmentalisation: each task in its own Xen-isolated VM. A genuinely original architecture, not a hardened distro. Joanna Rutkowska's design." }

{ id: 'whonix', name: 'Whonix', house: 'linux', born: 2012, ended: null,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Whonix Project',
  movement: true, clade: 'privacy',
  note: "Two VMs: one workstation, one Tor gateway. Network isolation enforced architecturally. Often run inside Qubes." }

{ id: 'subgraph', name: 'Subgraph OS', house: 'linux', born: 2014, ended: 2017,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Subgraph',
  movement: true, clade: 'privacy',
  note: "Hardened Debian with mandatory application sandboxing and Tor by default. Active development effectively stopped after 2017." }
```

### 5.5 New entries — The Anti-systemd Schism

```js
{ id: 'alpine', name: 'Alpine Linux', house: 'linux', born: 2005, ended: null,
  parents: [{ id: 'gentoo', rel: 'inspired' }],
  company: 'Alpine Linux Development Team',
  movement: true, clade: 'anti-systemd',
  note: "musl libc, busybox userland, OpenRC init, apk package manager. The container-image substrate of choice — small, fast, no glibc, no systemd. Originated from LEAF Linux." }

{ id: 'void', name: 'Void Linux', house: 'linux', born: 2008, ended: null,
  parents: [{ id: 'gentoo', rel: 'inspired' }],
  company: 'Void Linux project · originally Juan R. P.',
  movement: true, clade: 'anti-systemd',
  note: "Rolling release, runit init, xbps package manager. Written from scratch, not forked. The principled middle path between Arch and Slackware." }

{ id: 'devuan', name: 'Devuan', house: 'linux', born: 2014, ended: null,
  parents: [{ id: 'debian', rel: 'forked' }],
  company: 'Devuan Project · "Veteran Unix Admins"',
  movement: true, clade: 'anti-systemd',
  movementNote: "Forked from Debian in November 2014 in explicit protest at Debian's adoption of systemd as default init. The only major operating system in history forked over an init system. First stable release 'Jessie' April 2017.",
  note: "Debian without systemd. Default init is sysvinit; OpenRC and runit are supported alternatives." }

{ id: 'antix', name: 'antiX', house: 'linux', born: 2007, ended: null,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'antiX project · Greece',
  movement: true, clade: 'anti-systemd',
  note: "Lightweight, systemd-free Debian derivative aimed at older hardware. Predates the explicit anti-systemd movement but became part of it." }

{ id: 'artix', name: 'Artix Linux', house: 'linux', born: 2017, ended: null,
  parents: [{ id: 'arch', rel: 'forked' }],
  company: 'Artix project',
  movement: true, clade: 'anti-systemd',
  note: "Arch without systemd. Supports OpenRC, runit, s6, dinit. The Arch-side counterpart to Devuan." }
```

### 5.6 New entries — The Mobile Rebels

This clade is dense and *important*. The duopoly captured ~99% of phones; the resistance is real, fragmented, and partly corporate.

```js
{ id: 'maemo', name: 'Maemo', house: 'mobile', born: 2005, ended: 2010,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Nokia',
  movement: true, clade: 'mobile-rebel',
  note: "Nokia's Debian-based Linux for the 770/N800/N900 tablets. A genuine open Linux phone OS — and the one that produced the N900, still revered." }

{ id: 'meego', name: 'MeeGo', house: 'mobile', born: 2010, ended: 2012,
  parents: [{ id: 'maemo', rel: 'derived' }, { id: 'fedora', rel: 'derived' }],
  company: 'Nokia + Intel · Linux Foundation',
  movement: true, clade: 'mobile-rebel',
  note: "Maemo (Debian-based) merged with Intel's Moblin (Fedora-based). Killed when Nokia's Elop chose Windows Phone in 2011. One handset shipped, the N9 — perhaps the most-loved mobile OS that never had a second chance." }

{ id: 'sailfish', name: 'Sailfish OS', house: 'mobile', born: 2013, ended: null,
  parents: [{ id: 'meego', rel: 'derived' }],
  company: 'Jolla · Finland',
  movement: true, clade: 'mobile-rebel',
  note: "MeeGo's continuation by ex-Nokia engineers. Still shipping. Russian government adopted it for state phones in the late 2010s — a strange afterlife." }

{ id: 'ubuntu-touch', name: 'Ubuntu Touch', house: 'mobile', born: 2013, ended: 2017,
  parents: [{ id: 'ubuntu', rel: 'derived' }],
  company: 'Canonical',
  movement: true, clade: 'mobile-rebel',
  note: "Canonical's mobile bet on convergence — phone + desktop in one OS. Abandoned April 2017 along with the Unity 8 / Mir stack." }

{ id: 'ubports', name: 'UBports', house: 'mobile', born: 2017, ended: null,
  parents: [{ id: 'ubuntu-touch', rel: 'forked' }],
  company: 'UBports Foundation',
  movement: true, clade: 'mobile-rebel',
  note: "Community continuation of Ubuntu Touch after Canonical's exit. Active, releases regular, runs on PinePhone." }

{ id: 'postmarketos', name: 'postmarketOS', house: 'mobile', born: 2017, ended: null,
  parents: [{ id: 'alpine', rel: 'derived' }],
  company: 'postmarketOS community',
  movement: true, clade: 'mobile-rebel',
  note: "Alpine Linux for old phones. Goal: a real Linux distribution on mobile, with a ten-year support lifespan. Possibly the most idealistic project in this clade." }

{ id: 'cyanogenmod', name: 'CyanogenMod', house: 'mobile', born: 2009, ended: 2016,
  parents: [{ id: 'android', rel: 'forked' }],
  company: 'CyanogenMod community → Cyanogen Inc.',
  movement: true, clade: 'mobile-rebel',
  note: "The original de-Googled Android. Community fork that grew enormous, became a startup, imploded around 2016." }

{ id: 'lineageos', name: 'LineageOS', house: 'mobile', born: 2016, ended: null,
  parents: [{ id: 'cyanogenmod', rel: 'forked' }],
  company: 'LineageOS community',
  movement: true, clade: 'mobile-rebel',
  note: "CyanogenMod's community resurrection after Cyanogen Inc. collapsed. Most-used custom Android distribution." }

{ id: 'grapheneos', name: 'GrapheneOS', house: 'mobile', born: 2014, ended: null,
  parents: [{ id: 'android', rel: 'derived' }],
  company: 'GrapheneOS · Daniel Micay',
  movement: true, clade: 'mobile-rebel',
  movementNote: "Began 2014 as CopperheadOS; rebranded GrapheneOS in 2019 after a split between Daniel Micay and Copperhead Limited.",
  note: "Hardened, de-Googled Android focused on real security guarantees, not just feature removal. Pixel-only." }

{ id: 'calyxos', name: 'CalyxOS', house: 'mobile', born: 2018, ended: null,
  parents: [{ id: 'android', rel: 'derived' }],
  company: 'Calyx Institute',
  movement: true, clade: 'mobile-rebel',
  note: "Privacy-focused Android with microG for de-Googling, run by a non-profit. Less paranoid about hardware than GrapheneOS." }

{ id: 'eos-murena', name: '/e/OS · Murena', house: 'mobile', born: 2018, ended: null,
  parents: [{ id: 'lineageos', rel: 'derived' }],
  company: 'Murena · Gaël Duval',
  movement: true, clade: 'mobile-rebel',
  note: "Gaël Duval (Mandrake founder) re-enters the OS world with a fully de-Googled Android plus cloud replacement. Ships on actual phones." }

{ id: 'pureos', name: 'PureOS', house: 'linux', born: 2017, ended: null,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Purism',
  movement: true, clade: 'mobile-rebel',
  note: "FSF-endorsed Debian variant that ships on Purism's Librem laptops and the Librem 5 phone. Cross-clade with FSF and mobile-rebel." }
```

### 5.7 New entries — The From-Scratch Idealists

```js
{ id: 'temple-os', name: 'TempleOS', house: 'altpc', born: 2003, ended: 2013,
  parents: [],
  company: 'Terry A. Davis',
  movement: true, clade: 'from-scratch',
  note: "Singular case. Written entirely by one person over a decade as a 'third temple', with its own language (HolyC), 640×480 16-colour graphics, and a Bible random-walk oracle. Public domain. Davis died 2018; the OS remains." }

{ id: 'serenityos', name: 'SerenityOS', house: 'altpc', born: 2018, ended: null,
  parents: [{ id: 'unixv7', rel: 'inspired' }],
  company: 'Andreas Kling et al.',
  movement: true, clade: 'from-scratch',
  note: "A Unix-like OS written entirely from scratch — kernel, userland, browser (LibWeb → Ladybird), all of it — in modern C++. Explicit 'for the love of it' project. Has spawned an independent browser engine." }

{ id: 'redox', name: 'Redox OS', house: 'altpc', born: 2015, ended: null,
  parents: [{ id: 'plan9', rel: 'inspired' }, { id: 'minix', rel: 'inspired' }],
  company: 'Redox OS · Jeremy Soller',
  movement: true, clade: 'from-scratch',
  note: "Microkernel OS in Rust. Crosses with from-scratch and microkernel clades. The most credible candidate for a memory-safe operating system from below." }
```

### 5.8 New entries — The Microkernel Holdouts

```js
{ id: 'minix3', name: 'Minix 3', house: 'unix', born: 2005, ended: null,
  parents: [{ id: 'minix', rel: 'derived' }],
  company: 'Tanenbaum · Vrije Universiteit Amsterdam',
  movement: true, clade: 'microkernel',
  note: "Tanenbaum's microkernel comeback — production-targeted, not educational. Quietly the most-deployed OS in the world: Intel ships it as the Management Engine firmware on every modern x86 chip. A dark joke about the Tanenbaum–Torvalds debate." }

{ id: 'genode', name: 'Genode', house: 'altpc', born: 2008, ended: null,
  parents: [],
  company: 'Genode Labs · Germany',
  movement: true, clade: 'microkernel',
  note: "An OS framework that runs on a dozen different microkernels (L4, seL4, NOVA, Fiasco). Their Sculpt distribution is an actual usable desktop." }

{ id: 'sel4', name: 'seL4', house: 'unix', born: 2009, ended: null,
  parents: [],
  company: 'NICTA · UNSW · later Data61 / Trustworthy Systems',
  movement: true, clade: 'microkernel',
  note: "The first general-purpose microkernel with a complete mathematical proof of correctness. Used in real high-assurance systems, including some autonomous vehicles." }

{ id: 'helenos', name: 'HelenOS', house: 'altpc', born: 2001, ended: null,
  parents: [],
  company: 'HelenOS project',
  movement: true, clade: 'microkernel',
  note: "Multiserver microkernel OS started at Charles University, Prague. Quietly active for a quarter-century." }
```

### 5.9 New entries — The Immutable Wave

```js
{ id: 'nixos', name: 'NixOS', house: 'linux', born: 2003, ended: null,
  parents: [],
  company: 'Eelco Dolstra (PhD thesis) → NixOS Foundation',
  movement: true, clade: 'immutable',
  note: "Declarative, functional, reproducible. Origin of the modern immutable-OS argument. Born as Dolstra's 2003 PhD work; first stable release 2007. Now seeds an entire sub-movement." }

{ id: 'silverblue', name: 'Fedora Silverblue', house: 'linux', born: 2018, ended: null,
  parents: [{ id: 'fedora', rel: 'derived' }],
  company: 'Red Hat · Fedora Project',
  movement: true, clade: 'immutable',
  note: "Fedora's immutable variant using rpm-ostree and Flatpak/Toolbox. The corporate-blessed immutable Linux. Spawned the Universal Blue ecosystem." }

{ id: 'microos', name: 'openSUSE MicroOS', house: 'linux', born: 2018, ended: null,
  parents: [{ id: 'suse', rel: 'derived' }],
  company: 'SUSE · openSUSE Project',
  movement: true, clade: 'immutable',
  note: "SUSE's transactional-update immutable system. Aeon and Kalpa are its desktop variants. The German-engineering wing of immutability." }

{ id: 'vanilla-os', name: 'Vanilla OS', house: 'linux', born: 2022, ended: null,
  parents: [{ id: 'ubuntu', rel: 'derived' }, { id: 'debian', rel: 'derived' }],
  company: 'Vanilla OS · Mirko Brombin',
  movementNote: "First release 22.10 in December 2022, originally Ubuntu-based. Vanilla OS 2.0 'Orchid' (July 2024) switched the base to Debian Sid. Atomic A/B updates, apx sandboxed package manager.",
  movement: true, clade: 'immutable',
  note: "Immutable Linux for non-expert users. Notable for swapping its upstream base mid-project — Ubuntu out, Debian in." }

{ id: 'bluefin', name: 'Bluefin', house: 'linux', born: 2023, ended: null,
  parents: [{ id: 'silverblue', rel: 'derived' }],
  company: 'Universal Blue',
  movement: true, clade: 'immutable',
  note: "Universal Blue's developer-and-creator-targeted image. 'The Chromebook of Linux workstations.' Built atop Fedora Silverblue with bootc." }

{ id: 'bazzite', name: 'Bazzite', house: 'linux', born: 2023, ended: null,
  parents: [{ id: 'silverblue', rel: 'derived' }, { id: 'steamos3', rel: 'inspired' }],
  company: 'Universal Blue',
  movement: true, clade: 'immutable',
  movementNote: "First release November 2023, based on Fedora 38. The mineral 'bazzite' forms small blue crystals — naming consistent with Universal Blue.",
  note: "Universal Blue's gaming-handheld-and-desktop image. SteamOS-like experience, Fedora base. Cross-clade with market-rebel." }
```

### 5.10 New entries — The Market Rebels

```js
{ id: 'steamos1', name: 'SteamOS 1–2', house: 'linux', born: 2013, ended: 2019,
  parents: [{ id: 'debian', rel: 'derived' }],
  company: 'Valve',
  movement: true, clade: 'market-rebel',
  note: "Valve's first attempt: Debian-based, aimed at the Steam Machines initiative. The hardware initiative failed; the OS quietly continued." }

{ id: 'steamos3', name: 'SteamOS 3', house: 'linux', born: 2022, ended: null,
  parents: [{ id: 'arch', rel: 'derived' }],
  company: 'Valve',
  movement: true, clade: 'market-rebel',
  movementNote: "Released February 2022 with the Steam Deck. Switched base from Debian to Arch. Immutable A/B root filesystem.",
  note: "The Steam Deck's OS. Arch-based, immutable, KDE Plasma, Proton-mediated Windows game compatibility. The most consequential Linux desktop release since Ubuntu." }

{ id: 'holoiso', name: 'HoloISO', house: 'linux', born: 2022, ended: null,
  parents: [{ id: 'steamos3', rel: 'derived' }],
  company: 'community',
  movement: true, clade: 'market-rebel',
  note: "Community reimplementation packaging SteamOS 3 for non-Deck hardware. A bootleg-but-tolerated continuation." }

{ id: 'chimeraos', name: 'ChimeraOS', house: 'linux', born: 2019, ended: null,
  parents: [{ id: 'arch', rel: 'derived' }],
  company: 'ChimeraOS community',
  movement: true, clade: 'market-rebel',
  note: "Originally GamerOS. Arch-based, Steam-Big-Picture-first, predates the Steam Deck's SteamOS 3 with a similar goal." }

{ id: 'popos', name: 'Pop!_OS', house: 'linux', born: 2017, ended: null,
  parents: [{ id: 'ubuntu', rel: 'derived' }],
  company: 'System76',
  movement: true, clade: 'market-rebel',
  note: "System76's Ubuntu derivative, born when Canonical retreated from desktop. Auto-tiling, custom installer, COSMIC desktop (their from-scratch Rust DE)." }
```

### 5.11 New entries — The Eschatologists

```js
{ id: 'collapse-os', name: 'Collapse OS', house: 'altpc', born: 2019, ended: null,
  parents: [],
  company: 'Virgil Dupras',
  movement: true, clade: 'eschaton',
  note: "Designed to be bootstrapped on scavenged hardware after civilisational collapse. Forth, z80, minimal. Documentation as artefact." }

{ id: 'dusk-os', name: 'Dusk OS', house: 'altpc', born: 2022, ended: null,
  parents: [{ id: 'collapse-os', rel: 'derived' }],
  company: 'Virgil Dupras',
  movement: true, clade: 'eschaton',
  note: "Collapse OS's successor — same author, broader target (32-bit), still Forth-rooted. Intended as the 'dusk' OS for the long descent, not the post-apocalypse." }
```

---

## 6. Implementation plan

Suggested order, each step independently testable:

1. **Schema migration**: add `movement` and `clade` fields as optional. Run `applyFilters()` and the SVG render with movement entries present but layer state default-off. Confirm chart is visually identical to current.
2. **Add the data**: insert all entries from §5. Run the existing dataset validation (count IDs, check no missing parent refs, verify year range).
3. **Tagging pass**: add `movement: true` and `clade` to the existing entries in §5.1.
4. **Lane packing**: decide stable vs. compact (recommend stable). If stable: `packHouses()` should consider movement entries always present. Render the chart with the layer logically ON to confirm packing works.
5. **Toggle UI**: add the `The Free Peoples` chip to the control bar. Wire it to a `showMovements` boolean. Default false.
6. **Visual treatment**: when `showMovements === false`, hide movement bars and their incoming/outgoing lineage lines. When true, render with the secondary treatment (lower fill opacity, stroke-outlined). Lineage lines into/out of movement entries get a distinct dash pattern.
7. **Detail panel**: add the movement badge under the name when the selected OS has `movement: true`. The Begat / Begotten Of sections work unchanged.
8. **Clade legend**: add a small panel listing the ten clades with counts. Style-match the existing legend.
9. **Bloodline traversal**: the existing `bloodline()` function should already work — movement entries are just nodes in the graph. Test: select Devuan, confirm its ancestors include debian → unixv7 etc., and its descendants are correctly highlighted.

---

## 7. Open questions for Gerald

Decisions I left for you:

1. **Clade colours**: the ten suggested in §4 are placeholders. Worth a pass for visual coherence against the existing house palette.
2. **Sub-clade visual indicator**: small left-border on bars in the clade colour? Italic letter glyph? Margin dot? Or skip entirely and let the detail panel be the only place sub-clade is surfaced.
3. **Stable vs. compact lane packing on toggle.** Recommendation: stable (no chart-height reflow).
4. **Tizen tagging**: include in `mobile-rebel` or leave out? It is genealogically descended from MeeGo, but Samsung-corporate. I tagged it; defensible either way.
5. **FreeDOS tagging**: weak case for `from-scratch`. Could leave untagged.
6. **Cross-clade entries**: PureOS is in `mobile-rebel` but is also FSF-endorsed. Bazzite is `immutable` but also `market-rebel`. Redox is `from-scratch` and `microkernel`. Current schema uses a single `clade` string — should it be `clades: []`? My recommendation: keep it single, accept the editorial lossiness, surface the cross-clade fact in the `movementNote`.
7. **Should GNU and Hurd be in a *new* house** (`gnu` or `freepeoples`) rather than tucked into `unix`? Argument for: they are not derivative of unix in any code-descent sense, only ideological. Argument against: opening the houses to ideological membership undermines the genealogical premise of the chart. I put them in `unix` reluctantly. **This is the most architecturally consequential question** of the entire exercise.

---

## 8. Dead ends — things already ruled out

So you don't re-derive these:

- **Don't make "Movements" a tenth house.** It collapses the layer into the house metaphor and removes the toggle's reason for being. The whole point is *orthogonal* membership.
- **Don't name the layer "Rebels" or "Pretenders".** The first smuggles in a moral frame; the second is hostile. *The Free Peoples* / *The Open Source Movements* is what we settled on.
- **Don't try to charge the dashed/solid line distinction with extra meaning** for movement entries. The existing `derived` / `inspired` / `forked` rel system is sufficient. Adding a fourth relation type for "movement-influence" creates a combinatorial mess in the rendering.
- **Don't add Container OSes** (CoreOS, Flatcar, Bottlerocket, Talos). Considered and rejected — they are a different kind of rebel (infrastructural minimalism) and would visually clutter without paying their bar. Possible future *Server Rebels* sub-layer; not now.
- **Don't add educational/research OSes** (xv6, Pintos, Nachos, JOS). They are pedagogy, not movement.
- **Don't add penetration-testing distros** (Kali, Parrot, BlackArch). They are professional tools, not ideological projects.
- **Don't add AROS / MorphOS / AmigaOS 4** — they are nostalgia continuations and already exist in `altpc`. Tagging them as `from-scratch` would dilute that clade with what is really a *legacy continuation* category.
- **Don't add every NixOS-inspired immutable distro.** Six entries in §5.9 is enough; the rest are variations on a theme.
- **Don't try to fork the chart into "Movement view" vs "Houses view"** as separate pages. The layer is the right pattern — it shows both arguments in tension, which is the editorially interesting thing.

---

## 9. Sources

Authoritative references, organised by clade. URLs are starting points; the CLI agent should fetch and verify dates rather than trust this document.

**FSF clade**
- GNU Project home and history: https://www.gnu.org/gnu/gnu-history.html
- GNU Hurd: https://www.gnu.org/software/hurd/
- FSF list of fully free distros: https://www.gnu.org/distros/free-distros.html
- Trisquel: https://trisquel.info/
- gNewSense (archive): https://en.wikipedia.org/wiki/GNewSense — discontinued 2017
- Parabola: https://www.parabola.nu/
- Hyperbola: https://www.hyperbola.info/ — Wikipedia notes founding April 2017, FSF-endorsed Dec 2018, HyperbolaBSD pivot announced Dec 2019
- Guix System: https://guix.gnu.org/
- Replicant: https://www.replicant.us/

**Bell Labs apostates**
- Plan 9 (Bell Labs): https://9p.io/plan9/
- Inferno history (Vita Nuova): http://www.vitanuova.com/inferno/
- 9front: http://9front.org/

**Privacy purists**
- Tails: https://tails.net/
- Qubes OS: https://www.qubes-os.org/ — first release 3 September 2012
- Whonix: https://www.whonix.org/
- Subgraph (archive): https://en.wikipedia.org/wiki/Subgraph_(operating_system)

**Anti-systemd schism**
- Alpine: https://alpinelinux.org/
- Void: https://voidlinux.org/
- Devuan: https://www.devuan.org/ — see also the original "Veteran Unix Admins" open letter, late 2014
- antiX: https://antixlinux.com/
- Artix: https://artixlinux.org/

**Mobile rebels**
- Maemo (archive): https://en.wikipedia.org/wiki/Maemo
- MeeGo (archive): https://en.wikipedia.org/wiki/MeeGo
- Sailfish OS (Jolla): https://sailfishos.org/
- UBports: https://ubports.com/
- postmarketOS: https://postmarketos.org/
- LineageOS: https://lineageos.org/ and history page for the CyanogenMod fork
- GrapheneOS: https://grapheneos.org/ — note the 2014 origin as CopperheadOS, rebranded 2019
- CalyxOS: https://calyxos.org/
- /e/OS (Murena): https://e.foundation/
- PureOS (Purism): https://pureos.net/

**From-scratch idealists**
- TempleOS (archive): http://templeos.org/ and Wikipedia for biographical context
- SerenityOS: https://serenityos.org/
- Redox OS: https://www.redox-os.org/

**Microkernel holdouts**
- Minix 3: https://www.minix3.org/
- Genode: https://genode.org/
- seL4 (Trustworthy Systems / DARPA HACMS history): https://sel4.systems/
- HelenOS: http://www.helenos.org/

**Immutable wave**
- NixOS: https://nixos.org/ — Eelco Dolstra's PhD thesis 'The Purely Functional Software Deployment Model' (2006) is the canonical primary source
- Fedora Silverblue: https://fedoraproject.org/silverblue/
- openSUSE MicroOS: https://microos.opensuse.org/
- Vanilla OS: https://vanillaos.org/ — first release Vanilla OS 22.10 on 29 December 2022 (founder Mirko Brombin); Vanilla OS 2.0 Orchid switched base from Ubuntu to Debian, released July 2024
- Universal Blue (Bluefin, Bazzite, Aurora): https://universal-blue.org/ — community formed mid-2023; Bluefin announced 2023; Bazzite first release November 2023 based on Fedora 38

**Market rebels**
- SteamOS history (Valve): https://store.steampowered.com/steamos/ — version 1 Debian-based 2013, version 3 Arch-based February 2022 with the Steam Deck
- HoloISO: https://github.com/HoloISO/holoiso
- ChimeraOS: https://chimeraos.org/ — originally GamerOS 2019
- Pop!_OS (System76): https://pop.system76.com/

**Eschatologists**
- Collapse OS: http://collapseos.org/ — Virgil Dupras, 2019
- Dusk OS: http://duskos.org/ — successor project

---

## 10. Acceptance criteria

The layer is done when:

1. Loading `os_genealogy.html` with no toggle interaction shows **exactly** the current chart, visually unchanged.
2. Clicking *The Free Peoples* chip reveals ~45 new bars, distributed correctly across the houses (most in `linux` and `mobile`, with handfuls in `unix`, `altpc`).
3. Selecting Devuan shows its full bloodline back to `unixv7` via `debian` → ... → and the movement badge appears under the name.
4. Selecting a non-movement OS (e.g. macOS) shows the normal panel with no movement badge.
5. The lineage line from `cyanogenmod` to `lineageos` (forked) is visible in the chart and traceable on hover.
6. Dataset integrity holds: no missing parent refs, no duplicate IDs, year range remains 1966–2027.
7. The clade legend lists ten clades with non-zero counts (with the possible exception of one or two single-entry clades like `eschaton`).
8. Toggling the layer off restores the chart to the initial state with no visual glitches in the SVG (lanes don't reflow if stable packing is chosen).

---

*End of brief.*
