# Witcher's Medallion — Design Document

*Working title. A terminal user interface for Linux that shows the live state of the machine as a world drawn from Andrzej Sapkowski's Witcher books.*

| | |
|---|---|
| Status | Draft 1, foundation document |
| Date | 2026-09-19 |
| Author | adamko |
| License intent | personal, non-commercial fan project; all art original |
| Target | Linux (amd64, arm64), developed on macOS |
| Stack | Go, Bubble Tea, Lip Gloss, Bubbles |

---

## How to read this document

- **Domain first.** Section 2 is the heart. Everything else hangs off it. If you ever feel lost in the UI or the collectors, come back to the domain model and ask "which entity is this about?"
- **Backwards planning.** Section 7 starts at v1.0 and walks down to a weekend-sized first milestone. Each milestone says what v1.0 needs from it.
- **Analogies.** Each fundamentals topic (sections 4 and 5) has an *Analogy* paragraph. Witcher analogies where they fit naturally, plain ones otherwise.
- **Confidence markers.** Where I am not sure of an API detail or a kernel behaviour, the text says **Verify:** and tells you where to look. Trust the man pages over this document.
- **Vocabulary.** Witcher terms in *italics* on first use; Linux terms in `code`. A quick two-way glossary is in Appendix C.

### Contents

