# External Roblox Cheat — Educational README (Rust Edition)

> ⚠️ **EDUCATIONAL USE ONLY**
> This project is published strictly for **learning about game security, reverse engineering, and anti-cheat design**. Using it on live servers violates Roblox's Terms of Service and will result in account bans. The author assumes **zero liability** for misuse. Run only in offline/private environments you own.

---

## Project Name
`roblox-external-edu` — External Memory Read/Write Research Tool (Rust)

## Purpose
A minimal external (out-of-process) memory reader/writer written **entirely in Rust**, built to study:
- How external cheats interact with a target process
- How anti-cheat systems detect these techniques
- The limits of userspace-only, bypass-free approaches

**No bypasses are implemented.** This is intentional — it serves as a baseline for anti-cheat researchers to test detection coverage.

---

## Why Rust?

- **Memory safety** for the cheat's own logic (no UB in the tool itself, only in the target)
- **`windows` crate** provides safe-ish wrappers over Win32 APIs (`OpenProcess`, `ReadProcessMemory`, `WriteProcessMemory`)
- **`egui` / `eframe`** for a lightweight, GPU-accelerated overlay
- **Cargo** for reproducible builds with pinned dependency hashes

---

## Features

| Category | Feature | Description |
|---|---|---|
| Core | Read/Write Memory | Safe wrappers around `ReadProcessMemory` / `WriteProcessMemory` |
| Core | Module Base Resolution | Enumerate modules of target process |
| Core | Pointer Chain Resolver | Follow multi-level offsets |
| Lua VM | Lua State Inspection | Locate and read Lua VM structures (offsets not provided) |

---

## How It Works (High-Level)

1. **Attach** — `OpenProcess(PROCESS_VM_READ | PROCESS_VM_WRITE | PROCESS_VM_OPERATION)` via the `windows` crate.
2. **Resolve** — Locates module base + static offsets for the current build.
3. **Read** — Reads entity lists, positions, camera data into Rust structs.
4. **Render/Act** — Draws overlay (`egui`) or writes values back (aimbot/fly/silent aim).
5. **Lua VM** — Reads Lua state to enumerate scripts/globals (read-only by default).

All of the above are **detectable by design** — see *Detection Surface* below.

---

## Detection Surface (Why No Bypasses)

This project deliberately leaves every common detection vector exposed so researchers can study them:

- **Handle-based detection** — Open handle to the process is visible to kernel-mode AC.
- **`ReadProcessMemory` / `WriteProcessMemory`** — Call frequency and patterns are monitored.
- **Module enumeration** — `CreateToolhelp32Snapshot` / `EnumProcessModules` triggers.
- **Overlay windows** — `WS_EX_LAYERED | WS_EX_TRANSPARENT` topmost overlays are flagged.
- **Write patterns** — Repeated writes to position/CFrame memory are signatured.
- **Lua VM access** — Reading VM memory pages is a known indicator.

**No** handle hiding, **no** syscall stubs, **no** manual mapping, **no** driver, **no** page guard evasion.

---

## Building

```
# Requirements
- Rust 1.79+ (stable, MSVC toolchain)
- Visual Studio 2022 Build Tools (MSVC linker)
- Windows 10/11 SDK
```

```bash
cargo build --release
```

> Offsets are **not** included. You must derive them yourself for your test target. This is a learning exercise, not a turnkey tool.

---

## Ethical & Legal Notes

- Roblox ToS §4 prohibits unauthorized modification of the client. Using this on Roblox results in **permanent bans** and possible legal action.
- Anti-cheat vendors (Hyperion/Byfron, BattlEye, EAC) actively signature external readers. Expect **immediate detection**.
- Publishing working offsets for a live commercial game is **not** endorsed by this project.

## Intended Use Cases

✅ Studying external process memory models
✅ Building anti-cheat detection rules against this baseline
✅ Academic reverse-engineering coursework
✅ CTF / private test harnesses

❌ Live multiplayer cheating
❌ Selling or distributing as a working cheat
❌ Bypassing anti-cheat protections

---

## Contributing

Pull requests that improve **detection research** (e.g., adding logging hooks for AC vendors) are welcome. PRs adding bypasses will be **rejected**.

---

## License

MIT — for educational use. See `LICENSE`.

## Disclaimer

This software is provided "as is", without warranty of any kind. The authors are not responsible for any bans, damages, or legal consequences resulting from use. **You are responsible for how you use this code.**