1. [Vision](#1-vision)
2. [Domain model](#2-domain-model)
3. [Architecture](#3-architecture)
4. [Terminal fundamentals](#4-terminal-fundamentals)
5. [Linux fundamentals](#5-linux-fundamentals)
6. [Safety](#6-safety)
7. [Roadmap, worked backwards](#7-roadmap-worked-backwards)
8. [Stretch ideas](#8-stretch-ideas)
9. [Risks and pitfalls](#9-risks-and-common-pitfalls)
10. [Learning resources](#10-curated-learning-resources)
- Appendix A: `/proc/[pid]/stat` fields
- Appendix B: Signals
- Appendix C: Glossary
- Appendix D: Fixture capture script

---

## 1. Vision

### 1.1 What it feels like

You open a terminal, type `medallion`, and the screen becomes *the Continent*: a map of your machine. The dwarven forges of *Mahakam* glow to the rhythm of your CPU cores. The *Redanian treasury* fills and empties as memory is used. Your *toxicity* (load average) creeps up when a build runs. The *notice board* lists every process as a contract, with the beast's name, who posted it, and how much it is eating. *Trade roads* carry network traffic; *portals* are your open sockets. At the bottom, *Jaskier* (Dandelion, in the English translation) sings the journal: the newest log lines, in his own words.

When something is wrong, the *medallion* vibrates: the title bar trembles, a warning line appears, and the panel that caused it is highlighted. A core pegged at 100% for half a minute, a disk at 97%, a temperature near its trip point, the *Wild Hunt* (the OOM killer) having taken a victim.

Select a contract and press Enter: a full-screen contract sheet with a landscape, the beast's vitals, its *wards* (which signals it catches or ignores), its lair (`cwd`), its sire (parent), and the five *Signs* you can cast on it. Every Sign is a real syscall. Every Sign asks first.

Press `L` anywhere and the *scholar overlay* appears: it names the real Linux concept, the exact file the panel reads, the raw line, and the formula. This is the learning feature. The theme is a costume; the overlay is the anatomy lesson underneath.

Principles:

- **Every number is real.** Nothing is faked, smoothed into meaninglessness, or invented for drama.
- **Keyboard first.** Mouse is optional and never required for anything, and never used for anything destructive.
- **Read-only by default.** Watching costs nothing and cannot break anything. Acting is explicit and confirmed.
- **No root for the core experience.** Everything on the main view is world-readable. *Elder Blood* (root) unlocks more, and the app tells you exactly what and why.
- **Theme is a skin.** The domain speaks Linux; the theme speaks Witcher. A `plain` theme ships alongside, for debugging, screenshots, and people who have not read the books.
- **Original art only.** Landscapes, borders, runes, abstract creatures. No character likenesses, no official logos or medallion designs.

### 1.2 Mockup: the Continent (main view)

Drawn at 96 columns. Minimum supported size is 80x24; below that the app shows a "the medallion is too small to read" placeholder with the required size.

```
╭──────────────────────────────────────────────────────────────────────────────────────────────╮
│ WITCHER'S MEDALLION      kaer-morhen · linux 6.12.9 · 14 days since the Conjunction    12:41 │
├──────────────────────────────┬───────────────────────────────┬───────────────────────────────┤
│ MAHAKAM FORGES        cpu    │ REDANIAN TREASURY       mem   │ TOXICITY             load     │
│ forge 0  ▓▓▓▓▓▓░░░░░░  48%   │ in use   ▓▓▓▓▓▓▓░░░░░  9.8G   │  1 min  ▓▓▓▓░░░░░░░░   1.92   │
│ forge 1  ▓▓░░░░░░░░░░  17%   │ cached   ▓▓▓░░░░░░░░░  3.1G   │  5 min  ▓▓▓▓▓░░░░░░░   2.31   │
│ forge 2  ▓▓▓▓▓▓▓▓▓▓▓░  92% ! │ free     ░░░░░░░░░░░░  3.4G   │ 15 min  ▓▓▓▓▓░░░░░░░   2.40   │
│ forge 3  ▓░░░░░░░░░░░   6%   │ swap     ░░░░░░░░░░░░  0.2G   │ tolerance 4 forges = 4.00     │
│ heat 61°C  ▁▂▂▃▅▆▅▃▂▁▂▃▄     │ Hunt sightings since boot: 0  │ Power 82%  drawing from vein  │
├──────────────────────────────┴───────────────────────────────┴───────────────────────────────┤
│ NOTICE BOARD   contracts 213 · hunting 3 · resting 208 · trapped 1 · wraiths 1    sort: cpu  │
│   pid    posted by  beast           state     cpu%    mem  nice  wards  since                │
│ » 41872  adam       go build        hunting   91.3   412M     0  A      12:38:14             │
│   1204   root       dockerd         resting    2.1   188M     0  A H    boot                 │
│   8830   adam       node            resting    1.4   690M     0  -      11:02:51             │
│   27     root       kworker/2:1     golem      0.7      -     0  -      boot                 │
│   9917   adam       sleep           trapped    0.0     1M    10  -      12:40:02             │
│   9918   adam       (defunct)       wraith     0.0      -     0  -      12:40:03             │
│   ... 207 more contracts (PgDn)                                                              │
├────────────────────────────────┬─────────────────────────────────────────────────────────────┤
│ TRADE ROADS & PORTALS          │ JASKIER'S CHRONICLE                                  latest │
│ eth0   rx 1.2M/s  tx 88K/s     │ 12:41:07 sshd     portal opened from 10.0.0.12 for adam     │
│ wlan0  rx 0       tx 0    down │ 12:40:59 medallion forge 2 runs hot: 92% for 30s            │
│ portals 3 open · 2 waiting     │ 12:40:31 systemd  Started Daily apt download activities.    │
├────────────────────────────────┴─────────────────────────────────────────────────────────────┤
│ j/k choose   enter contract   / search   s sort   c chronicle   b bestiary   L scholar   q   │
╰──────────────────────────────────────────────────────────────────────────────────────────────╯
```

Reading it:

- **Forges**: one bar per core, from `/proc/stat`. The `!` marks a core above the alert threshold. The heat line is the hottest thermal zone with a sparkline of the last 13 samples.
- **Treasury**: `MemTotal - MemAvailable` as "in use"; `Cached`; `MemAvailable` as "free"; swap used. "Hunt sightings" is the `oom_kill` counter from `/proc/vmstat`.
- **Toxicity**: the three load averages, with the bar scaled to the number of online cores ("tolerance"). "Power" is the battery, if any, with the charging state in lore words.
- **Notice board**: the process table. `wards` shows what the beast resists: `A` catches or ignores SIGTERM (so *Aard* may bounce off), `H` ignores SIGHUP, `Q` is shielded from the Hunt (`oom_score_adj <= -900`). `state` uses lore words backed by the real state letter (see 2.5).
- **Roads and portals**: per-interface rates from `/proc/net/dev`; socket counts from `/proc/net/tcp*`.
- **Chronicle**: the latest journal lines plus the app's own events, newest first.

### 1.3 Mockup: a contract (process detail with Sign actions)

```
╭─ CONTRACT 41872 ─────────────────────────────────────────────────────────────────────────────╮
│                                                                                              │
│   ╭──────────────────────────────╮   beast       go build                                    │
│   │      /\        .   ~   /\    │   kind        griffin - devours forges (cpu 91% for 2m)   │
│   │  ^  /  \   /\    ^    /  \   │   posted by   adam (uid 1000)                             │
│   │ / \/    \_/  \__/ \__/    \  │   sire        42001 zsh  ·  lineage: 1 child              │
│   │/   ▒▒▒   ▒▒    |    ▒▒  ▒▒ \ │   since       12:38:14 (2m 31s)  ·  state: hunting (R)    │
│   │  ▒▒▒▒▒▒▒▒▒▒▒▒▒▒|▒▒▒▒▒▒▒▒▒▒▒▒ │   lair        /home/adam/src/medallion                    │
│   │~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~│   command     go build ./cmd/medallion                    │
│   ╰──────────────────────────────╯   threads     9   ·   last seen on forge 2                │
│                                                                                              │
│   forges (cpu)    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░  91.3%   ▁▃▅▇█▇▆▇█▇   user 2m01s  sys 0m12s           │
│   treasury (mem)  ▓░░░░░░░░░░░░░░░░░░░   2.6%   rss 412M  virt 1.9G  swapped 0               │
│   nice            0   (Axii not cast)          oom_score 214   adj 0   (no Quen)             │
│   wards           catches Aard (SIGTERM handler)  ·  ignores SIGHUP  ·  no Yrden on it       │
│   portals         none                                                                       │
│                                                                                              │
├──────────────────────────────────────────────────────────────────────────────────────────────┤
│  SIGNS                                                                                       │
│  a  Aard    push it away             SIGTERM       y  Yrden   trap it              SIGSTOP   │
│  i  Igni    burn it, no return       SIGKILL       Y  break Yrden                  SIGCONT   │
│  x  Axii    sway its will            renice        q  Quen    ward from the Hunt   oom adj   │
│                                                                                              │
│  Every Sign asks before it is cast. Without Elder Blood you can only make a beast more       │
│  yielding (raise nice) or less warded (raise oom adj), and only your own beasts.             │
├──────────────────────────────────────────────────────────────────────────────────────────────┤
│  esc back   t threads   f files   p portals   e environment   L scholar   ? help             │
╰──────────────────────────────────────────────────────────────────────────────────────────────╯
```

Casting a Sign opens a confirmation. For Igni (SIGKILL) you type the beast's name:

```
        ╭─ Cast Igni on 41872 go build? ───────────────────╮
        │  The beast will burn. It cannot dodge, catch     │
        │  or ignore this Sign. Unsaved work is lost.      │
        │                                                  │
        │  Type the beast's name to confirm:  go buil_     │
        │                                                  │
        │            enter cast        esc sheathe         │
        ╰──────────────────────────────────────────────────╯
```

The scholar overlay, on any panel:

```
╭─ SCHOLAR: TOXICITY ─────────────────────────────────────────╮
│ source    /proc/loadavg                                     │
│ raw       1.92 2.31 2.40 3/1187 41990                       │
│ formula   toxicity = load1 / online forges = 1.92 / 4 = 0.48│
│ refresh   every 1s                                          │
│ read      proc_loadavg(5), getloadavg(3)                    │
╰─────────────────────────────────────────────────────────────╯
```

### 1.4 Mockup: help and bestiary

```
╭─ BESTIARY ───────────────────────────────────────────────────────────────────────────────────╮
│  What the medallion senses, what it truly is, and which Sign (if any) answers it.            │
│                                                                                              │
│  beast           the sign of it               what it really is                  answer      │
│  ──────────────  ───────────────────────────  ─────────────────────────────────  ──────────  │
│  griffin         devours forges               sustained high cpu%                Axii        │
│  alghoul         hoards the treasury          rss growing sample over sample     watch, Aard │
│  wraith          cannot be slain              zombie: exited, sire never waited  tell sire   │
│  drowner         stuck beneath the surface    D state, uninterruptible sleep     wait        │
│  golem           made of the land itself      kernel thread (PF_KTHREAD)         none        │
│  doppler         wears another's face         comm differs from exe name         inspect     │
│  troll           under the bridge for ages    daemon alive since boot            Aard, gently│
│  ghoul pack      many small hungry mouths     dozens of short-lived siblings     Yrden sire  │
│  striga          shrugs off Aard              catches or ignores SIGTERM         Igni        │
│  higher vampire  your steel does not bite     another user's process (EPERM)     Elder Blood │
│  djinn           bound, dangerous power       holds capabilities (CapEff != 0)   inspect     │
│  golden dragon   do not hunt                  protected: pid 1, your shell, ...  none        │
│                                                                                              │
│  KEYS                                                                                        │
│  j/k or arrows move   enter open contract   / search   s cycle sort   c chronicle            │
│  b bestiary   L scholar overlay (real names and /proc sources)   ? this help   q quit        │
│                                                                                              │
│  SIGNS ARE REAL. Aard sends SIGTERM, Igni SIGKILL, Yrden SIGSTOP/SIGCONT, Axii calls         │
│  setpriority(2), Quen writes /proc/<pid>/oom_score_adj. Each one asks first.                 │
╰──────────────────────────────────────────────────────────────────────────────────────────────╯
```

---

## 2. Domain model

### 2.1 The one design decision that matters most

**The domain speaks Linux. The theme speaks Witcher.**

The entity is `Process`, not `Contract`. The value object is `LoadAverage`, not `Toxicity`. The action is `Terminate`, not `Aard`. The Witcher vocabulary lives in one package, `theme`, which maps domain concepts to names, glyphs, colours, flavour text, and landscapes. Swapping the theme means swapping that package. Debugging means switching to the `plain` theme and seeing `SIGTERM` where you would otherwise see *Aard*.

Why this matters beyond swappability: the Linux concepts have exact semantics (a zombie is a very specific thing; `MemAvailable` is a very specific number). If the domain used lore names, you would eventually bend the semantics to fit the story. The story must bend to the machine, not the other way round.

*Analogy*: a witcher's medallion reacts to magic. It does not decide what magic is; it only translates it into a vibration you can feel. The theme is the medallion. The domain is the magic.

### 2.2 The mapping, refined and challenged

Your core mapping, with adjustments. Where I changed something, the reason is in the last column.

| Linux concept | Witcher parallel | Verdict |
|---|---|---|
| SIGTERM | *Aard* (telekinetic push) | Keep. A push knocks the beast down; a strong one gets back up (the process can catch or ignore SIGTERM). Exactly right. |
| SIGKILL | *Igni* (fire) | Keep. Uncatchable, unblockable, irreversible. One caveat that makes a good lesson: a process in `D` state (uninterruptible sleep) will not die until it wakes. "Even Igni does not burn a beast sunk in the bog." |
| SIGSTOP / SIGCONT | *Yrden* (binding trap) / breaking it | Keep. SIGSTOP is uncatchable like SIGKILL; the trap holds until broken. SIGTSTP (Ctrl-Z) is the weaker, catchable version: a Yrden the beast can step out of. |
| `setpriority(2)` (renice) | *Axii* (sway the mind) | Keep, and it is better than it first looks: without privilege you can only make a process *more* yielding (raise nice). Persuading it to fight harder (lower nice) needs *Elder Blood*. Axii calms; it does not enrage. |
| `oom_score_adj` | *Quen* (protective shield) | Keep. Same privilege shape as Axii: unprivileged users can only weaken the shield (raise the value); strengthening it (lowering below the process's minimum) needs `CAP_SYS_RESOURCE`. At `-1000` the process is immune to the Hunt. |
| Process list | Contracts on the notice board | Keep. The board is per village (per host). |
| Zombie process | *Ghoul* → **recommend *wraith*** | Challenge. Ghouls in the books are living necrophages, they eat corpses. A zombie is a corpse: it has exited, only its exit status remains, and nothing you do to it works; it is laid to rest only when its parent calls `wait()`. That is a *wraith*: bound to the world by unfinished business, released only when the business is settled. I keep "ghoul" for something else (2.5). This is a one-word change in the theme package either way. |
| fork/exec | Conjunction of the Spheres → **recommend boot = Conjunction; fork+exec = the *Trial of the Grasses*** | Challenge. The Conjunction was a single cataclysm that populated the world. That is boot: the kernel starts PID 1 and the world fills with processes. Uptime becomes "days since the Conjunction". `fork()` produces a child that is a copy of its parent; `execve()` then transforms it into something new while keeping the same PID. A boy is taken in, and the Trial of the Grasses remakes him into a witcher. Many do not survive the Trial (exec fails: `ENOENT`, `EACCES`). |
| OOM killer | *The Wild Hunt* | Keep. It rides when the sky darkens (memory pressure), it takes one victim chosen by a score, and only Quen wards against it. `/proc/pressure/memory` (PSI) is the darkening sky, before the Hunt appears. |
| Alerts | Medallion vibrating | Keep. In the books the medallion twitches near magic and monsters. Severity levels: *stirs*, *trembles*, *vibrates*, *tugs at the chain*. |
| Load average | Potion toxicity | Keep. Three readings (1, 5, 15 min). A witcher's tolerance is his number of cores. |
| Root / capabilities | *Elder Blood* | Keep, and extend: full root is Elder Blood. Individual capabilities (`CAP_KILL`, `CAP_SYS_NICE`, `CAP_SYS_RESOURCE`, `CAP_SYS_PTRACE`) are partial gifts of the blood. The app reads its own `CapEff` and reports which gifts it has. |
| Sockets, SSH | Portals | Keep. `LISTEN` is a portal held open; `ESTABLISHED` is one in use; `TIME_WAIT` is a portal collapsing behind someone. Geralt distrusts portals in the books, which fits: connections can drop you somewhere in pieces. |
| journald | Jaskier's ballads | Keep. Log priority maps to how dramatic the ballad is. Jaskier embellishes; the scholar overlay shows the raw JSON. |
| CPU cores | Mahakam forges | Keep. Dwarven forges in the mountain: heat, hammering, output. Per-core temperature fits naturally. |
| RAM | Redanian treasury | Keep. `MemAvailable` is what is actually spendable. Swap is a loan from the *Giancardi bank* (Molnar Giancardi, *Time of Contempt*): money you can get, slowly, at a cost. |
| Disks | Oxenfurt archives | Keep. Free space is shelf space; throughput is scribes at work; `util%` is how many scribes are busy. |
| Network interfaces | Trade roads | Keep. Down interfaces are roads closed by war. |
| Thermal zones | Forge heat | New. Lives inside Mahakam on the main view; own panel on a "kingdoms" screen. |
| Battery / AC | The mage's reserve of Power, drawn from a water vein | New. Sorceresses in the books draw the Force from veins and intersections. Charging is "drawing from the vein". |
| PID 1 | *Vesemir* at Kaer Morhen | New. Every process descends from PID 1; orphans are taken in there (the Law of Surprise delivers children to the keep). |
| Kernel threads | *Golems*, made of the land itself | New. Children of `kthreadd` (PID 2). Signals do nothing useful; they are not contracts you can take. |
| Protected processes (PID 1, your own shell, sshd, the compositor) | *The golden dragon* | New. In "The Bounds of Reason" Geralt refuses to hunt the dragon. Some things a witcher does not kill. |
| Process in `D` state | *Drowner* | New. Under the surface, unreachable, waiting on I/O. |
| Another user's process (`EPERM`) | *Higher vampire* | New. Your silver does not bite. Only Elder Blood can touch it. |
| Process holding capabilities | *Djinn* | New. Bound power in an ordinary-looking vessel. |
| `comm` differs from `exe` basename | *Doppler* | New. Wears another's face. Legitimate (busybox, interpreters) or suspicious. |
| Namespaces (`/proc/[pid]/ns/*` differ from ours) | *Another sphere* | New. Containers are other worlds that touch ours. Stretch. |
| cgroups / systemd units | Guilds and orders | New. Stretch. |

### 2.3 Entities and value objects

Notation is Go-flavoured pseudocode. Entities have identity and change over time; value objects are compared by value and are immutable. Nothing in this section imports Bubble Tea or Lip Gloss.

#### Identity

```go
// A PID alone is not an identity: the kernel reuses PIDs. A process is
// identified by its PID plus its start time in clock ticks since boot
// (/proc/[pid]/stat field 22). If the PID matches but StartTicks does not,
// it is a different process wearing the same number.
type ProcessID struct {
    PID        int
    StartTicks uint64
}
```

#### Value objects

```go
type Jiffies    uint64          // clock ticks (USER_HZ, normally 100/s)
type Bytes      uint64
type Percent    float64         // 0..100, may exceed 100 for multi-threaded per-process cpu
type Nice       int8            // -20 (greedy) .. 19 (yielding)
type OOMAdj     int16           // -1000 (immune) .. 1000 (first to go)
type MilliC     int32           // temperature, millidegrees Celsius
type Rate       float64         // bytes per second
type SignalMask uint64          // bit (n-1) set means signal n

func (m SignalMask) Has(sig syscall.Signal) bool { return m&(1<<(uint(sig)-1)) != 0 }

type ProcessState byte          // 'R','S','D','T','t','Z','X','I'

type LoadAverage struct{ One, Five, Fifteen float64 }

// Derived, unitless: load per online core. 1.0 = every core has one runnable task.
type Toxicity float64

type UserRef struct {
    UID  uint32
    Name string   // "" if lookup failed
}

type Privileges struct {
    EUID     uint32
    CapEff   uint64        // from /proc/self/status CapEff
}
func (p Privileges) IsRoot() bool         { return p.EUID == 0 }
func (p Privileges) Has(cap int) bool     { return p.CapEff&(1<<uint(cap)) != 0 }
```

#### Entities

```go
type Host struct {                     // "the kingdom"
    Hostname   string                  // /proc/sys/kernel/hostname
    Kernel     string                  // /proc/sys/kernel/osrelease
    BootTime   time.Time               // /proc/stat "btime"
    ClockTicks int                     // USER_HZ, see 5.2
    PageSize   int                     // os.Getpagesize() on the *target*
    NumCPU     int                     // count of "cpuN" lines in /proc/stat
}

type CPUCore struct {                  // "a forge"
    Index   int
    Sample  CPUSample                  // raw jiffies at this snapshot
    // derived over two snapshots:
    Busy    Percent
    IOWait  Percent
    Steal   Percent
}
type CPUSample struct{ User, Nice, System, Idle, IOWait, IRQ, SoftIRQ, Steal, Guest, GuestNice Jiffies }

type Memory struct {                   // "the treasury"
    Total, Free, Available, Buffers, Cached, Shmem, Dirty Bytes
    SwapTotal, SwapFree                                     Bytes
    // derived:
    Used      Bytes                    // Total - Available
    UsedPct   Percent
    SwapUsed  Bytes
}

type Process struct {                  // "a contract"
    ID        ProcessID
    Comm      string                   // /proc/[pid]/comm, max 15 chars
    Cmdline   []string                 // /proc/[pid]/cmdline, NUL-separated; empty for kernel threads and zombies
    Exe       string                   // readlink /proc/[pid]/exe; "" if EACCES
    Cwd       string                   // readlink /proc/[pid]/cwd; "" if EACCES
    State     ProcessState
    PPID      int
    User      UserRef                  // effective uid from /proc/[pid]/status "Uid:" (2nd value)
    Nice      Nice
    Priority  int
    Threads   int
    Flags     uint32                   // stat field 9; PF_KTHREAD = 0x00200000
    UTime, STime Jiffies               // stat fields 14, 15
    RSS       Bytes                    // stat field 24 (pages) * PageSize
    VSize     Bytes                    // stat field 23
    Swap      Bytes                    // status VmSwap
    LastCPU   int                      // stat field 39
    OOMScore  int                      // /proc/[pid]/oom_score
    OOMAdj    OOMAdj                   // /proc/[pid]/oom_score_adj
    SigCgt    SignalMask               // status SigCgt: handlers installed
    SigIgn    SignalMask               // status SigIgn: ignored
    CapEff    uint64                   // status CapEff
    TTY       int                      // stat field 7; 0 = no controlling terminal
    CGroup    string                   // /proc/[pid]/cgroup (v2: single line "0::/user.slice/...")
    // derived over two snapshots and the whole table:
    CPU       Percent                  // share of one core
    Children  []int
    Age       time.Duration
    Kind      Kind                     // bestiary classification, 2.5
    Wards     Wards                    // what it resists, 2.6
}

type Thread struct{ TID int; Comm string; State ProcessState; UTime, STime Jiffies; LastCPU int }

type Disk struct {                     // "an archive"
    Name       string                  // sda, nvme0n1 (whole devices only; partitions excluded)
    Sample     DiskSample              // /proc/diskstats raw counters
    // derived:
    ReadRate, WriteRate Rate
    Util       Percent                 // delta(ms doing I/O) / delta(wall ms)
}
type DiskSample struct{ ReadsCompleted, SectorsRead, MsReading, WritesCompleted, SectorsWritten, MsWriting, IOInProgress, MsDoingIO uint64 }

type Mount struct {                    // "a shelf in the archive"
    Device, Path, FSType string        // /proc/self/mounts
    Total, Free, Avail   Bytes         // statfs(2): f_blocks, f_bfree, f_bavail * f_bsize
}

type NetInterface struct {             // "a trade road"
    Name       string
    Sample     NetSample                // /proc/net/dev
    OperState  string                   // /sys/class/net/<if>/operstate: up, down, unknown, dormant
    // derived:
    RxRate, TxRate Rate
    RxErrDelta, TxErrDelta uint64
}
type NetSample struct{ RxBytes, RxPackets, RxErrs, RxDrop, TxBytes, TxPackets, TxErrs, TxDrop uint64 }

type Socket struct {                   // "a portal"
    Proto      string                  // tcp, tcp6, udp, udp6, unix
    Inode      uint64
    Local, Remote netip.AddrPort
    State      SocketState             // 0x01 ESTABLISHED .. 0x0A LISTEN
    UID        uint32
    PID        int                     // 0 if unknown (needs fd scan, see 5.7)
}

type ThermalZone struct {              // "forge heat"
    Name     string                    // /sys/class/thermal/thermal_zoneN/type or hwmon name+label
    Temp     MilliC
    Hot      *MilliC                   // trip point of type "hot" or "passive", if present
    Critical *MilliC                   // trip point "critical"
}

type PowerSupply struct {              // "the reserve of Power"
    Name        string                 // BAT0, AC, ADP1
    IsBattery   bool
    Capacity    int                    // percent
    Status      string                 // Charging, Discharging, Full, Not charging, Unknown
    EnergyNow, EnergyFull, PowerNow uint64  // µWh, µWh, µW (or charge_*/current_* variants converted)
    Online      bool                   // for AC adapters
}

type LogEntry struct {                 // "a verse of the ballad"
    Time       time.Time
    Priority   int                     // 0 emerg .. 7 debug (syslog levels)
    Unit       string                  // _SYSTEMD_UNIT
    Identifier string                  // SYSLOG_IDENTIFIER
    PID        int                     // _PID
    Message    string
    Source     LogSource               // Journal, Kmsg, Medallion (our own events)
}

type Alert struct {                    // "the medallion vibrates"
    Kind      AlertKind                // CPUHot, CoreSaturated, MemoryLow, MemoryPressure, OOMKill, DiskFull, DiskBusy, Thermal, BatteryLow, LinkDown, NewZombie, StuckIO, NewListener, CastFailed
    Severity  Severity                 // Info (stirs), Warn (trembles), Crit (vibrates), Emergency (tugs)
    Subject   string                   // "forge 2", "/home", "pid 41872"
    Since     time.Time
    Message   string                   // theme-independent, e.g. "core 2 above 90% for 30s"
}
```

#### Actions (Signs)

```go
type ActionKind int
const (
    Terminate ActionKind = iota   // SIGTERM      Aard
    Kill                          // SIGKILL      Igni
    Stop                          // SIGSTOP      Yrden
    Continue                      // SIGCONT      break Yrden
    Renice                        // setpriority  Axii
    Shield                        // oom_score_adj Quen
)

type Action struct {
    Kind   ActionKind
    Target ProcessID
    Nice   *Nice        // for Renice: absolute target value
    OOMAdj *OOMAdj      // for Shield: absolute target value
}

// The domain decides whether an action may be attempted, before any syscall.
type Verdict int
const (
    Allowed        Verdict = iota
    NeedsPrivilege         // would get EPERM/EACCES: other user's process, lowering nice, strengthening shield
    Protected              // policy: pid 1, kernel thread, ourselves, our ancestors, configured wards
    Pointless              // zombie: nothing to signal; already stopped for Stop; etc.
    LikelyIneffective      // Terminate on a process that catches/ignores SIGTERM; Kill on D state
)

type Plan struct {
    Action   Action
    Verdict  Verdict
    Reason   string          // "process 41872 installs a SIGTERM handler; it may not exit"
    Confirm  ConfirmLevel    // None, Simple, TypeName
}

func PlanAction(a Action, p Process, self Self, priv Privileges, policy Policy) Plan
```

The `Plan` is what the confirmation dialog renders. The UI never decides policy; it only shows the plan and collects the confirmation.

### 2.4 Snapshot → World: how derivation works

Two immutable snapshots are needed for any rate (CPU%, disk throughput, network rate). The domain keeps this explicit:

```
                 collect (I/O)                    derive (pure)
 /proc, /sys  ────────────────►  Snapshot(t) ──┐
                                               ├─►  World(t) = rates, alerts, kinds, history
                                Snapshot(t-1) ─┘
```

```go
// Snapshot is what the collector produces: raw values at one instant. No rates.
type Snapshot struct {
    At        time.Time                 // monotonic-backed; use time.Now() and also store a monotonic delta
    Host      Host
    Cores     []CPUSample
    Total     CPUSample                 // the "cpu" aggregate line
    Load      LoadAverage
    Uptime    time.Duration
    Memory    Memory
    VMStat    struct{ OOMKills uint64 } // /proc/vmstat "oom_kill"
    Pressure  Pressure                  // /proc/pressure/{cpu,memory,io}, optional
    Procs     map[int]Process           // keyed by PID; derived fields zero
    Disks     []DiskSample
    Mounts    []Mount
    Nets      []NetSample
    Sockets   []Socket
    Thermal   []ThermalZone
    Power     []PowerSupply
}

// World is what the UI renders: the current snapshot plus everything that needs history.
type World struct {
    Now       Snapshot
    Prev      *Snapshot
    Cores     []CPUCore
    Procs     []Process                 // sorted per UI request, derived fields filled
    Disks     []Disk
    Nets      []NetInterface
    Alerts    []Alert
    History   History                   // ring buffers for sparklines: per-core busy, load1, temp, rx/tx
    Privs     Privileges
}

func Advance(prev World, next Snapshot, rules AlertRules) World
```

Rules of the derivation layer:

- Pure functions. Given the same two snapshots, the same `World`. This is what makes fixture-based testing on macOS possible (3.6).
- A process present in `Now` but absent from `Prev` gets `CPU = 0` for this tick, never a guess.
- A process whose `ProcessID` changed (same PID, different `StartTicks`) is a new process.
- Rates use the wall-clock delta between snapshots, not the nominal tick interval. Ticks jitter; sleeps and suspends make big holes.

### 2.5 Process kinds (the bestiary as classification rules)

`Kind` is derived, in this order; the first match wins. Thresholds live in `Policy`, not in code.

| Kind | Rule | Lore |
|---|---|---|
| `Zombie` | `State == 'Z'` | wraith |
| `KernelThread` | `Flags & PF_KTHREAD != 0` (or `PPID == 2` and empty cmdline on old kernels) | golem |
| `Protected` | matches policy (see 6.3) | golden dragon |
| `UninterruptibleSleep` | `State == 'D'` for ≥ N consecutive samples | drowner |
| `Stopped` | `State == 'T'` or `'t'` | trapped in Yrden |
| `Foreign` | `User.UID != self.UID` and not root ourselves | higher vampire |
| `Capable` | `CapEff != 0` and `UID != 0` | djinn |
| `Shapeshifter` | `Exe != ""` and `basename(Exe) != Comm` (with a small allowlist: interpreters, busybox) | doppler |
| `CPUHog` | `CPU ≥ 80%` of one core for ≥ 30 s | griffin |
| `MemoryGrower` | `RSS` increased in ≥ 5 of the last 6 samples and is above 100 MiB | alghoul |
| `Daemon` | `TTY == 0`, `PPID == 1`, age > 1 h, and a `.service` cgroup | troll |
| `Swarm` | a parent with ≥ 10 children whose median age < 5 s | ghoul pack (the parent is what you would Yrden) |
| `SignalResistant` | `SigCgt.Has(SIGTERM) || SigIgn.Has(SIGTERM)` | striga |
| `Ordinary` | everything else | contract |

Note on `Kind` being single-valued: a process can be both a daemon and a CPU hog. `Kind` is the headline; `Wards` (below) and the raw fields carry the rest. If that feels lossy later, make `Kinds` a set. Start simple.

### 2.6 Wards (what a process resists)

Derived from `/proc/[pid]/status`:

| Ward | Source | Meaning for the Signs |
|---|---|---|
| `CatchesTerm` | `SigCgt` bit 14 | Aard may be caught and handled (graceful shutdown or ignored) |
| `IgnoresTerm` | `SigIgn` bit 14 | Aard does nothing |
| `IgnoresHup` | `SigIgn` bit 0 | survives terminal hangup (nohup, daemons) |
| `Shielded` | `OOMAdj ≤ -900` | the Hunt passes it by |
| `Immune` | `OOMAdj == -1000` | the Hunt cannot take it |
| `Uninterruptible` | `State == 'D'` | no Sign lands until it wakes |
| `Traced` | `status TracerPid != 0` | a mage (debugger) holds it; signals go to the tracer first |

### 2.7 Alert rules (when the medallion vibrates)

All thresholds are in `Policy` with these defaults. Every rule has a *sustain* time so a one-sample spike does not vibrate the medallion.

| Kind | Condition (default) | Sustain | Severity |
|---|---|---|---|
| `CoreSaturated` | any core busy ≥ 90% | 30 s | Info |
| `CPUHot` | total busy ≥ 95% | 60 s | Warn |
| `ToxicityHigh` | `load1 / NumCPU ≥ 1.0` / `≥ 2.0` | 60 s | Warn / Crit |
| `MemoryLow` | `Available / Total < 10%` / `< 5%` | 10 s | Warn / Crit |
| `MemoryPressure` | `/proc/pressure/memory` `some avg10 ≥ 10` / `full avg10 ≥ 5` | 0 | Warn / Crit |
| `OOMKill` | `/proc/vmstat oom_kill` increased | 0 | Crit ("the Wild Hunt has taken a victim"; name from journal if readable) |
| `DiskFull` | any mount `Avail / Total < 10%` / `< 3%` (excluding pseudo filesystems) | 0 | Warn / Crit |
| `DiskBusy` | any disk `Util ≥ 90%` | 30 s | Info |
| `Thermal` | `Temp ≥ Hot` trip, or `≥ 85°C` if no trip / `≥ Critical - 5°C` | 5 s | Warn / Crit |
| `BatteryLow` | discharging and capacity `< 20%` / `< 10%` | 0 | Warn / Crit |
| `LinkDown` | an interface with an IP address changed `operstate` to `down` | 0 | Warn |
| `NetErrors` | `RxErrDelta + TxErrDelta > 0` | 0 | Info |
| `NewZombie` | a process entered `Z` | 0 | Info |
| `StuckIO` | a process in `D` | 30 s | Warn |
| `NewListener` | a new `LISTEN` socket appeared on a non-loopback address | 0 | Info |
| `CastFailed` | an action returned an error | 0 | Info |

An alert is an entity with identity `(Kind, Subject)`; it persists while its condition holds and is cleared (with a "the medallion goes still" chronicle line) when it stops.

### 2.8 The full mapping table: concept → parallel → source → refresh

Refresh rates are defaults; the collector schedules each group on its own cadence (3.4). "Once" means at startup and on demand.

| Linux concept | Witcher parallel | Data source (exact) | Refresh |
|---|---|---|---|
| Hostname, kernel | the kingdom's name and age | `/proc/sys/kernel/hostname`, `/proc/sys/kernel/osrelease` | once |
| Boot time, uptime | the Conjunction, days since | `/proc/stat` line `btime`; `/proc/uptime` field 1 | once / 1 s |
| CPU cores utilisation | Mahakam forges | `/proc/stat` lines `cpu`, `cpu0..N`: user nice system idle iowait irq softirq steal guest guest_nice (jiffies) | 1 s |
| Context switches, forks | hammer strokes, births | `/proc/stat` lines `ctxt`, `processes`, `procs_running`, `procs_blocked` | 1 s |
| Load average | potion toxicity | `/proc/loadavg`: `1m 5m 15m running/total lastpid` | 1 s |
| Memory | Redanian treasury | `/proc/meminfo`: `MemTotal MemFree MemAvailable Buffers Cached Shmem Dirty SwapTotal SwapFree` (kB) | 1 s |
| Swap | loan from the Giancardi bank | `/proc/meminfo` `SwapTotal - SwapFree`; per process `VmSwap` in `/proc/[pid]/status` | 1 s / 2 s |
| Memory pressure | the sky darkening | `/proc/pressure/memory` (PSI, kernel ≥ 4.20): `some avg10=… avg60=… avg300=… total=…`, `full …` | 1 s |
| OOM kills | the Wild Hunt's victims | `/proc/vmstat` line `oom_kill` (kernel ≥ 4.13); details in journal `-k` "Out of memory: Killed process" | 1 s |
| Process list | notice board | `readdir(/proc)` numeric entries; `/proc/[pid]/stat`, `/proc/[pid]/comm` | 2 s |
| Process details | contract sheet | `/proc/[pid]/status` (`Uid`, `VmRSS`, `VmSwap`, `Threads`, `SigCgt`, `SigIgn`, `CapEff`, `TracerPid`), `/proc/[pid]/cmdline`, `/proc/[pid]/cgroup`, `/proc/[pid]/oom_score`, `/proc/[pid]/oom_score_adj` | 1 s, selected process only |
| Process lair, image | lair, true face | `readlink(/proc/[pid]/cwd)`, `readlink(/proc/[pid]/exe)` (need ptrace read access: same user or `CAP_SYS_PTRACE`) | on select |
| Threads | the beast's limbs | `/proc/[pid]/task/*/stat` | 1 s, selected only |
| Open files | the beast's hoard | `readlink(/proc/[pid]/fd/*)` (same user or `CAP_SYS_PTRACE`) | on demand |
| Environment | the beast's memories | `/proc/[pid]/environ` (same user or `CAP_SYS_PTRACE`) | on demand |
| Per-process CPU | how hard the beast fights | `/proc/[pid]/stat` fields 14 `utime`, 15 `stime`; delta ÷ (`CLK_TCK` × Δt) | 2 s |
| Per-process memory | how much it eats | `/proc/[pid]/stat` field 24 `rss` (pages) × page size; field 23 `vsize` | 2 s |
| Process start time | when the contract was posted | `/proc/[pid]/stat` field 22 `starttime` (ticks since boot) + `btime` | with list |
| Zombie | wraith | `/proc/[pid]/stat` field 3 == `Z` | 2 s |
| Kernel thread | golem | `/proc/[pid]/stat` field 9 `flags & 0x00200000` (`PF_KTHREAD`) | 2 s |
| Nice / priority | Axii's hold | `/proc/[pid]/stat` fields 19 `nice`, 18 `priority`; set with `setpriority(PRIO_PROCESS, pid, nice)` | 2 s |
| OOM adj | Quen's strength | `/proc/[pid]/oom_score_adj` (read/write), `/proc/[pid]/oom_score` (read) | 1 s selected |
| Signals | the Signs | `kill(2)` via `syscall.Kill` / `os.Process.Signal`; preferably `pidfd_open(2)` + `pidfd_send_signal(2)` (kernel ≥ 5.3) | on cast |
| Our privileges | Elder Blood | `/proc/self/status` lines `Uid`, `CapEff`; `os.Geteuid()` | once |
| Disks throughput | archive scribes | `/proc/diskstats`: fields 4 reads, 6 sectors read, 7 ms reading, 8 writes, 10 sectors written, 11 ms writing, 12 in progress, 13 ms doing I/O (sectors are always 512 B here) | 1 s |
| Whole-disk vs partition | shelves vs rooms | `/sys/block/*` lists whole devices; partitions are `/sys/block/sda/sda1` | once |
| Mount usage | shelf space | `/proc/self/mounts` (device, mountpoint, fstype); `statfs(2)` via `unix.Statfs` for `Blocks`, `Bfree`, `Bavail`, `Bsize` | 30 s |
| I/O pressure | scribes overwhelmed | `/proc/pressure/io` | 1 s |
| Network rates | trade roads | `/proc/net/dev`: per interface rx bytes, packets, errs, drop … tx bytes, packets, errs, drop | 1 s |
| Link state | road open/closed | `/sys/class/net/<if>/operstate`, `/sys/class/net/<if>/carrier`, `/sys/class/net/<if>/speed` | 5 s |
| Addresses | where the road leads | `net.Interfaces()` + `Addrs()` (Go stdlib, uses netlink under the hood) | 30 s |
| TCP/UDP sockets | portals | `/proc/net/tcp`, `/proc/net/tcp6`, `/proc/net/udp`, `/proc/net/udp6`: `local_address rem_address st … uid … inode` (hex, little-endian per 32-bit word) | 5 s |
| Unix sockets | local passages | `/proc/net/unix` | 5 s |
| Socket → process | who opened the portal | scan `readlink(/proc/[pid]/fd/*)` for `socket:[inode]` (own processes only without root) | 5 s |
| Temperature | forge heat | `/sys/class/thermal/thermal_zone*/temp` (m°C), `.../type`, `.../trip_point_N_temp`, `.../trip_point_N_type`; and/or `/sys/class/hwmon/hwmon*/temp*_input`, `temp*_label`, `temp*_max`, `temp*_crit`, `hwmon*/name` | 5 s |
| Battery | reserve of Power | `/sys/class/power_supply/BAT*/{capacity,status,energy_now,energy_full,energy_full_design,power_now,voltage_now,cycle_count}` (some batteries use `charge_now`/`charge_full`/`current_now` instead) | 30 s |
| AC adapter | the water vein | `/sys/class/power_supply/{AC,ADP*,ACAD}/online` | 30 s |
| System log | Jaskier's ballads | `journalctl --output=json --follow --lines=200 --no-pager` (fields `MESSAGE`, `PRIORITY`, `_SYSTEMD_UNIT`, `SYSLOG_IDENTIFIER`, `_PID`, `_COMM`, `__REALTIME_TIMESTAMP` in µs) | streaming |
| Kernel log | the land itself speaking | `/dev/kmsg` (needs `CAP_SYSLOG` or `kernel.dmesg_restrict=0`), or `journalctl -k` | streaming |
| Users | who posted the contract | `/etc/passwd` via `os/user.LookupId` (cache the result) | on first sight |
| cgroup / unit | guild | `/proc/[pid]/cgroup` (v2: `0::/system.slice/sshd.service`) | with details |
| Namespaces | other spheres | `readlink(/proc/[pid]/ns/pid)` compared to `/proc/self/ns/pid` | stretch |

### 2.9 Theme contract

The theme package turns domain concepts into presentation. It never reads data.

```go
type Theme interface {
    Name() string                                    // "witcher", "plain"
    Term(c Concept) string                           // Concept.CPU -> "Mahakam forges" / "CPU"
    KindName(k Kind) string                          // Zombie -> "wraith" / "zombie"
    StateName(s ProcessState) string                 // 'R' -> "hunting" / "running"
    ActionName(a ActionKind) (label, mechanism string) // Kill -> "Igni", "SIGKILL"
    AlertText(a Alert) string                        // OOMKill -> "The Wild Hunt has taken pid 4242"
    Severity(s Severity) string                      // Warn -> "the medallion trembles"
    Landscape(k Kind, width int) []string            // original ASCII art, sized
    Glyphs() Glyphs                                  // bar fill/empty, sparkline set, borders; has ASCII fallback
    Palette() Palette                                // colours with 256/16 fallbacks
}
```

The witcher theme can carry its opinions (wraith vs ghoul) without touching the domain. The plain theme is also what golden-file tests render, so that lore wording changes never break tests of the data.

---

## 3. Architecture

### 3.1 Stack decision: Go + Bubble Tea + Lip Gloss

I agree with the choice. Reasons, and the honest costs.

**For it**

- Bubble Tea's Elm architecture (`Model`, `Update`, `View`) is a forcing function for the domain-first design you want. State is a value, side effects are explicit `Cmd`s, and the view is a pure function of state. If you have used Redux or `useReducer` in TypeScript, it is the same shape: `(state, action) → state`, plus effects returned rather than performed.
- Pure Go, no cgo. Cross-compiling from macOS to Linux is one environment variable, and the binary is static. Every alternative that links a C library (ncurses, libsystemd) breaks this.
- Reading `/proc` is byte parsing, which is a small, honest, testable job in Go. You will write the parsers yourself, which is the point of the project.
- The ecosystem is mature: Bubbles (table, viewport, textinput, help, key bindings), Lip Gloss (layout and style), `teatest` (golden-file tests), `wish` (serve a TUI over SSH, a stretch idea that costs almost nothing).

**Against it, so you know what you are signing up for**

- Bubble Tea hides the terminal from you. Raw mode, escape sequences, resize signals, mouse encoding: all handled. Since learning terminals is a goal, milestone M2 (section 7) has you write a 150-line raw-terminal program *without* Bubble Tea first, so you see what the library does for you.
- There is no layout engine. You compute widths from the window size and join strings. Lip Gloss makes that pleasant, not automatic. Budget time for layout code.
- The `Model` is passed by value. Keep large data behind slices, maps, or pointers, or every `Update` copies it.
- **Verify:** Bubble Tea v2 (`charm.land/bubbletea/v2`) has been in development alongside v1 (`github.com/charmbracelet/bubbletea`). I cannot confirm from here which is marked stable on the day you start. Check the README. Sketches in this document use the v1 API because it is the one every tutorial uses. Known v2 changes to watch for (confirm against the changelog): `tea.KeyMsg` becomes `tea.KeyPressMsg`/`tea.KeyReleaseMsg`; `View()` returns a `tea.View` value rather than a `string`; Lip Gloss v2 (`charm.land/lipgloss/v2`) uses `color.Color` and gets the background colour via a `tea.BackgroundColorMsg` instead of a blocking query at startup.

**Alternatives considered**

| Option | Why not (for this project) |
|---|---|
| `tcell` + `tview` (Go) | Imperative widget tree, like Swing. Fine, but it does not push you toward pure state and it hides less of interest. |
| Rust + `ratatui` | The best immediate-mode TUI library going, and Rust's `procfs` crate is excellent. But you are learning Go, and two new things at once halves the learning. |
| Python + Textual | Gorgeous, but a browser-like abstraction over the terminal. You would learn Textual, not terminals. |
| C + ncurses (like `htop`) | Maximal learning about terminals, minimal fun. Read `htop`'s source instead (section 10). |

Supporting libraries, all pure Go:

| Need | Library | Notes |
|---|---|---|
| Syscalls, constants | `golang.org/x/sys/unix` | `unix.Kill`, `unix.Setpriority`, `unix.Statfs`, `unix.PidfdOpen`, `unix.PidfdSendSignal`, `unix.Uname`. **Verify** the pidfd wrappers exist in the version you pin. |
| Terminal helpers | `golang.org/x/term` | `term.IsTerminal`, `term.MakeRaw`, `term.Restore`, `term.GetSize`. Used directly only in the M2 toy. |
| `CLK_TCK` | `github.com/tklauser/go-sysconf` | `sysconf.Sysconf(sysconf.SC_CLK_TCK)`. Or assume 100 (see 5.2). |
| Display width | `github.com/charmbracelet/x/ansi` (`ansi.StringWidth`), or `github.com/mattn/go-runewidth` | Lip Gloss's `lipgloss.Width` already strips ANSI and measures. |
| Golden tests | `github.com/charmbracelet/x/exp/teatest` | `teatest.NewTestModel`, `WaitFor`, `RequireEqualOutput`. |
| Config | `github.com/pelletier/go-toml/v2` or `github.com/BurntSushi/toml` | Either. |
| Reference parsers | `github.com/prometheus/procfs` | Do not depend on it; read it when your parser disagrees with reality. |

### 3.2 Layers

```
                       ┌───────────────────────────────────────────────┐
   user input ───────► │  ui       Bubble Tea models, screens, layout  │ ──► terminal
                       │           imports: domain, theme               │
                       ├───────────────────────────────────────────────┤
                       │  theme    words, glyphs, colours, landscapes  │
                       │           imports: domain (types only)        │
                       ├───────────────────────────────────────────────┤
                       │  domain   entities, Advance(), kinds, alerts, │
                       │           PlanAction(); imports: stdlib only  │
                       ├───────────────────────────────────────────────┤
   /proc, /sys ──────► │  collect  Source → Snapshot (live / replay)   │
                       │  procfs   pure parsers: []byte → structs      │
                       │  sysfs    pure parsers                        │
   kill(2) etc. ◄───── │  act      Caster (linux) / fake               │
   journalctl ───────► │  journal  LogEntry stream                     │
                       └───────────────────────────────────────────────┘
```

Dependency rule: arrows of `import` point toward `domain`. `domain` imports nothing from the project. `ui` never reads a file. `collect` never formats a string for display. If you find yourself wanting `lipgloss` inside `domain`, stop; that is the theme's job.

*Analogy*: the witcher (ui) reads the notice board (domain) and consults the bestiary (theme). The board is written by the village scribe (collect) who walks around counting things. The witcher does not do the counting; the scribe does not decide which beast is worth hunting.

### 3.3 The Bubble Tea loop

```
              ┌──────────────────────────────────────────────────┐
              │                     Program                       │
   keys,      │   ┌────────┐   Msg    ┌────────┐  string  ┌─────┐│
   resize,    │   │ inputs │ ───────► │ Update │ ───────► │View ││──► renderer ──► terminal
   mouse ────►│   └────────┘          └───┬────┘          └─────┘│     (diffs lines,
              │        ▲                  │ Cmd                  │      repaints changed)
              │        │  Msg             ▼                      │
              │   ┌────┴─────────────────────────┐               │
              │   │ Cmd goroutines: tick, collect,│               │
              │   │ cast, wait-for-log-line       │               │
              │   └──────────────────────────────┘               │
              └──────────────────────────────────────────────────┘
```

- `Init() tea.Cmd`: return the first commands (start the tick, do the first collection).
- `Update(msg) (Model, Cmd)`: the only place state changes. Must return fast. Never read a file here.
- `View() string`: render the whole screen as one string. Bubble Tea's renderer compares it with the previous frame line by line and repaints only changed lines, at up to 60 fps (`tea.WithFPS`).
- `tea.Cmd` is `func() tea.Msg`. The program runs it in a goroutine and feeds the returned message back into `Update`. That is the entire concurrency model you need.

*Analogy*: you sit at a table in the inn with a map (the model). Couriers arrive with messages (a tick of the clock, a key pressed, the scribe's new count). You read each one and redraw the map (Update). If you need something fetched, you do not leave the table; you send a runner (Cmd) who returns later as another courier. The map on the table is always complete and always yours to look at (View).

Sketch, v1 API:

```go
type Model struct {
    world    domain.World
    screen   Screen                 // continent, contract, chronicle, bestiary, help
    board    table.Model            // bubbles/table for the notice board
    logs     viewport.Model         // bubbles/viewport for the chronicle
    search   textinput.Model
    confirm  *ConfirmModel          // non-nil while a Sign awaits confirmation
    scholar  bool                   // overlay on/off
    width, height int
    theme    theme.Theme
    src      collect.Source
    caster   act.Caster
    sched    collect.Scheduler
    inFlight bool                   // a collection is running; skip this tick
    keys     KeyMap
}

type tickMsg     time.Time
type snapshotMsg struct{ snap domain.Snapshot; err error }
type logLinesMsg []domain.LogEntry
type castMsg     struct{ plan domain.Plan; err error }

func (m Model) Init() tea.Cmd {
    return tea.Batch(tick(), collect(m.src, m.sched.Due(time.Now())), waitForLogs(m.logCh))
}

func tick() tea.Cmd {
    return tea.Tick(time.Second, func(t time.Time) tea.Msg { return tickMsg(t) })
}

func collect(src collect.Source, groups collect.Groups) tea.Cmd {
    return func() tea.Msg { s, err := src.Collect(groups); return snapshotMsg{s, err} }
}

func waitForLogs(ch <-chan []domain.LogEntry) tea.Cmd {
    return func() tea.Msg { return logLinesMsg(<-ch) }    // blocks in its own goroutine; fine
}

func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        m.width, m.height = msg.Width, msg.Height
        m.relayout()
        return m, nil
    case tickMsg:
        if m.inFlight { return m, tick() }             // drop the tick, never queue collections
        m.inFlight = true
        return m, tea.Batch(tick(), collect(m.src, m.sched.Due(time.Time(msg))))
    case snapshotMsg:
        m.inFlight = false
        if msg.err != nil { m.lastErr = msg.err; return m, nil }
        m.world = domain.Advance(m.world, msg.snap, m.policy)
        m.refreshBoard()
        return m, nil
    case logLinesMsg:
        m.appendLogs(msg)
        return m, waitForLogs(m.logCh)                 // re-arm; one Cmd per batch of lines
    case castMsg:
        m.recordCast(msg)
        return m, nil
    case tea.KeyMsg:
        return m.handleKey(msg)
    }
    return m, nil
}
```

Message catalogue:

| Message | Produced by | Update does |
|---|---|---|
| `tea.WindowSizeMsg` | Bubble Tea on start and SIGWINCH | store size, recompute layout |
| `tickMsg` | `tea.Tick` | re-arm tick; start a collection if none is running |
| `snapshotMsg` | `collect` Cmd | `domain.Advance`; refresh table rows; clear `inFlight` |
| `logLinesMsg` | `waitForLogs` Cmd (journal goroutine → channel) | append to chronicle; re-arm |
| `castMsg` | `cast` Cmd | write to cast log; show result; add chronicle line |
| `tea.KeyMsg` | Bubble Tea | navigation, screen changes, open confirm, confirm/cancel |
| `tea.MouseMsg` | Bubble Tea (if enabled) | row selection, scroll; never confirms a cast |
| `vibrateMsg` | `tea.Tick` at 50 ms while an alert is fresh | advance the shake animation frame (stretch) |

Two rules that prevent most Bubble Tea bugs:

1. **Every Cmd that should repeat must be re-issued from Update.** `tea.Tick` fires once. Forget to re-arm and the app freezes silently.
2. **Never block in Update.** If a collection takes 40 ms, do it in a Cmd. If it takes longer than the tick, skip ticks rather than queue them.

### 3.4 Collectors

```go
// FS is the only thing the collectors need from the OS. Live: rooted at "/".
// Replay: rooted at testdata/scenarios/<name>/tick-0001/. Paths are unrooted
// ("proc/stat", not "/proc/stat"), like io/fs.
type FS interface {
    ReadFile(path string) ([]byte, error)
    ReadDir(path string) ([]fs.DirEntry, error)
    ReadLink(path string) (string, error)         // io/fs has no readlink; see Verify below
}

// Probe covers things that are syscalls, not files.
type Probe interface {
    Statfs(mountpoint string) (Total, Free, Avail Bytes, err error)
    PageSize() int
    ClockTicks() int
    Now() time.Time
}

type Source interface {
    Collect(groups Groups) (domain.Snapshot, error)
}
```

**Verify:** Go 1.25 added `io/fs.ReadLinkFS` (with `ReadLink` and `Lstat`), and `os.DirFS` implements it. If so, `FS` can be `fs.FS` plus a type assertion. Either way, keep the interface small and your own; it is the seam that makes macOS development possible.

Scheduling groups and cadence:

| Group | Contents | Default |
|---|---|---|
| `Fast` | `/proc/stat`, `/proc/loadavg`, `/proc/meminfo`, `/proc/vmstat`, `/proc/pressure/*`, `/proc/diskstats`, `/proc/net/dev`, `/proc/uptime` | 1 s |
| `Procs` | `readdir /proc`, `stat` + `comm` + `status` per PID | 2 s |
| `Selected` | `status`, `oom_score`, `oom_score_adj`, `task/*`, `cwd`, `exe` for the selected PID | 1 s |
| `Sockets` | `/proc/net/{tcp,tcp6,udp,udp6,unix}`, fd scan for own PIDs | 5 s |
| `Thermal` | `/sys/class/thermal/*`, `/sys/class/hwmon/*` | 5 s |
| `Power` | `/sys/class/power_supply/*` | 30 s |
| `Mounts` | `/proc/self/mounts` + `statfs` | 30 s |
| `Static` | hostname, kernel, `btime`, `NumCPU`, own privileges | once |

Groups not due keep their previous values in the next `Snapshot`. `Snapshot` carries a `SampledAt` per group so rates use the right delta.

Parsing rules for `procfs`:

- Every parser is `func ParseX(b []byte) (X, error)`, pure, with table-driven tests and a `testdata/` sample. No file I/O in parsers.
- `/proc/[pid]/stat`: the `comm` field is in parentheses and may contain spaces and parentheses. Find the *last* `)` and split the remainder with `bytes.Fields`. Field numbers in Appendix A are 1-based as in `proc_pid_stat(5)`, so field 3 (`state`) is index 0 after the `)`.
- `/proc/[pid]/cmdline` is NUL-separated; split on `\x00`, drop the trailing empty element. Empty means kernel thread or zombie.
- Unknown extra fields at the end of a line are ignored; missing fields on old kernels default to zero. Never fail a whole snapshot because one PID vanished (`ENOENT`, `ESRCH`) or one file was unreadable (`EACCES`). Skip the PID, record the error count.
- Numbers: `strconv.ParseUint(string(f), 10, 64)`. The string conversion allocates; it is fine until profiling says otherwise.

Performance budget: a full `Procs` collection with 500 processes reads about 1500 small files. That is a few milliseconds of syscalls and under 20 ms parsed. Set a hard target of 50 ms and check it with `go test -bench` on a fixture and `pprof` on the VM.

Record and replay, the feature that pays for itself:

- `medallion record --out testdata/scenarios/build-storm --interval 1s --duration 30s`: wraps the live `FS` so every path read is copied into `tick-NNNN/<path>`; `meta.json` stores timestamps, page size, `CLK_TCK`, and the `statfs` results.
- `medallion --replay testdata/scenarios/build-storm [--speed 4]`: a `ReplaySource` walks the tick directories and feeds the *same parsers and the same derivation*. The UI cannot tell the difference. This runs on macOS.
- The same directories are the fixtures for `teatest` golden files and for domain tests ("after tick 7 the `CPUHot` alert must be active").

### 3.5 Actions

```go
type Caster interface {
    Signal(id domain.ProcessID, sig syscall.Signal) error
    SetNice(id domain.ProcessID, n domain.Nice) error
    SetOOMAdj(id domain.ProcessID, adj domain.OOMAdj) error
}
```

- `caster_linux.go` (`//go:build linux`): before every syscall, re-read `/proc/[pid]/stat` and compare `StartTicks` with `id.StartTicks`; mismatch means the PID was reused, return `ErrGone`. Prefer `unix.PidfdOpen(pid, 0)` then `unix.PidfdSendSignal(fd, sig, nil, 0)` on kernels ≥ 5.3: the file descriptor pins the identity so the reuse race disappears. Fall back to `unix.Kill`. Nice: `unix.Setpriority(unix.PRIO_PROCESS, pid, int(n))`. Shield: `os.WriteFile("/proc/<pid>/oom_score_adj", []byte(strconv.Itoa(int(adj))), 0)`.
- `caster_other.go` (`//go:build !linux`): every method returns `act.ErrUnsupported`. The UI shows "the Signs do not work in this land".
- `fake.go`: records calls; used in tests and with `--fake-signs` for UI work on macOS.
- Error mapping: `EPERM` → "your steel does not bite" (`NeedsPrivilege`), `ESRCH` → "the beast is already gone", `EACCES` on the oom file → same as EPERM, `EINVAL` → a bug in our code, say so.

### 3.6 Journal

Shell out; do not link libsystemd.

```go
cmd := exec.CommandContext(ctx, "journalctl", "--output=json", "--follow", "--lines=200", "--no-pager")
```

A goroutine reads stdout with `bufio.Scanner` (raise the buffer: journal lines can be long, `scanner.Buffer(make([]byte, 0, 64*1024), 1<<20)`), parses each JSON object into `LogEntry`, and sends batches (drain up to 200 lines or 100 ms, whichever first) on a channel consumed by `waitForLogs`. Cancel the context on quit so `journalctl` dies with us. If `journalctl` is missing or exits with "No journal files were found", the chronicle shows "Jaskier is not in town" and the app carries on. Unprivileged users see their own user journal plus the system journal only if they are in `systemd-journal`, `adm`, or `wheel`; the scholar overlay explains that.

Note that `MESSAGE` can be an array of bytes (non-UTF-8 messages are encoded as a JSON array of numbers by `journalctl`); handle both string and array. `PRIORITY` is a string in the JSON.

### 3.7 Package layout

```
medallion/
  go.mod                          module github.com/<you>/medallion
  cmd/medallion/main.go           flags, wiring, tea.NewProgram(...).Run()
  internal/domain/                entities.go derive.go kinds.go wards.go alerts.go plan.go history.go policy.go
  internal/procfs/                stat.go pidstat.go status.go meminfo.go loadavg.go vmstat.go pressure.go
                                  diskstats.go netdev.go nettcp.go mounts.go cmdline.go  (+ testdata/, *_test.go)
  internal/sysfs/                 thermal.go hwmon.go power.go netclass.go
  internal/collect/               fs.go probe.go source.go live.go replay.go record.go schedule.go
  internal/act/                   caster.go caster_linux.go caster_other.go fake.go
  internal/journal/               reader.go parse.go fake.go
  internal/theme/                 theme.go concept.go glyphs.go palette.go
  internal/theme/witcher/         witcher.go words.go art/  (embedded *.txt landscapes)
  internal/theme/plain/           plain.go
  internal/ui/                    app.go keys.go layout.go msgs.go cmds.go
  internal/ui/screens/            continent.go contract.go chronicle.go bestiary.go help.go confirm.go scholar.go
  internal/ui/widgets/            bar.go spark.go panel.go
  testdata/scenarios/<name>/      tick-0001/proc/... tick-0001/sys/... meta.json
  hack/rawterm/main.go            the M2 "naked terminal" toy
  hack/capture.sh                 Appendix D
  Makefile
```

`internal/` stops anything outside the module importing these packages, which is what you want for an app. `cmd/medallion` holds subcommands: the TUI (default), `record`, `snapshot` (print one `Snapshot` as JSON; also the transport for the SSH stretch idea), and flags `--replay DIR`, `--theme plain`, `--no-signs`, `--fake-signs`, `--ascii`.

Layout and rendering notes for `ui`:

- Compute a `Layout` struct from `(width, height)` once per resize: column widths, table height, whether the lower panels fit. Render from it. Do not measure strings during `View` more than necessary.
- Lip Gloss v1: `Style.Width(n)` is the width of the content box *including padding, excluding border*. Use `style.GetHorizontalFrameSize()` (border + padding + margin) when budgeting columns. This is the number one cause of "my panels are two columns too wide".
- Join panels with `lipgloss.JoinHorizontal(lipgloss.Top, a, b, c)` and rows with `lipgloss.JoinVertical`. Pad every panel to its full width and height so the frame is stable; a line that changes length between frames flickers.
- Sanitise every string from the system before rendering: `cmdline` and `comm` can contain control characters and escape sequences. Strip everything below `0x20` except tab, and `0x7f`. A process named `\x1b[2J` must not clear your screen.

### 3.8 Developing on macOS, testing for Linux

Facts: macOS has no `/proc`, no `/sys`, no journald, and its `kill`/`setpriority` semantics differ subtly. Only three packages touch the OS (`collect.live`, `act.linux`, `journal`). Everything else runs anywhere.

The loop:

1. **On macOS, all day**: unit tests for parsers and domain; the TUI in `--replay` mode with `--fake-signs`. Fast, no VM.
2. **In a Linux VM, several times a day**: cross-compile and run for real. Check the numbers against `top`, `free -m`, `cat /proc/loadavg`.
3. **In CI, every push**: GitHub Actions `ubuntu-latest` runners have a real `/proc`, `/sys`, and journald. Tag integration tests with `//go:build integration` and run them there.

Cross-compile (pure Go makes this trivial; it is the strongest reason to avoid cgo):

```sh
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -trimpath -o bin/medallion-linux-arm64 ./cmd/medallion   # Apple Silicon VM
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -trimpath -o bin/medallion-linux-amd64 ./cmd/medallion   # Intel machines, servers
```

VM options:

| Option | Notes |
|---|---|
| Lima (`brew install lima`, `limactl start`) | Free. Ubuntu with systemd, so journald works. Mounts your home directory into the VM at the same path (read-only by default; configure writable if you want `record` to write into the repo). `limactl shell default` gives you a terminal in the VM; run the arm64 binary from there. Recommended. |
| OrbStack | Fastest file sharing and startup; free for personal use. Same idea as Lima. |
| Multipass, UTM | Fine. Multipass is Ubuntu-only and simple. |
| Docker (`docker run -it --rm --pid=host -v "$PWD/bin:/app" ubuntu /app/medallion-linux-arm64`) | Quick checks only. `--pid=host` shows the Docker VM's processes. No systemd, so no journal. Note the TUI needs `-it` for a TTY. |

Makefile targets to have from day one: `test`, `lint` (`go vet`, `staticcheck`), `build-linux`, `vm-run` (build + `limactl shell default -- /path/to/bin`), `record`, `replay`, `golden-update`.

Fixture pitfalls (see Appendix D for the script):

- `/proc` files report size 0. `cp -r` and `tar` produce empty files. Use `cat` per file.
- Symlinks (`exe`, `cwd`, `fd/*`, `ns/*`): store the link target as text in `<name>.link`; the replay `FS` serves `ReadLink` from it.
- Record `page size` and `CLK_TCK` in `meta.json`. Apple Silicon Linux VMs may use 4 KiB or 16 KiB pages; `rss` in `stat` is in pages.
- Do not capture `environ`. Scrub hostnames and usernames from `cmdline` if you plan to publish the repo.
- Keep scenarios small and named for what they prove: `idle`, `build-storm`, `memory-pressure`, `zombie`, `stopped`, `many-procs`, `no-thermal` (a VM without sensors).

UI tests with `teatest` (**Verify** exact function names against the package docs):

```go
func TestContinentGolden(t *testing.T) {
    lipgloss.SetColorProfile(termenv.Ascii)           // no colour codes in goldens
    m := ui.New(replaySource("idle"), act.Fake{}, theme.Plain{}, fakeClock)
    tm := teatest.NewTestModel(t, m, teatest.WithInitialTermSize(96, 30))
    teatest.WaitFor(t, tm.Output(), func(b []byte) bool { return bytes.Contains(b, []byte("LOAD")) })
    tm.Send(tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("q")})
    out, _ := io.ReadAll(tm.FinalOutput(t))
    teatest.RequireEqualOutput(t, out)                // -update flag rewrites the golden
}
```

### 3.9 Configuration

`$XDG_CONFIG_HOME/medallion/config.toml` (default `~/.config/medallion/config.toml`). Cast history at `$XDG_STATE_HOME/medallion/casts.log`.

```toml
[refresh]
fast = "1s"
procs = "2s"
thermal = "5s"
power = "30s"
sockets = "5s"

[thresholds]
toxicity_warn = 1.0
toxicity_crit = 2.0
mem_available_warn_pct = 10
mem_available_crit_pct = 5
core_saturated_pct = 90
core_saturated_sustain = "30s"
thermal_warn_c = 85

[wards]                          # protected processes, see 6.3
protect_comm = ["sshd", "systemd*", "dbus-*", "tmux: server", "Xorg", "gnome-shell", "sway", "Hyprland", "login", "agetty"]

[signs]
enabled = true                   # false = read-only; same as --no-signs
type_name_for = ["kill"]         # actions that require typing the process name

[theme]
name = "witcher"                 # or "plain"
ascii = false                    # true forces ASCII borders and bars
```

---

## 4. Terminal fundamentals

Each topic: what it is, why the medallion cares, what Bubble Tea does about it, something to try in a shell, and an analogy. Try the commands on the Linux VM; most also work on macOS.

### 4.1 TTY and PTY

**What.** A TTY (teletype) is the kernel's idea of a terminal device: a character device with a *line discipline* in between the program and the hardware. Real hardware terminals are gone; what you have is a *pseudo-terminal* (PTY): a pair of devices. The terminal emulator (Ghostty, iTerm2, GNOME Terminal) holds the *master* side (`/dev/ptmx`); your shell and your programs hold the *slave* side (`/dev/pts/3` on Linux, `/dev/ttys004` on macOS). Bytes written to one side appear on the other, after passing through the line discipline (`N_TTY`). SSH, tmux, and `script` each allocate their own PTY pairs. Each process has at most one *controlling terminal*; `/proc/[pid]/stat` field 7 (`tty_nr`) tells you which, and 0 means none, which is how daemons look.

**Why it matters.** The medallion is a program on the slave side of a PTY. Everything it draws is bytes written to a file descriptor. Everything it reads (keys, mouse) is bytes read from the same descriptor. If there is no TTY (`medallion | less`, `ssh host medallion` without `-t`, cron), the app must refuse cleanly. Daemons with `tty_nr == 0` are the *trolls* of the bestiary.

**Bubble Tea.** Checks `isatty` on start (`golang.org/x/term.IsTerminal`) and errors out if there is none. Uses the PTY size from `ioctl(TIOCGWINSZ)`.

**Try.**
```sh
tty                          # /dev/pts/0
ls -l /proc/self/fd/0        # -> /dev/pts/0  (Linux)
ps -o pid,tty,comm           # which processes share your terminal; daemons show "?"
script -q /dev/null bash     # a shell inside a fresh PTY; run `tty` again
ssh host 'tty'               # "not a tty"; ssh -t host 'tty' allocates one
```

*Analogy.* The PTY is the innkeeper's counter. You (the program) sit on one side; the traveller (the terminal emulator) on the other. Nothing passes except across the counter, and the innkeeper (line discipline) is in the middle deciding what to hand over and when.

### 4.2 ANSI escape codes

**What.** In-band control: byte sequences that begin with `ESC` (`0x1b`) and tell the terminal to do something other than print. The important family is CSI, `ESC [`, followed by parameters and a final letter. Output (program → terminal):

| Sequence | Effect |
|---|---|
| `ESC[H` / `ESC[{row};{col}H` | cursor home / to position (1-based) |
| `ESC[2J`, `ESC[K` | clear screen, clear to end of line |
| `ESC[?25l`, `ESC[?25h` | hide, show cursor |
| `ESC[0m`, `ESC[1m`, `ESC[2m` | reset, bold, dim |
| `ESC[31m` … `ESC[37m`, `ESC[90m` … `ESC[97m` | 16 foreground colours (`4x` background) |
| `ESC[38;5;{n}m` | 256-colour foreground (`48;5` background) |
| `ESC[38;2;{r};{g};{b}m` | truecolor foreground (`48;2` background) |
| `ESC[?1049h` / `ESC[?1049l` | enter / leave the alternate screen |
| `ESC[?1000h` `ESC[?1002h` `ESC[?1003h` `ESC[?1006h` | mouse tracking: clicks, drag, all motion, SGR encoding |
| `ESC[?2004h` | bracketed paste |
| `ESC[?2026h` / `ESC[?2026l` | synchronized output begin/end (newer terminals; reduces tearing) |

Input (terminal → program) uses the same alphabet: the up arrow arrives as `ESC [ A`, F1 as `ESC O P`, a mouse click (SGR mode) as `ESC [ < 0 ; 12 ; 5 M`. A lone `ESC` is ambiguous: is it the Escape key, or the start of a sequence whose rest has not arrived yet? Every TUI library resolves that with a short timeout.

**Why it matters.** Everything you see in the mockups is these sequences plus text. Lip Gloss emits the `m` sequences; Bubble Tea emits the cursor and screen ones. Understanding them lets you debug "why is there garbage on my screen" with `cat -v`.

**Bubble Tea.** Parses input sequences into `tea.KeyMsg` and `tea.MouseMsg`; emits output sequences from the renderer. You never write `ESC[` yourself, except in the M2 toy.

**Try.**
```sh
printf '\e[1;31mred bold\e[0m\n'
printf '\e[38;2;255;120;0mtruecolor?\e[0m\n'         # if it is orange, your terminal supports 24-bit colour
printf '\e[?1049h'; sleep 2; printf '\e[?1049l'      # alternate screen, briefly
cat -v                                                # then press arrow keys, Esc, F1; Ctrl-C to exit
infocmp | head -20                                    # what terminfo says about $TERM
```

*Analogy.* The Signs are gestures with a fixed meaning: a particular hand shape produces Aard, every time, in every land. Escape sequences are the gestures of the terminal. `ESC[` is the witcher raising his hand; what follows is which Sign.

### 4.3 Raw versus cooked mode (termios)

**What.** The line discipline has settings, read and written with `tcgetattr`/`tcsetattr` on a `termios` struct. In *cooked* (canonical) mode, the kernel buffers a whole line, handles Backspace, echoes what you type, and turns Ctrl-C into `SIGINT`, Ctrl-Z into `SIGTSTP`, Ctrl-D into end-of-file. Your program sees complete lines. In *raw* mode, all of that is off: every byte arrives immediately (`ICANON` off), nothing is echoed (`ECHO` off), Ctrl-C is just byte `0x03` (`ISIG` off), and `VMIN`/`VTIME` control blocking. The relevant flags live in `c_lflag` (`ICANON`, `ECHO`, `ISIG`, `IEXTEN`), `c_iflag` (`IXON`, `ICRNL`), and `c_oflag` (`OPOST`).

**Why it matters.** A TUI needs raw mode: `j` must move the cursor without waiting for Enter. The cost: you are responsible for everything the kernel used to do, including quitting on Ctrl-C and restoring the terminal on exit. A program that dies in raw mode leaves the shell blind and deaf (nothing echoes; Enter does not work). `reset` or `stty sane` fixes it.

**Bubble Tea.** Calls `term.MakeRaw` on start and `term.Restore` on exit. It catches panics and restores the terminal before re-panicking (opt out with `tea.WithoutCatchPanics`), but nothing can help you against `kill -9`. Ctrl-C arrives as a `tea.KeyMsg` whose `String()` is `"ctrl+c"`; you decide what it means.

**Try.**
```sh
stty -a                       # icanon, echo, isig ... the current settings
stty raw -echo; cat -v; stty sane     # type; see bytes immediately; Ctrl-C prints ^C instead of killing cat. Then type: reset
```
(If that leaves the terminal odd, type `reset` blind and press Enter.)

*Analogy.* Cooked mode is ordering through the innkeeper: you finish your sentence, correct yourself, and only then does the order go to the kitchen. Raw mode is standing in the kitchen: every word reaches the cook as you say it, nobody corrects you, and shouting "stop" is just another word unless the cook decides otherwise.

### 4.4 Alternate screen

**What.** Terminals keep two screen buffers. The *normal* one has scrollback; the *alternate* one is a blank page with no scrollback that `vim`, `less`, and `htop` draw on. Switching in with `ESC[?1049h` saves the cursor and shows the blank page; `ESC[?1049l` restores the normal buffer exactly as it was, so your shell history is not smeared with UI frames.

**Why it matters.** The medallion is a full-screen app; it lives on the alternate screen. Two consequences: nothing you print with `fmt.Println` will be visible (it goes under the UI), and after quitting the user's scrollback is untouched. For debugging, log to a file (`--debug medallion.log`), never to stdout.

**Bubble Tea.** `tea.WithAltScreen()` on `NewProgram`, or `tea.EnterAltScreen` / `tea.ExitAltScreen` commands at runtime. Bubble Tea's `tea.Println` prints *above* the program on the normal screen, useful in non-alt-screen tools, useless here.

**Try.** The `printf '\e[?1049h'` line in 4.2; then run `less /etc/passwd`, quit, and notice your previous output is intact.

*Analogy.* Unrolling a fresh map over the inn's table. You work on the map. When you roll it up, the table underneath is exactly as you left it, tankards and all.

### 4.5 16, 256, and true colour

**What.** Three generations. The 16 ANSI colours (`30–37`, `90–97`) are *names*; the terminal's theme decides what "red" looks like. The 256-colour palette (`38;5;n`) is a fixed 6×6×6 colour cube (16–231) plus 24 greys (232–255) on top of the 16. Truecolor (`38;2;r;g;b`) is 24-bit. Support is advertised badly: `COLORTERM=truecolor` or `24bit` is the convention, `TERM=xterm-256color` says 256, and `NO_COLOR` set means "none, please". Apple's Terminal.app does not support truecolor; iTerm2, Ghostty, kitty, WezTerm, Alacritty, and modern GNOME Terminal do. Inside tmux, truecolor needs `set -as terminal-overrides ",*:RGB"` in `tmux.conf`, otherwise colours get quantised.

**Why it matters.** The witcher theme wants a moody palette (ember orange for forges, cold blue for portals, sickly green for toxicity). It must degrade gracefully to 256 and to 16, and to *no* colour, without the layout depending on colour to be readable (use glyph fill and labels, not just colour, for the `!` on a hot core).

**Bubble Tea / Lip Gloss.** Lip Gloss detects the profile via `termenv` and downsamples automatically: `lipgloss.Color("#ff7f00")` becomes the nearest 256 or 16 colour when needed. `lipgloss.AdaptiveColor{Light: …, Dark: …}` picks by background. Force a profile in tests with `lipgloss.SetColorProfile(termenv.Ascii)`. **Verify:** v1's background detection queries the terminal; call it before `tea.NewProgram` starts reading stdin, or it can hang. v2 replaces this with a message.

**Try.**
```sh
echo "$TERM $COLORTERM"
for i in $(seq 0 255); do printf '\e[48;5;%dm  ' "$i"; [ $(( (i+1) % 32 )) -eq 0 ] && printf '\e[0m\n'; done
NO_COLOR=1 ./medallion       # must still be readable
```

*Analogy.* A painter with a full set of pigments versus one with sixteen. The good painter composes so the picture still reads in sixteen, or in charcoal. Design the theme in charcoal first, then add colour.

### 4.6 Unicode: box drawing, blocks, and wide characters

**What.** The terminal is a grid of *cells*. A character occupies one cell, or two (East Asian wide characters, most emoji), or zero (combining marks, zero-width joiners). Width comes from Unicode's East Asian Width property (UAX #11); characters marked *ambiguous* (many arrows and geometric shapes such as `▲` and `↑`) are one cell in Western terminals and two in CJK locales. Emoji with variation selectors and ZWJ sequences are where terminals disagree most. Go's `len(s)` is bytes; `utf8.RuneCountInString` is code points; neither is cells. Use `lipgloss.Width` (ANSI-aware) or `runewidth.StringWidth`. Grapheme clusters (UAX #29) are what a human sees as "one character"; `github.com/rivo/uniseg` splits them.

Glyphs this project uses, chosen to be one cell everywhere:

| Set | Characters | Use |
|---|---|---|
| Box drawing | `─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼ ╭ ╮ ╰ ╯ ═ ║` (U+2500–U+257F) | panel borders |
| Block elements | `█ ▓ ▒ ░ ▁ ▂ ▃ ▄ ▅ ▆ ▇` (U+2580–U+259F) | bars, sparklines, landscapes |
| Braille | U+2800–U+28FF | dense graphs (stretch) |
| Punctuation | `· » ° ~ ^ / \ _ |` | decoration, in every font |

Not used in layout-critical places: emoji, `▲▼◆✦⚠`, arrows. If they appear at all, only at the end of a line where a width error cannot break a border.

**Why it matters.** One wrong width and every border on that row breaks. Process names come from the system and can contain anything, including CJK and emoji; truncate them by *cell width*, not by bytes, and replace control characters (3.7).

**Bubble Tea / Lip Gloss.** Lip Gloss measures with `ansi.StringWidth`, truncates with `ansi.Truncate`, and pads to the width you ask for. The renderer assumes your `View()` string's lines are the widths you claim.

**Try.**
```sh
printf '│%s│\n' 'abc' '日本語' '🧙' 'a̐b'     # count the cells; the borders drift
locale                                       # LANG must be a UTF-8 locale or box drawing prints as ??? on Linux
```

*Analogy.* Runes carved on a lintel. Some runes are twice as wide. If the mason counts runes instead of hand-widths, the lintel does not fit the door.

### 4.7 Resizing (SIGWINCH)

**What.** When the emulator window changes size, it sets the new size on the PTY master (`ioctl(TIOCSWINSZ)`), and the kernel sends `SIGWINCH` to the foreground process group of that terminal. The program then asks for the size (`ioctl(TIOCGWINSZ)` returns rows, columns, and nominal pixels) and redraws. SSH and tmux forward the signal to the far side.

**Why it matters.** The layout is a function of size. The continent must reflow from 96×30 to 80×24 and refuse politely below that. Layout code that assumes a fixed width is the second most common TUI bug after width miscounts.

**Bubble Tea.** Installs the `SIGWINCH` handler and delivers `tea.WindowSizeMsg{Width, Height}` on start and on every resize. Store it, recompute the `Layout`, and re-size child models (`table.SetHeight`, `viewport.Width`).

**Try.**
```sh
stty size                    # rows cols
watch -n 0.5 stty size       # resize the window while this runs
kill -WINCH $$               # the shell redraws its prompt
```

*Analogy.* The ground shifts and the table shrinks. A good cartographer redraws the map to the table, keeping the important places, dropping the ornaments.

### 4.8 Mouse input

**What.** Mouse reporting is opt-in via the private modes in 4.2: `1000` reports clicks, `1002` adds drag motion, `1003` reports all motion, and `1006` switches to SGR encoding, which is the one you want (the older X10 encoding cannot express coordinates above 223). Reports arrive as escape sequences on stdin. Wheel events are buttons 4 and 5. Once tracking is on, the emulator stops doing native text selection; users hold Shift (or Option/Alt on macOS) to select.

**Why it matters.** Nice to have: click a row to select it, wheel to scroll the chronicle. Never required, and never used to confirm a Sign. Keyboard-first keeps the app usable over slow SSH and inside tmux with odd mouse settings.

**Bubble Tea.** `tea.WithMouseCellMotion()` (modes 1002 + 1006) or `tea.WithMouseAllMotion()` (1003 + 1006). Delivers `tea.MouseMsg` with `X`, `Y`, `Button`, `Action` (press, release, motion), and modifier flags. Coordinates are zero-based cells; you map them to panels using the `Layout`.

**Try.**
```sh
printf '\e[?1000h\e[?1006h'; cat -v; printf '\e[?1000l\e[?1006l'    # click around; Ctrl-C; then the last printf
```

*Analogy.* The medallion responds to a touch, but a witcher does not need to touch it to know it is vibrating.

### 4.9 Two more you will meet

- **Terminfo and `$TERM`.** The database (`infocmp`, `/usr/share/terminfo`) describes each terminal's dialect. Bubble Tea mostly assumes xterm-compatible behaviour, which is what every modern emulator speaks. `TERM=dumb` and the Linux virtual console (`tty1`, 16 colours, limited fonts) are the cases where you need the ASCII theme.
- **Keyboard protocols.** Legacy input cannot distinguish Ctrl-I from Tab, or report key release. The kitty keyboard protocol fixes that; Bubble Tea v2 supports it. Not needed for this project; good to know why `ctrl+shift+…` bindings are unreliable.

---

## 5. Linux fundamentals

Same format as section 4. Commands are for the Linux VM.

### 5.1 `/proc` and `/sys` as virtual filesystems

**What.** Neither is on disk. `procfs` (mounted at `/proc`) and `sysfs` (at `/sys`) are kernel interfaces that *look like* files so that ordinary tools (`cat`, `open`, `read`) can query and sometimes set kernel state. Reading `/proc/loadavg` runs a kernel function that formats the current load average into your buffer. Files report size 0 because the content is generated on read. `/proc/[pid]/` is one directory per process (thread group leader); `/proc/self` is a symlink to your own. `/proc/[tid]` for a thread is not listed by `readdir` but exists if you name it. `/sys` is the device model: one directory per device, one attribute per file, organised by class (`/sys/class/thermal`, `/sys/class/net`, `/sys/class/power_supply`) and by bus. Under `/sys`, one value per file is the rule; under `/proc`, formats vary by file and are documented in `proc(5)` (split into `proc_*.5` pages in recent man-pages) and in the kernel's `Documentation/filesystems/proc.rst`.

**Why it matters.** This is the whole data layer. Everything on the main view is a file read. The formats are stable ABI: the kernel promises not to break them, which is why 25-year-old scripts still work.

**Try.**
```sh
ls /proc/$$                        # your shell's directory
cat /proc/$$/status                # human-readable key: value
cat /proc/$$/stat                  # one line, 52 fields, machine-readable
ls -l /proc/$$/exe /proc/$$/cwd    # symlinks
cat /proc/loadavg /proc/uptime
ls /sys/class/thermal/ /sys/class/power_supply/ /sys/class/net/
cat /sys/class/net/*/operstate
mount | grep -E ' /proc | /sys '   # note hidepid= if present
```

*Analogy.* The notice board in the village square. Nobody stores the notices; the crier writes each one fresh when you look. The board is always current and never takes up shelf space.

### 5.2 CPU percentage from jiffies

**What.** The kernel counts time in *jiffies*, ticks of the scheduler clock. `/proc/stat` exposes per-CPU counters in `USER_HZ` units (the ABI constant, 100 per second on every mainstream architecture; `getconf CLK_TCK` prints it, and `sysconf(_SC_CLK_TCK)` is the portable way). The `cpu` line:

```
cpu  user nice system idle iowait irq softirq steal guest guest_nice
```

Counters only increase. A percentage is always a ratio of *deltas* between two reads:

```
total  = user + nice + system + idle + iowait + irq + softirq + steal
busy   = total - idle - iowait
cpu%   = 100 * Δbusy / Δtotal
```

Do not add `guest` and `guest_nice`; they are already included in `user` and `nice`. `iowait` is time the CPU was idle *while* some task waited on I/O; `top` counts it as not busy, so do the same. `steal` is time a hypervisor took from this VM (visible in Lima; interesting to show).

Per process, `/proc/[pid]/stat` fields 14 (`utime`) and 15 (`stime`) are jiffies the process spent in user and kernel mode, summed over its threads:

```
proc% = 100 * (Δutime + Δstime) / CLK_TCK / Δseconds      # share of one core; 9 threads can show 900%
```

Divide by `NumCPU` for a share of the whole machine. Use the wall-clock delta you measured, not the nominal interval.

Age: field 22 (`starttime`) is jiffies since boot, so `start = btime + starttime / CLK_TCK`.

**Why it matters.** The forges. Also the first place where "one snapshot is not enough" becomes concrete; the domain's two-snapshot design (2.4) exists because of this.

**Try.**
```sh
getconf CLK_TCK
head -1 /proc/stat; sleep 1; head -1 /proc/stat     # subtract by hand once; you will never forget it
yes > /dev/null &  sleep 2; cat /proc/$!/stat | cut -d' ' -f14,15; kill %1
```

*Analogy.* You cannot tell how hard a forge is working from one glance at the anvil. You count hammer strokes for a minute, then again for the next minute, and compare.

### 5.3 Process lifecycle: fork, exec, wait, zombies, orphans, PID 1

**What.**

```
   parent ──fork()──► child (a copy: same code, same open files, new PID)
                        │
                        ├──execve("/usr/bin/go")──► same PID, new program image
                        │
                        ├── runs …  R (running/runnable)  S (sleeping)  D (uninterruptible)
                        │
                        └──exit()──► Z (zombie): only the exit status remains
                                        │
   parent ──wait()/waitpid()/waitid()───┘  reaps: the entry disappears
```

- `fork()` (today `clone()` under the hood) duplicates the calling process. `/proc/stat`'s `processes` line counts forks since boot.
- `execve()` replaces the program image in place. PID, parent, and most file descriptors survive; memory, code, and signal handlers do not (ignored signals stay ignored; caught ones reset to default, which matters for the `wards` display).
- `exit()` frees almost everything but leaves a *zombie*: a process table entry holding the exit status for the parent to collect. Zombies use no memory or CPU; they occupy a PID. You cannot kill a zombie, it is already dead. The fix is for the parent to `wait()`; killing the parent works because the zombie is then reparented and reaped.
- An *orphan* is a live process whose parent died. It is reparented to PID 1, or to the nearest ancestor that declared itself a *subreaper* (`prctl(PR_SET_CHILD_SUBREAPER)`; `systemd --user` does this). PID 1 reaps everything it inherits.
- PID 1 (`systemd` on most distributions) is special: it is started by the kernel, it must never exit (the kernel panics), and signals it has no handler for are ignored, including `SIGKILL` from userspace.
- Kernel threads (`[kworker/0:1]`, `[ksoftirqd/2]`) are children of `kthreadd` (PID 2), have `PF_KTHREAD` set, and have no `cmdline`. `ps` shows them in brackets.
- Threads share a thread group; `/proc/[pid]/task/[tid]` lists them. The process's `stat` sums their CPU time.
- Sessions, process groups, and the controlling terminal are how the shell's job control works: Ctrl-Z sends `SIGTSTP` to the *foreground process group* of the terminal; closing the terminal sends `SIGHUP` to the session. This is why `nohup` and daemons ignore `SIGHUP` (the `H` ward).

State letters in `stat` field 3: `R` runnable, `S` interruptible sleep, `D` uninterruptible sleep (usually disk or network I/O; signals queue until it wakes), `T` stopped by a signal, `t` stopped by a tracer, `Z` zombie, `X` dead, `I` idle kernel thread.

**Why it matters.** Wraiths, drowners, golems, trolls, and Vesemir at Kaer Morhen are all this section. So is the rule that Igni fails on a `D` state process and that a wraith cannot be signalled.

**Try.**
```sh
sh -c 'sleep 1 & exec sleep 60' &      # after 1 s: the inner sleep exited, its parent never waits: a zombie
sleep 2; ps -o pid,ppid,stat,comm --ppid $!    # look for Z / <defunct>
kill %1                                  # kill the parent; the zombie is reaped by PID 1
(sleep 60 &) ; ps -o pid,ppid,comm -C sleep   # an orphan: ppid is 1 (or a subreaper)
cat /proc/1/comm; cat /proc/2/comm       # systemd, kthreadd
grep -E '^(State|PPid|Threads)' /proc/$$/status
```

*Analogy.* A boy is born (fork): at first he is his parent's copy, same house, same name. The Trial of the Grasses (exec) remakes him into a witcher; same person, new nature, and not all survive it. When a witcher dies with his contract unsettled (exit without wait), a wraith lingers on the road until the one who sent him settles the account (`wait`). Orphans are brought to Kaer Morhen (PID 1), where Vesemir takes them in and, eventually, buries them.

### 5.4 Signals

**What.** A signal is a one-bit asynchronous message from the kernel or another process. Each has a *default action* (terminate, terminate with core dump, ignore, stop, continue) and, except for `SIGKILL` and `SIGSTOP`, a process may *catch* it (install a handler), *ignore* it, or *block* it temporarily. Per-process masks are visible in `/proc/[pid]/status`: `SigCgt` (caught), `SigIgn` (ignored), `SigBlk` (blocked), `SigPnd`/`ShdPnd` (pending). Bit `n-1` corresponds to signal `n`.

The five the medallion casts:

| Signal | Number (x86-64, arm64) | Catchable | Default | The Sign |
|---|---|---|---|---|
| `SIGTERM` | 15 | yes | terminate | Aard: "please leave"; well-behaved programs clean up and exit |
| `SIGKILL` | 9 | **no** | terminate | Igni: the kernel destroys the process; no cleanup runs; pipes and children are left as they are |
| `SIGSTOP` | 19 | **no** | stop | Yrden: frozen in place; still holds memory, files, and locks |
| `SIGCONT` | 18 | yes | continue | breaking Yrden; a stopped process always resumes on `SIGCONT`, handler or not |
| `SIGTSTP` | 20 | yes | stop | the weaker Yrden (Ctrl-Z); shown in the bestiary, not cast by the app |

Delivery rules that shape the app's behaviour:

- `kill(2)` permission: the caller must be root (`CAP_KILL`), or its real or effective UID must match the target's real or saved-set UID. Otherwise `EPERM` (the *higher vampire*). `ESRCH` means no such process.
- Signal 0 is a probe: `kill(pid, 0)` checks existence and permission without sending anything. Useful, but the identity check must still compare start times.
- A process in `D` state does not receive signals until it wakes; some kernel waits are `TASK_KILLABLE`, so `SIGKILL` sometimes works anyway. Show "the beast is beneath the surface" rather than promising.
- `SIGKILL` and `SIGSTOP` cannot be caught, blocked, or ignored. That is the whole difference between Aard and Igni.
- PID reuse: PIDs wrap at `/proc/sys/kernel/pid_max` (often 4194304 on modern systems, 32768 on older). Between reading the table and sending the signal, the target may have exited and a new process may have taken its PID. Compare `starttime` before acting, or use `pidfd_open(2)` (kernel 5.3) plus `pidfd_send_signal(2)` (5.1), which pin the identity.
- Signals to a whole group: negative PIDs in `kill(2)` target a process group. Not used by the app; worth knowing when you wonder why Ctrl-C killed a pipeline.

**Try.**
```sh
sleep 300 &
kill -STOP %1; ps -o pid,stat,comm -p $!      # stat T
kill -CONT %1; ps -o pid,stat,comm -p $!      # stat S
grep -E '^Sig(Cgt|Ign)' /proc/$!/status        # sleep catches nothing
grep -E '^Sig(Cgt|Ign)' /proc/$$/status        # your shell catches plenty
kill -TERM $!; sleep 0.2; ps -p $! || echo gone
kill -l                                        # the full list with numbers
```

*Analogy.* Aard knocks the beast down; a strong one gets up, some are trained to roll with it. Igni is not a message; it is a fact. Yrden holds anything, but the held thing still occupies the road (memory, locks, open files) until the trap is broken.

### 5.5 Nice and the scheduler

**What.** `nice` is a hint to the CFS/EEVDF scheduler about a process's share of CPU when there is contention: `-20` is greediest, `19` is most yielding, `0` default. Each step is roughly a 1.25× change in weight, so `nice 10` versus `nice 0` is about a 10:1 share under contention. Nice has *no effect* when the CPU is idle; a niced process alone on a core still gets the whole core. `setpriority(PRIO_PROCESS, pid, nice)` sets it; `/proc/[pid]/stat` field 19 shows it (field 18 is the kernel's internal priority, `20 + nice` for normal tasks, lower numbers for real-time).

Permission: an unprivileged process may only *raise* the nice value (make it more yielding) of its own processes, and can never lower it back below what it was, unless `RLIMIT_NICE` permits (`ulimit -e`) or it has `CAP_SYS_NICE`. **Verify:** Go's `unix.Getpriority` returns the raw kernel value, `20 - nice`, without glibc's translation; read `nice` from `stat` instead and only use `Setpriority`, which takes the nice value directly.

Real-time policies (`SCHED_FIFO`, `SCHED_RR`) and deadline scheduling exist above the nice scale. Not for this app; the scholar overlay can mention when a process has one (`/proc/[pid]/sched`, or `chrt -p PID`).

**Why it matters.** Axii. And the lesson that a "friendlier" process is not slower unless someone else wants the forge.

**Try.**
```sh
nice -n 10 yes > /dev/null & yes > /dev/null &        # two hogs on one core: compare %CPU in top
renice -n 15 -p $!                                    # allowed: raising
renice -n 0 -p $!                                     # EPERM without root: lowering
ulimit -e                                             # RLIMIT_NICE, usually 0
kill %1 %2
```

*Analogy.* Axii sways a will. A witcher can persuade a brute to step aside for others (raise nice). Persuading a peasant to fight harder than he wants needs more than a Sign; that takes Elder Blood.

### 5.6 The OOM killer

**What.** Linux *overcommits* memory: `malloc` succeeds even when the promises exceed physical RAM plus swap, because most programs never touch everything they ask for. When a page must be allocated and nothing can be reclaimed (page cache dropped, swap full), the kernel invokes `out_of_memory()`. It picks one process by *badness*: roughly resident memory plus swap plus page tables, as a per-mille of available memory, plus `oom_score_adj` (range −1000 to 1000, where −1000 makes it unkillable). The victim gets `SIGKILL`, and a line like this goes to the kernel log:

```
Out of memory: Killed process 41872 (go) total-vm:2097152kB, anon-rss:1843200kB, file-rss:0kB, shmem-rss:0kB, UID:1000 pgtables:3720kB oom_score_adj:0
```

Observation points: `/proc/[pid]/oom_score` (the current badness, read-only), `/proc/[pid]/oom_score_adj` (the knob), `/proc/vmstat` line `oom_kill` (a global counter, kernel ≥ 4.13), `/proc/pressure/memory` (PSI: how much time tasks spend stalled on memory; rising `some avg10` is the sky darkening), and `/proc/sys/vm/overcommit_memory` (0 heuristic, 1 always, 2 never). With cgroup v2, each cgroup can have its own `memory.max` and its own OOM events (`memory.events` `oom_kill`); systemd's `systemd-oomd` and `earlyoom` are userspace hunters that act earlier, on PSI.

Permission on the knob: `/proc/[pid]/oom_score_adj` is writable by the process's owner and root. Raising the value (more killable) is always allowed. Lowering it below the process's `oom_score_adj_min` (the lowest value ever set with privilege, 0 for a normal process) needs `CAP_SYS_RESOURCE`.

**Why it matters.** The Wild Hunt, Quen, and the honest answer to "why did my process disappear at 3 a.m.".

**Try.**
```sh
cat /proc/$$/oom_score /proc/$$/oom_score_adj
echo 500 > /proc/$$/oom_score_adj; cat /proc/$$/oom_score   # allowed
echo -500 > /proc/$$/oom_score_adj                          # EACCES without CAP_SYS_RESOURCE
grep oom_kill /proc/vmstat
cat /proc/pressure/memory
journalctl -k | grep -i 'out of memory'
```
To *see* the Hunt in the VM (with a small VM and swap off): `python3 -c 'a=bytearray(10**10)'` or `stress-ng --vm 2 --vm-bytes 95% -t 20s`. Do it in the VM, not on the Mac.

*Analogy.* The Hunt rides when the sky goes dark. It takes one rider, chosen by how heavy the sack he carries is. Quen makes the riders pass you by; only a sorceress of the old blood can cast a Quen strong enough to be sure.

### 5.7 Permissions: what needs root and why

**What.** The process's identity is a tuple of UIDs (real, effective, saved, filesystem: the four numbers on the `Uid:` line of `status`) plus groups plus *capabilities*, the 41 fine-grained slices of root (`capabilities(7)`). Unprivileged reads are allowed for anything the kernel considers non-sensitive; the sensitive parts of `/proc/[pid]` are gated by *ptrace access mode*: you may read them if you could attach a debugger to that process, which means same UID (and the process is *dumpable*), or `CAP_SYS_PTRACE`.

| Operation | Needs | Capability |
|---|---|---|
| `/proc/[pid]/{stat,status,comm,cmdline,statm,oom_score,cgroup}` for any process | nothing | |
| `readlink /proc/[pid]/{exe,cwd,root}`, `/proc/[pid]/{fd,environ,maps,io,smaps}` | same user, or | `CAP_SYS_PTRACE` |
| `/proc/[pid]/stack`, `/dev/kmsg` (if `dmesg_restrict=1`) | root, or | `CAP_SYSLOG` for kmsg |
| Signal another user's process | | `CAP_KILL` |
| Lower nice, or renice another user's process | | `CAP_SYS_NICE` |
| Lower `oom_score_adj` below the minimum | | `CAP_SYS_RESOURCE` |
| Read the system journal | membership in `systemd-journal`, `adm`, or `wheel` | |
| `/sys/class/{thermal,hwmon,power_supply,net}` reads | nothing | |
| `/proc/net/*` reads | nothing (but socket → PID needs the fd scan, so same user) | |
| `hidepid=2` on `/proc` | hides other users' processes entirely; the board shows only yours | |

Ways to get partial Elder Blood without running the whole app as root, in order of preference:

1. Do without. The core experience never needs it (section 6).
2. `sudo medallion` for a session where you want it. The app detects `EUID == 0` and warns.
3. File capabilities on the binary: `sudo setcap cap_kill,cap_sys_nice,cap_sys_ptrace+ep ./medallion`. The binary then has exactly those gifts, for anyone who can run it. Fine on a personal machine; understand that it is a privilege grant before you do it, and that it is lost on every rebuild (make it a Makefile target).

**Try.**
```sh
id; grep -E '^(Uid|Gid|Cap)' /proc/$$/status
capsh --decode="$(grep CapEff /proc/$$/status | cut -f2)"       # names the bits
ls -l /proc/1/exe                # EACCES as a normal user
cat /proc/1/status | head -5     # allowed
groups                           # is systemd-journal or adm in there?
journalctl -n 3 --no-pager       # system journal or only your user journal?
```

*Analogy.* Anyone may read the notice board. Reading a beast's mind (its memory, its open doors) needs to be its kin (same user) or a sorceress (`CAP_SYS_PTRACE`). Hurting another's beast needs the Elder Blood, and even then the wise witcher asks whether he should.

---

## 6. Safety

The medallion is a monitoring tool that can also hurt things. The rules below are design constraints, not features; the roadmap treats them as part of the definition of done for M6 and every milestone after.

### 6.1 Read-only by default

- The only writes to the system are the three actions in `act.Caster`: `kill(2)`, `setpriority(2)`, and a write to `/proc/[pid]/oom_score_adj`. Everything else is `O_RDONLY`. Grep the code for `os.WriteFile`, `os.OpenFile`, and `unix.` calls in review; they belong in `act` and in the config/state writers only.
- `--no-signs` (or `signs.enabled = false`) removes the Sign key bindings and the SIGNS panel entirely. The app is then incapable of writing to the system. This is the default when connected to a remote source (stretch) and the recommended mode for demos.
- Files the app writes for itself: `$XDG_CONFIG_HOME/medallion/config.toml` (never written by the app, only read), `$XDG_STATE_HOME/medallion/casts.log` (append only), and an optional `--debug` log. Nothing else.

### 6.2 Every Sign asks

Confirmation is decided by the domain (`Plan.Confirm`), rendered by the UI, and never skipped.

| Situation | Confirmation |
|---|---|
| Aard, Yrden, break Yrden, Axii (raising nice), Quen (weakening) on your own ordinary process | `Simple`: a dialog with the plan, Enter to cast, Esc to sheathe |
| Igni (SIGKILL), any Sign | `TypeName`: type the process's `comm` exactly |
| Any Sign on a process with children | `TypeName`, and the dialog lists the children count and what happens to them (orphaned, reparented; not killed) |
| Any Sign while running as root, or on another user's process with `CAP_KILL` | `TypeName` |
| Axii lowering nice, Quen strengthening (privileged) | `TypeName` |

Dialog contents, always: the mechanism in plain words (`SIGKILL`), the target's PID, name, user, age, and command line, the plan's verdict and reason ("this process catches SIGTERM; it may ignore Aard"), the wards, and the lineage. Keyboard only. Esc always cancels. The dialog ignores the confirm key for 300 ms after opening so that a held key cannot confirm. If the target vanishes while the dialog is open, it closes with "the beast is already gone".

### 6.3 Protected processes (the golden dragon)

Never castable, regardless of configuration or privilege:

- PID 1 and PID 2, and every kernel thread (`PF_KTHREAD`).
- The medallion itself, and its ancestors up to PID 1: the shell, tmux, the terminal emulator, the SSH session. Computed once at start by walking `PPid` from `/proc/self/status`, and refreshed if the tree changes. Killing your own terminal from inside the app is a mistake nobody should be able to make.
- Zombies: nothing to signal. The UI says so and points at the sire.

Castable only after the `TypeName` ladder and a specific warning, configurable via `wards.protect_comm` (globs matched against `comm`): `sshd`, `systemd*`, `dbus-*`, `tmux: server`, display servers and compositors (`Xorg`, `gnome-shell`, `sway`, `Hyprland`, `kwin_wayland`), `login`, `agetty`, `containerd`, `dockerd`. The default list errs on the side of caution; the user can shorten it.

Quen on the medallion's own process is not offered; the app must not make itself immune to the Hunt.

### 6.4 Never require root

- Every panel on the Continent reads world-readable files. Sensors, battery, network counters, sockets, the process table, memory, load: all unprivileged. Missing privilege degrades a field to "unknown" with an explanation in the scholar overlay, never to an error screen.
- If `EUID == 0` or `CapEff` has `CAP_KILL`, the header shows *Elder Blood* and the confirmation ladder tightens (6.2). The app never prints "try sudo". The overlay explains which capability a greyed-out action needs and why, once.
- `setcap` on the binary (5.7) is documented as an option with its trade-off, not recommended as the default.

### 6.5 Identity, races, and staleness

- Every action re-verifies `ProcessID` (PID plus `starttime`) immediately before the syscall, and uses `pidfd_open` + `pidfd_send_signal` where available so that the identity is pinned by the kernel.
- The confirmation dialog triggers a fresh `Selected` collection; it shows data no older than one tick.
- One cast per second, app-wide. There is no "cast on all matching" in v1.0.
- Sign keys are single letters (`a`, `i`, `y`, `Y`, `x`, `q` in the contract screen) with no modifiers, and are only bound on the contract screen, never on the notice board. You cannot kill something you have not opened and looked at.

### 6.6 Auditing

Every attempted cast appends one line to `casts.log`: timestamp, action, mechanism, PID, `starttime`, `comm`, user, verdict, result (`ok`, `EPERM`, `ESRCH`). The same line appears in the chronicle as a *medallion* entry. Errors are shown in the app in the theme's words with the raw `errno` in the overlay.

### 6.7 Rendering and input hygiene

- All strings from the system are sanitised before rendering (3.7). Terminal injection via a process name is a real attack surface for any TUI process viewer.
- No shell interpolation anywhere. `exec.Command("journalctl", args...)`, never `sh -c` with data in it.
- The app never reads process memory, executables, or environment unless the user opens that view for their own process. It never follows `exe` symlinks to open the binary.

### 6.8 The app's own footprint

Target: under 2% of one core and under 50 MiB RSS with 500 processes at default refresh rates. A monitoring tool that becomes the load is a bad joke. The `--interval` flag scales every cadence; on battery power the default doubles (stretch).

---

## 7. Roadmap, worked backwards

Start at the end. Each milestone below states what v1.0 needs from it, so that the reason for the work is always visible. Effort estimates are for evenings and weekends and are rough; the order matters more than the dates. Read this section top to bottom for the plan, bottom to top for the order of work.

```
 v1.0  release ◄── M9 alerts ◄── M8 roads/archives/portals ◄── M7 chronicle ◄── M6 signs
                                                                                    ▲
 M0 setup ──► M1 the medallion stirs ──► M2 naked terminal ──► M3 continent ──► M4 board ──► M5 contract
```

### v1.0 — The Witcher's Medallion

**Goal.** A release another person could install on a Linux laptop or server, run without root, and learn something from.

**Definition of done**

- [ ] All four screens from section 1 (continent, contract, chronicle, bestiary/help) plus the scholar overlay.
- [ ] Both themes (`witcher`, `plain`) and the `--ascii` glyph set; usable at 16 colours and with `NO_COLOR`.
- [ ] Layout correct from 80×24 to 200×60; a polite placeholder below 80×24.
- [ ] No panic on: a VM without sensors or battery, a container, `hidepid=2`, a machine with 2000 processes, a kernel without PSI, `TERM=xterm` over SSH, inside tmux.
- [ ] `--replay` demo scenarios ship in the repo; README screenshots are made from them.
- [ ] `--no-signs` and `--fake-signs`; casts logged; protected list enforced; integration tests for every Sign against child processes on CI.
- [ ] Static binaries for `linux/amd64` and `linux/arm64` built by GoReleaser; `go install` works.
- [ ] `go vet`, `staticcheck` clean; parsers and domain above 80% test coverage; CI green.
- [ ] Own footprint within 6.8.
- [ ] README (what, why, how to run, how to read the screen, safety), CHANGELOG, this document updated to match reality.

**You will learn.** Releasing Go software (GoReleaser, versioning with `-ldflags -X`), writing documentation for people who did not read the books, and the discipline of finishing.

**Rough effort.** One week of evenings, after M9.

### M9 — The medallion vibrates

**Needed by v1.0 because** the medallion is the product's name and the alert engine is what makes it a monitor rather than a viewer.

**Goal.** Alert rules with sustain and hysteresis (2.7); thermal and power collectors; PSI and the `oom_kill` counter; an alerts strip under the header; chronicle lines when an alert starts and ends; the vibration effect (title shake for one second, panel highlight).

**You will learn.** `sysfs` class devices and the hwmon interface; PSI; the OOM killer from the outside; animation in an Elm loop (a 50 ms `tea.Tick` that is only armed while something animates); hysteresis and why monitoring without it flaps.

**Definition of done**

- [ ] Replay scenarios `memory-pressure`, `thermal`, `oom-kill` produce the expected alerts at the expected ticks (domain tests).
- [ ] Alert appears within one tick of the condition and clears only after the sustain window.
- [ ] A VM without `/sys/class/thermal` or a battery shows "no forge heat known" and "no reserve of Power", with no error.
- [ ] Trip points are read when present; the 85 °C fallback is used otherwise.
- [ ] Vibration stops after one second and costs no CPU when idle (verify with `top` in the VM).
- [ ] The overlay explains each alert's source file and threshold.

**Rough effort.** One week.

### M8 — Roads, archives, portals

**Needed by v1.0 because** network, disk, and sockets complete the Continent; portals are the only way SSH shows up.

**Goal.** `/proc/net/dev` rates and `/sys/class/net` state; `/proc/diskstats` throughput and util with partition filtering; mounts with `statfs`; `/proc/net/{tcp,tcp6,udp,udp6,unix}` parsing; inode → PID mapping for the user's own processes; the roads and portals panel; the contract's "portals" line; a mounts list in the kingdoms screen.

**You will learn.** Counters and rates; sector units; little-endian hex addresses (and the IPv6 variant); why `ss` uses netlink instead (and that you do not need to); pseudo-filesystem filtering; `statfs(2)` and the difference between `f_bfree` and `f_bavail`.

**Definition of done**

- [ ] Rates agree with `ifstat 1` and `iostat -x 1` within 10%.
- [ ] Mount usage agrees with `df -h`; pseudo filesystems (`proc`, `sysfs`, `tmpfs` unless requested, `cgroup2`, `overlay` duplicates) are excluded.
- [ ] Portals agree with `ss -tunp` for the user's own processes; other users' sockets show with "unknown" owner.
- [ ] IPv4 and IPv6 address parsing has table tests including loopback, unspecified, and mapped addresses.
- [ ] A new `LISTEN` socket on a non-loopback address produces the `NewListener` event.

**Rough effort.** One to two weeks.

### M7 — Jaskier's chronicle

**Needed by v1.0 because** the log is where the Wild Hunt's victims are named and where cast results are recorded.

**Goal.** `journalctl` streaming into a bounded ring buffer; the chronicle screen (viewport, filter by priority and unit, search); the app's own events merged in; graceful behaviour when the journal is missing or restricted.

**You will learn.** Running a child process with pipes; goroutines and channels feeding Bubble Tea via the wait-for-activity pattern; context cancellation; streaming JSON; syslog priorities; journald's field names and permissions.

**Definition of done**

- [ ] `logger "the medallion hums"` in the VM appears in the chronicle within one second.
- [ ] Quitting leaves no orphan `journalctl` (`pgrep journalctl` is empty afterwards).
- [ ] A flood (`for i in $(seq 100000); do logger x; done`) neither freezes the UI nor grows memory beyond the ring buffer.
- [ ] No journal, or no permission for the system journal, shows an explanation in the panel; the app runs on.
- [ ] Byte-array `MESSAGE` values are handled.

**Rough effort.** One week.

### M6 — Casting Signs

**Needed by v1.0 because** the Signs are the point of the contract screen, and safety has to be designed in, not added.

**Goal.** The `act` package with Linux and fake implementations; `PlanAction` with verdicts; the confirmation dialog with the ladder from 6.2; the protected list; privilege detection; `casts.log`; `--no-signs`, `--fake-signs`.

**You will learn.** `kill(2)`, `setpriority(2)`, `pidfd_open(2)`; `EPERM` versus `ESRCH`; capabilities and how to read `CapEff`; build tags; `textinput`; a modal in an Elm architecture (a nested model that swallows keys while open).

**Definition of done**

- [ ] Each Sign works on a `sleep 300` you started in the VM, verified with `ps -o stat,ni` and `cat oom_score_adj`.
- [ ] Casting on another user's process shows the higher-vampire message; nothing is sent.
- [ ] PID reuse is simulated in a test with the fake and is refused.
- [ ] Protected processes are not castable, including with `sudo`.
- [ ] Igni requires the typed name; a wrong name does nothing.
- [ ] Integration tests on CI spawn `sleep`, cast Yrden, assert `T`, break it, assert `S`, cast Aard, assert gone.

**Rough effort.** One week.

### M5 — The contract

**Needed by v1.0 because** the detail screen is where all the per-process learning lives, and M6 needs its wards and identity.

**Goal.** The contract screen; `/proc/[pid]/status` parsing (uid, memory, threads, signal masks, capabilities, tracer); wards; threads list; lineage from the PPID map; `cwd` and `exe` with `EACCES` handled; embedded landscapes; the bestiary classification rules; the scholar overlay for per-process fields.

**You will learn.** The `status` file end to end; signal masks as bit sets; capability bits; ptrace access mode; thread groups; `go:embed`.

**Definition of done**

- [ ] Every field shown matches `cat /proc/PID/status` for that PID.
- [ ] Wards are correct: your shell shows "catches Aard", a `sleep` does not, a `nohup` job shows "ignores SIGHUP".
- [ ] Each kind rule has a unit test with a hand-built `Process`.
- [ ] Landscapes render in both themes and in `--ascii`, at 80×24 and 120×40.
- [ ] Other users' processes show "unknown" for lair and true face, not an error.

**Rough effort.** One week.

### M4 — The notice board

**Needed by v1.0 because** the process table is the centre of the Continent and the performance budget is set here.

**Goal.** `readdir /proc` plus `stat`, `comm`, `status` per PID; per-process CPU%; RSS; user lookup with a cache; the table with sorting (cpu, mem, pid, name, age), search, paging, and a selection that survives re-sorting (by `ProcessID`, not row index); kernel threads and zombies displayed; the collection benchmark.

**You will learn.** Parsing `stat` correctly (the `comm` parentheses trap); the per-process CPU formula; PID reuse and why identity is a pair; `bubbles/table`; benchmarks and `pprof`; keeping a 60 fps table steady while data changes underneath it.

**Definition of done**

- [ ] CPU% of a `yes > /dev/null` matches `top` within ±2 points; a 4-thread hog shows ~400%.
- [ ] A 1000-process fixture collects in under 50 ms (benchmark in CI).
- [ ] A process disappearing between `readdir` and `open` is skipped silently.
- [ ] Sorting and searching keep the selection on the same process.
- [ ] Scrolling 2000 rows is smooth; no flicker on data refresh.

**Rough effort.** One to two weeks.

### M3 — The Continent

**Needed by v1.0 because** every later screen is built on this layout, theme, and testing machinery.

**Goal.** The header, three kingdom panels (forges per core, treasury, toxicity), footer; the `Layout` computed from the window size with the 80×24 minimum; the `Theme` interface with `witcher` and `plain`; the scholar overlay for these panels; `record` and `--replay`; the first `teatest` goldens; the `idle` and `build-storm` scenarios.

**You will learn.** Lip Gloss layout and frame sizes; `WindowSizeMsg`; sparklines from block elements; cell-width measurement; golden-file testing; designing a theme as data.

**Definition of done**

- [ ] Goldens at 80×24, 96×30, 160×48 pass in CI with the plain theme.
- [ ] `--theme plain` and `--ascii` render correctly; `NO_COLOR` is respected.
- [ ] `--replay testdata/scenarios/idle` on macOS looks identical to live on the VM.
- [ ] `L` shows path, raw line, formula, and refresh for each panel.
- [ ] Per-core bars agree with `mpstat -P ALL 1`.

**Rough effort.** One to two weeks.

### M2 — The naked terminal (a learning detour)

**Needed by v1.0 because** you will debug rendering problems later, and you cannot debug what you have never seen.

**Goal.** `hack/rawterm/main.go`: about 150 lines, no Bubble Tea. Put the terminal in raw mode with `x/term`, switch to the alternate screen, draw a bordered box with the load average (read from `/proc/loadavg` on Linux, fake on macOS), read keys byte by byte (`q` quits, arrows move a marker), handle `SIGWINCH` with `os/signal` and redraw, restore the terminal on exit and on panic.

**You will learn.** Exactly what Bubble Tea does for you: termios, escape sequences on both directions, the Escape-key ambiguity, `TIOCGWINSZ`, and why a crash in raw mode is so unpleasant.

**Definition of done**

- [ ] Works on macOS and in the VM.
- [ ] Ctrl-C is handled by you (it arrives as `0x03`), not by the kernel.
- [ ] Resize redraws correctly; quitting leaves the shell sane; `stty -a` before and after is identical.
- [ ] You can explain every escape sequence the program emits, from memory.

**Rough effort.** One evening. Keep it in the repo; it is the best explanation of section 4 you will ever have.

### M1 — The medallion stirs (the first weekend)

**Needed by v1.0 because** everything else is built on this skeleton: the module, the parsers, the `FS` seam, the loop, the VM workflow.

**Goal.** Repository skeleton as in 3.7 (only the packages you need); `procfs` parsers for `/proc/loadavg`, `/proc/meminfo`, and `/proc/stat` with fixture files and table tests; domain types `LoadAverage`, `Memory`, `CPUSample` and the busy-percentage delta; the `FS` interface with live and fixture implementations; a minimal Bubble Tea app in the alternate screen that ticks every second and shows three toxicity bars, one "in use" treasury bar, and a total forge percentage; `q` quits; cross-compiled and run in the VM; `--replay` of a one-tick fixture on macOS.

**You will learn.** Go modules and project layout; table-driven tests; the `io/fs`-style seam that makes macOS development possible; the Elm loop and the tick re-arm rule; cross-compiling; jiffies and why one snapshot is not enough.

**Definition of done**

- [ ] `go test ./...` passes on macOS.
- [ ] `make vm-run` shows live bars in the VM; the numbers match `uptime` and `free -m`.
- [ ] Total CPU% roughly matches `top`'s `%Cpu(s)` line while a `yes` runs.
- [ ] `--replay testdata/scenarios/idle` runs on macOS.
- [ ] README has a five-line "how to run"; everything is committed.

**Rough effort.** A weekend, with M0 done on the Friday evening.

### M0 — Sharpening the silver sword (Friday evening)

**Goal.** Go 1.25 or newer installed; Lima (or OrbStack) VM running Ubuntu with systemd; `hack/capture.sh` written and the `idle` scenario captured; a hello-world cross-compiled and run inside the VM; `.gitignore`, `Makefile` skeleton with `test`, `build-linux`, `vm-run`; this document committed.

**Definition of done**

- [ ] `limactl shell default -- uname -a` prints a Linux kernel.
- [ ] `bin/hello-linux-arm64` prints inside the VM.
- [ ] `testdata/scenarios/idle/tick-0001/proc/loadavg` exists and is not empty.
- [ ] `git log` shows the design document.

**Rough effort.** Two hours.

---

## 8. Stretch ideas

None of these are in v1.0. Each is written so that it can be started from the v1.0 codebase without changing the domain.

1. **The medallion's vibration, properly.** A shake (the title shifts one cell left and right for eight frames at 60 ms), a border pulse on the panel that caused the alert, and a colour ramp on the alert strip. Implementation: a `vibrateMsg` tick armed only while an animation is active; frame index in the model; the theme decides what "shake" looks like (the plain theme does nothing). Add `animations = false` to the config for people who dislike motion. Optionally ring the terminal bell (`\a`) on `Crit`, off by default.

2. **A living landscape.** The contract landscape and a small header vignette change with the time of day (dawn, day, dusk, night palettes from the wall clock), with the weather derived from load (rain glyphs when toxicity is high), and with smoke over the forges proportional to CPU. Precompute frames as embedded text; select, do not generate, at render time.

3. **The full bestiary.** Extend the kinds with field notes: for each kind, the commands a scholar would run next (`cat /proc/PID/wchan` for a drowner, `ls -l /proc/PID/fd` for an alghoul, `cat /proc/PID/stack` as root for a golem). Add *trophies*: a history of casts with outcome, rendered as a wall in the help screen.

4. **The chronicle as an event log.** Derive events the journal does not have: a process appeared ("a contract was posted"), disappeared ("fulfilled" or, if we cast Igni, "burned"), changed state to `Z`, a portal opened or closed. Diff consecutive snapshots' `ProcessID` sets. Then *ballad mode*: Jaskier renders events through templates ("Of the go build that devoured three forges, and the witcher who calmed it") with the raw event one key away.

5. **SSH portal, two ways.**
   - *Agent mode*: `ssh host medallion snapshot --stream` prints one JSON `Snapshot` per second on stdout; a local `RemoteSource` decodes them. Same domain, same UI, remote data, read-only. No daemon, no ports, just the one binary on both ends. Signs could be proxied later over the same channel, confirmed locally.
   - *Serve mode*: `github.com/charmbracelet/wish` serves the TUI itself over SSH with public-key auth, so `ssh -p 2222 host` opens the medallion of that host. Forced `--no-signs`. Ten lines of code with Wish's Bubble Tea middleware; a whole chapter of learning about SSH PTY allocation.

6. **Other spheres (containers).** Compare `readlink /proc/[pid]/ns/pid` with your own; processes in another PID namespace are in another sphere. Group them by `/proc/[pid]/cgroup` (`/system.slice/docker-<id>.scope`) and show each container as a small world touching ours. Also the honest memory numbers for a containerised medallion: `/sys/fs/cgroup/memory.max` and `memory.current` rather than `/proc/meminfo`.

7. **Guilds and orders.** Group processes by systemd unit from `/proc/[pid]/cgroup`, with the unit's own memory and CPU from `/sys/fs/cgroup/<path>/{memory.current,cpu.stat}`. This is where cgroup v2 becomes concrete.

8. **Watching a contract.** Mark a process; when it exits, the medallion says "contract fulfilled" with its final age. Implementation via `pidfd_open`: the descriptor becomes readable when the process exits, so a goroutine can `poll` it and send a message the instant it happens, instead of waiting for the next tick.

9. **Meditation.** Witchers meditate to recover. A key that freezes sampling and rendering so you can read a busy screen, and a low-power mode (all cadences doubled) that engages on battery.

10. **Places of Power.** CPU affinity (`Cpus_allowed_list` in `status`), NUMA nodes (`/sys/devices/system/node`), and which forge a process has been favouring (histogram of `processor`, field 39).

11. **Braille graphs.** A "kingdoms" screen with per-core history at four times the resolution of block sparklines, using U+2800–U+28FF.

12. **`medallion snapshot --json`** as a scripting interface. It falls out of the record machinery and makes the domain reusable from other tools.

---

## 9. Risks and common pitfalls

Grouped by where they bite. Each has the mitigation that this design already assumes.

### Rendering

| Pitfall | Mitigation |
|---|---|
| **Flicker on refresh.** Content that changes width between frames makes the renderer repaint whole regions; a full clear each frame flickers on slow terminals. | Fixed-width columns, every panel padded to its box; never clear the screen yourself; let Bubble Tea's line diff work. Consider synchronized output (`ESC[?2026h`) if the terminal supports it. **Verify** whether your Bubble Tea version emits it. |
| **Width miscounts** break borders. | The glyph budget (4.6); measure with `lipgloss.Width`; truncate by cells; goldens at three sizes. |
| **Lip Gloss `Width` excludes borders.** Panels come out two cells wider than planned. | Budget with `GetHorizontalFrameSize()`; test at 80 columns first. |
| **Colour-only meaning.** A red `!` is invisible with `NO_COLOR` or to a colour-blind reader. | Every state has a glyph or a word; colour is reinforcement. |
| **Terminal injection** via process names. | Sanitise all system strings (3.7). |
| **Background colour detection hangs** or eats input. | Call it before the program starts, or use `AdaptiveColor` conservatively; v2 fixes it with a message. |

### Performance

| Pitfall | Mitigation |
|---|---|
| Collecting in `Update`. | Never. Collect in a `Cmd`; drop ticks if one is in flight. |
| Reading `status` and `smaps` for every process every second. | `stat` + `comm` + `status` for the table at 2 s; `smaps`-class files never; the selected process at 1 s. |
| Allocations in parsers causing GC pauses on the render goroutine. | Parse into preallocated slices; benchmark; profile with `pprof` before optimising. |
| A journal flood re-rendering per line. | Batch lines per message; ring buffer; viewport only renders the visible slice. |
| 60 fps re-rendering when nothing changed. | Bubble Tea only renders after a message; do not send messages for nothing (no 50 ms tick unless animating). |

### Terminal compatibility

| Pitfall | Mitigation |
|---|---|
| Apple Terminal.app: no truecolor; ambiguous glyph widths. | 256-colour palette designed first; glyph budget; test there anyway. |
| tmux: colours quantised, mouse odd, `TERM=screen`. | Document `terminal-overrides ",*:RGB"`; keyboard-first; test inside tmux. |
| SSH latency makes each frame visibly draw. | Small diffs (stable layout); lower FPS over SSH (`--fps 30`). |
| Linux console (`tty1`): 16 colours, limited font, no box drawing in some fonts. | `--ascii` theme; detect `TERM=linux`. |
| `LANG=C` / non-UTF-8 locale prints box drawing as garbage. | Detect `LC_ALL`/`LC_CTYPE`/`LANG` for `UTF-8`; fall back to ASCII borders. |
| No TTY (`cron`, pipes, `ssh host cmd`). | Refuse with a one-line message; `snapshot --json` for scripting. |
| Escape key delay; Alt-key combos differ across emulators. | Do not bind Alt combos; Esc is only "cancel/back" where a delay is harmless. |

### Correctness

| Pitfall | Mitigation |
|---|---|
| PID reuse between reading and acting. | `ProcessID` = PID + `starttime`; pidfd where available. |
| `CLK_TCK` assumed 100 on an exotic architecture. | `go-sysconf`, recorded in fixtures. |
| Page size differs (16 KiB on some arm64 systems). | `os.Getpagesize()` on the target; recorded in fixtures. |
| `/proc/meminfo` shows host memory inside a container. | Read cgroup v2 `memory.max`/`memory.current` when `/proc/self/cgroup` is not `/` (stretch 6); at least say so in the overlay. |
| Old kernels: no `MemAvailable` (< 3.14), no PSI (< 4.20), no `oom_kill` (< 4.13), fewer `diskstats` fields. | Every optional field is a pointer or has a `Known bool`; parsers ignore missing fields; the overlay says "not on this kernel". |
| `hidepid=2` hides other users' processes. | The table shows what it can; the overlay explains. |
| Suspend and resume produce a huge delta; the clock jumps. | Rates use the monotonic delta; cap or discard deltas over 10× the interval. |
| Counters wrap (32-bit network counters on old kernels). | Treat a negative delta as unknown for that tick. |
| A process vanishes mid-collection. | `ENOENT`/`ESRCH` are not errors. |
| Zombies and `D` state "not dying" after a Sign. | The plan says `Pointless` or `LikelyIneffective` before you cast. |
| `journalctl` JSON `MESSAGE` as a byte array; long lines. | Handle both; bigger scanner buffer. |

### Safety

| Pitfall | Mitigation |
|---|---|
| Held key confirms a cast. | 300 ms guard after the dialog opens. |
| Killing your own shell or terminal. | Ancestor chain is protected (6.3). |
| Running as root by habit. | Elder Blood banner; tighter ladder; never suggested. |

### The project itself

| Pitfall | Mitigation |
|---|---|
| Theme vocabulary leaking into the domain ("just this once"). | Code review rule: `domain` imports nothing of ours; lore words only in `theme/witcher`. |
| Developing only on macOS against fixtures and discovering on the VM that reality differs. | `make vm-run` several times a day; integration tests on CI. |
| Bubble Tea v1/v2 churn. | Pick one at M1, pin it, and do not migrate until v1.0 ships. |
| Scope creep from section 8. | The roadmap is the scope. Stretch ideas go in an `IDEAS.md`, not in the sprint. |
| Fixtures with secrets committed. | No `environ` capture; scrub `cmdline`; review the diff of every new scenario. |

---

## 10. Curated learning resources

Read in roughly this order per topic. Man pages are on the VM (`man 5 proc`) and at man7.org.

### Linux: `/proc`, `/sys`, processes, signals

- `proc(5)` and its split pages in recent man-pages: `proc_pid_stat(5)`, `proc_pid_status(5)`, `proc_meminfo(5)`, `proc_loadavg(5)`, `proc_stat(5)`, `proc_diskstats(5)`, `proc_net(5)`, `proc_pid_oom_score_adj(5)`, `proc_sys_vm(5)`, `proc_vmstat(5)`. https://man7.org/linux/man-pages/man5/proc.5.html
- Kernel documentation for `/proc`: https://www.kernel.org/doc/html/latest/filesystems/proc.html
- I/O statistics fields (`/proc/diskstats`): https://www.kernel.org/doc/html/latest/admin-guide/iostats.html
- Pressure stall information: https://www.kernel.org/doc/html/latest/accounting/psi.html
- OOM and overcommit knobs: https://www.kernel.org/doc/html/latest/admin-guide/sysctl/vm.html
- cgroup v2: https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html
- `sysfs(5)`; hwmon: https://www.kernel.org/doc/html/latest/hwmon/sysfs-interface.html; thermal: https://www.kernel.org/doc/html/latest/driver-api/thermal/sysfs-api.html; power supply: https://www.kernel.org/doc/html/latest/power/power_supply_class.html
- Processes and signals: `fork(2)`, `execve(2)`, `wait(2)`, `exit(2)`, `signal(7)`, `kill(2)`, `sigaction(2)`, `pidfd_open(2)`, `pidfd_send_signal(2)`, `setpriority(2)`, `sched(7)`, `credentials(7)`, `capabilities(7)`, `ptrace(2)` (the "Ptrace access mode checking" section), `namespaces(7)`, `cgroups(7)`, `statfs(2)`.
- Michael Kerrisk, *The Linux Programming Interface*: chapters 6 (processes), 20–22 (signals), 24–27 (process creation and termination), 34 (process groups, sessions, job control), 35 (priorities), 62 (terminals), 64 (pseudoterminals). The single best book for this project.
- W. Richard Stevens, *Advanced Programming in the UNIX Environment*, for the same ground with a different voice.
- Brian Ward, *How Linux Works*, for the whole-system picture in an evening.
- Brendan Gregg, *Systems Performance* and https://www.brendangregg.com/linuxperf.html, for what the numbers mean.
- Julia Evans's zines and posts (https://jvns.ca), especially on `/proc`, signals, and "what happens when you press a key".
- Source to read: `htop` (https://github.com/htop-dev/htop, `linux/LinuxProcessTable.c`), `procps-ng` (`top`, `ps`), `prometheus/procfs` (https://github.com/prometheus/procfs) as a reference parser, `btop` and `bottom` for TUI monitor design.

### Terminals

- Linus Åkesson, *The TTY demystified*: https://www.linusakesson.net/programming/tty/ — read this first.
- Aram Drevekenin, *Anatomy of a Terminal Emulator*: https://poor.dev/blog/terminal-anatomy/
- *Build Your Own Text Editor* (the `kilo` walkthrough): https://viewsourcecode.org/snaptoken/kilo/ — raw mode and escape codes, step by step, in C. M2 is this in Go.
- XTerm Control Sequences (the reference for escape codes): https://invisible-island.net/xterm/ctlseqs/ctlseqs.html
- `termios(3)`, `tty(4)`, `pty(7)`, `pts(4)`, `ioctl_tty(2)`, `console_codes(4)`, `terminfo(5)`, `stty(1)`.
- Synchronized output spec: https://gist.github.com/christianparpart/d8a62cc1ab659194337d73e399004036 (**Verify** the link; search "terminal synchronized output 2026").
- kitty keyboard protocol: https://sw.kovidgoyal.net/kitty/keyboard-protocol/
- Unicode: East Asian Width UAX #11 https://www.unicode.org/reports/tr11/; grapheme clusters UAX #29 https://www.unicode.org/reports/tr29/; box drawing block chart https://www.unicode.org/charts/PDF/U2500.pdf; block elements https://www.unicode.org/charts/PDF/U2580.pdf
- Text-Terminal-HOWTO for the history: https://tldp.org/HOWTO/Text-Terminal-HOWTO.html

### Go and the Charm stack

- Bubble Tea: https://github.com/charmbracelet/bubbletea (README, `tutorials/basics`, `tutorials/commands`, and the `examples/` directory: `realtime`, `send-msg`, `altscreen-toggle`, `window-size`, `mouse`, `table`, `pager`, `split-editors`, `exec`).
- Bubbles: https://github.com/charmbracelet/bubbles (`table`, `viewport`, `textinput`, `help`, `key`, `spinner`, `progress`).
- Lip Gloss: https://github.com/charmbracelet/lipgloss
- `teatest`: https://github.com/charmbracelet/x/tree/main/exp/teatest
- Wish (SSH apps): https://github.com/charmbracelet/wish
- Louis Garman, *Building Bubble Tea programs*: https://leg100.github.io/en/posts/building-bubbletea-programs/ — practical patterns for larger apps.
- `golang.org/x/sys/unix`: https://pkg.go.dev/golang.org/x/sys/unix; `golang.org/x/term`: https://pkg.go.dev/golang.org/x/term
- `io/fs` and `testing/fstest`: https://pkg.go.dev/io/fs, https://pkg.go.dev/testing/fstest; `embed`: https://pkg.go.dev/embed
- Go testing and benchmarking: https://go.dev/doc/tutorial/add-a-test, https://pkg.go.dev/testing#hdr-Benchmarks; `pprof`: https://go.dev/blog/pprof
- Cross-compilation: https://go.dev/doc/install/source#environment (`GOOS`/`GOARCH` table)
- GoReleaser: https://goreleaser.com

### Development environment

- Lima: https://lima-vm.io; OrbStack: https://orbstack.dev; Multipass: https://multipass.run
- GitHub Actions for Go: https://github.com/actions/setup-go

### The books (for names and moods, in reading order)

*The Last Wish*, *Sword of Destiny*, *Blood of Elves*, *Time of Contempt*, *Baptism of Fire*, *The Tower of the Swallow*, *The Lady of the Lake*, and *Season of Storms*. The English translations render Jaskier as Dandelion; the theme should let the user choose. When you want a name for something, look in the books, not the game wikis; the project's premise is the books.

---

## Appendix A: `/proc/[pid]/stat` fields

One line, space-separated, after the `comm` field which is in parentheses and may itself contain spaces and parentheses (split after the *last* `)`). Numbering follows `proc_pid_stat(5)`. Times are in `CLK_TCK` units.

| # | Name | Meaning | Used for |
|---|---|---|---|
| 1 | `pid` | process id | identity |
| 2 | `comm` | executable name, up to 15 chars, in `( )` | name |
| 3 | `state` | `R S D T t Z X I` | state, kinds |
| 4 | `ppid` | parent pid | lineage, orphans |
| 5 | `pgrp` | process group | job control (overlay) |
| 6 | `session` | session id | |
| 7 | `tty_nr` | controlling terminal, 0 if none | daemon detection |
| 8 | `tpgid` | foreground group of the terminal | |
| 9 | `flags` | kernel flags; `PF_KTHREAD = 0x00200000` | kernel threads |
| 10–13 | `minflt cminflt majflt cmajflt` | page faults (own, children) | memory pressure hint |
| 14 | `utime` | user-mode time | CPU% |
| 15 | `stime` | kernel-mode time | CPU% |
| 16–17 | `cutime cstime` | waited-for children's times | |
| 18 | `priority` | kernel priority (`20 + nice` for normal tasks) | |
| 19 | `nice` | −20..19 | Axii |
| 20 | `num_threads` | thread count | |
| 21 | `itrealvalue` | obsolete, 0 | |
| 22 | `starttime` | ticks since boot at start | identity, age |
| 23 | `vsize` | virtual memory, bytes | |
| 24 | `rss` | resident pages (inaccurate for threads; fine here) | memory |
| 25 | `rsslim` | RSS soft limit | |
| 26–28 | `startcode endcode startstack` | addresses (may be 0 without ptrace access) | |
| 29–30 | `kstkesp kstkeip` | 0 on modern kernels | |
| 31–34 | `signal blocked sigignore sigcatch` | obsolete; use `status` `SigPnd/SigBlk/SigIgn/SigCgt` | |
| 35 | `wchan` | address, use `/proc/[pid]/wchan` for the symbol | drowner detail |
| 36–37 | `nswap cnswap` | unmaintained | |
| 38 | `exit_signal` | signal sent to parent on death | |
| 39 | `processor` | CPU last executed on | "last seen on forge N" |
| 40 | `rt_priority` | real-time priority | |
| 41 | `policy` | scheduling policy | |
| 42 | `delayacct_blkio_ticks` | block I/O delays | |
| 43–44 | `guest_time cguest_time` | virtualisation | |
| 45–51 | `start_data end_data start_brk arg_start arg_end env_start env_end` | addresses | |
| 52 | `exit_code` | as reported by `waitpid`, for zombies | wraith detail |

## Appendix B: Signals

Numbers for x86-64 and arm64 (`kill -l` to confirm). "Default" is the action when neither caught nor ignored.

| Signal | # | Default | Catchable | Notes |
|---|---|---|---|---|
| `SIGHUP` | 1 | terminate | yes | terminal hangup; daemons often reload config on it |
| `SIGINT` | 2 | terminate | yes | Ctrl-C from the line discipline (cooked mode) |
| `SIGQUIT` | 3 | core dump | yes | Ctrl-\ |
| `SIGILL` | 4 | core dump | yes | illegal instruction |
| `SIGTRAP` | 5 | core dump | yes | debuggers |
| `SIGABRT` | 6 | core dump | yes | `abort()` |
| `SIGBUS` | 7 | core dump | yes | bad memory access |
| `SIGFPE` | 8 | core dump | yes | arithmetic error |
| `SIGKILL` | 9 | terminate | **no** | Igni |
| `SIGUSR1` | 10 | terminate | yes | application-defined |
| `SIGSEGV` | 11 | core dump | yes | segmentation fault |
| `SIGUSR2` | 12 | terminate | yes | application-defined |
| `SIGPIPE` | 13 | terminate | yes | write to a closed pipe |
| `SIGALRM` | 14 | terminate | yes | timers |
| `SIGTERM` | 15 | terminate | yes | Aard; what `kill` sends by default |
| `SIGCHLD` | 17 | ignore | yes | a child stopped or terminated; the parent should `wait` |
| `SIGCONT` | 18 | continue | yes | breaking Yrden; always resumes a stopped process |
| `SIGSTOP` | 19 | stop | **no** | Yrden |
| `SIGTSTP` | 20 | stop | yes | Ctrl-Z; the catchable Yrden |
| `SIGTTIN` / `SIGTTOU` | 21 / 22 | stop | yes | background job touched the terminal |
| `SIGXCPU` / `SIGXFSZ` | 24 / 25 | core dump | yes | resource limits exceeded |
| `SIGWINCH` | 28 | ignore | yes | terminal resized |
| `SIGSYS` | 31 | core dump | yes | bad syscall; seccomp |

Bit positions in `SigCgt`/`SigIgn`/`SigBlk`: signal `n` is bit `n-1`. `SIGTERM` is `1 << 14`, `SIGHUP` is `1 << 0`.

## Appendix C: Glossary, both directions

| Witcher term | Linux meaning | Witcher term | Linux meaning |
|---|---|---|---|
| the Continent | this host | contract | a process |
| notice board | the process table | beast | the process's `comm` |
| posted by | the process's user | sire | parent process |
| lineage | children | lair | `cwd` |
| true face | `exe` | wards | signals caught/ignored, oom adj |
| hunting | `R` | resting | `S` |
| beneath the surface / drowner | `D` | trapped | `T`, `t` |
| wraith | zombie `Z` | golem | kernel thread |
| troll | long-lived daemon | ghoul pack | swarm of short-lived children |
| griffin | CPU hog | alghoul | memory grower |
| striga | resists SIGTERM | doppler | `comm` ≠ `exe` |
| higher vampire | another user's process | djinn | has capabilities |
| golden dragon | protected process | Vesemir / Kaer Morhen | PID 1 |
| Aard | SIGTERM | Igni | SIGKILL |
| Yrden / break Yrden | SIGSTOP / SIGCONT | Axii | renice |
| Quen | `oom_score_adj` | the Wild Hunt | OOM killer |
| the sky darkens | memory PSI rising | Hunt sightings | `oom_kill` counter |
| Elder Blood | root / capabilities | your steel does not bite | `EPERM` |
| Mahakam forges | CPU cores | forge heat | thermal zones |
| Redanian treasury | RAM | Giancardi loan | swap |
| Oxenfurt archives | disks | shelf space | mount free space |
| scribes | disk I/O | trade roads | network interfaces |
| portals | sockets | portal waiting | `LISTEN` |
| reserve of Power | battery | drawing from the vein | charging |
| toxicity | load average | tolerance | number of cores |
| the Conjunction | boot | days since | uptime |
| Trial of the Grasses | `fork` + `execve` | contract fulfilled | process exited |
| Jaskier's chronicle | journald | the medallion stirs / trembles / vibrates / tugs | Info / Warn / Crit / Emergency |
| scholar overlay | raw data and sources | meditation | pause sampling (stretch) |
| other spheres | namespaces / containers | guilds | cgroups / units |

## Appendix D: Fixture capture script

`hack/capture.sh`. Run inside the Linux VM. Captures one tick of everything the collectors read into a directory that the replay `FS` can serve. Uses `cat` because `/proc` files report size 0 and `cp`/`tar` would produce empty files.

```bash
#!/usr/bin/env bash
# hack/capture.sh — capture one tick of /proc and /sys into a fixture directory.
# Usage: hack/capture.sh testdata/scenarios/idle/tick-0001
set -euo pipefail
out="${1:?usage: capture.sh <output-dir>}"
mkdir -p "$out"

copy() {                       # copy /abs/path -> $out/abs/path (contents via cat)
  local p="$1"
  [ -r "$p" ] && [ -f "$p" ] || return 0
  mkdir -p "$out/$(dirname "${p#/}")"
  cat "$p" > "$out/${p#/}" 2>/dev/null || true
}
link() {                       # store a symlink's target as text in <name>.link
  local p="$1" t
  t="$(readlink "$p" 2>/dev/null)" || return 0
  mkdir -p "$out/$(dirname "${p#/}")"
  printf '%s' "$t" > "$out/${p#/}.link"
}

# --- /proc, host-wide
for f in stat loadavg meminfo uptime vmstat diskstats version; do copy "/proc/$f"; done
for f in cpu memory io; do copy "/proc/pressure/$f"; done
for f in dev tcp tcp6 udp udp6 unix; do copy "/proc/net/$f"; done
for f in hostname osrelease pid_max; do copy "/proc/sys/kernel/$f"; done
mkdir -p "$out/proc/self"; cat /proc/self/mounts > "$out/proc/self/mounts"
grep -E '^(Uid|Gid|CapEff):' /proc/self/status > "$out/proc/self/status"   # our privileges only

# --- /proc, per process (no environ, no fd: privacy)
for d in /proc/[0-9]*; do
  for f in stat status comm cmdline cgroup oom_score oom_score_adj statm; do copy "$d/$f"; done
  link "$d/exe"; link "$d/cwd"
done

# --- /sys
for z in /sys/class/thermal/thermal_zone*; do
  for f in type temp; do copy "$z/$f"; done
  for t in "$z"/trip_point_*_temp "$z"/trip_point_*_type; do copy "$t"; done
done
for h in /sys/class/hwmon/hwmon*; do
  copy "$h/name"
  for f in "$h"/temp*_input "$h"/temp*_label "$h"/temp*_max "$h"/temp*_crit; do copy "$f"; done
done
for p in /sys/class/power_supply/*; do
  for f in type capacity status online energy_now energy_full energy_full_design power_now \
           charge_now charge_full charge_full_design current_now voltage_now cycle_count; do copy "$p/$f"; done
done
for n in /sys/class/net/*; do
  for f in operstate carrier speed address; do copy "$n/$f"; done
done
for b in /sys/block/*; do mkdir -p "$out/${b#/}"; done     # names only: whole-device list

# --- things that are syscalls, not files
{
  echo "captured_at=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
  echo "clk_tck=$(getconf CLK_TCK)"
  echo "page_size=$(getconf PAGESIZE)"
  echo "uname=$(uname -srm)"
} > "$out/meta.txt"
df -B1 --output=target,size,avail -x tmpfs -x devtmpfs -x squashfs -x overlay 2>/dev/null \
  | tail -n +2 > "$out/statfs.txt"

echo "captured $(find "$out" -type f | wc -l) files into $out"
```

A scenario is several ticks:

```bash
# hack/capture-scenario.sh <name> <ticks> [interval-seconds]
name="$1"; ticks="$2"; every="${3:-1}"
for i in $(seq -f '%04g' 1 "$ticks"); do
  hack/capture.sh "testdata/scenarios/$name/tick-$i"
  sleep "$every"
done
```

Before committing a scenario: check its size (`du -sh`), grep `cmdline` files for anything private, and write a one-line `README` in the scenario directory saying what was running and what the scenario is meant to prove.
