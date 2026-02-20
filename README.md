# gbc_emulator

# Building a Game Boy Color Emulator

## Theory

The Game Boy, like all old video game consoles, is essentially a simple computer. The video game cartridges are the programs that run on it and tell it which instructions to execute. An emulator works by virtually representing the physical hardware of the guest machine. To create a Game Boy emulator, you need to simulate how the real hardware functions.

---

## Hardware Overview

The Game Boy has the following hardware components that all need to be emulated:

- 8-bit Central Processing Unit
- 32Kb Work RAM
- 16Kb Display RAM
- Liquid Crystal Display Screen
- Read Only Memory (the game cartridge)
- Joypad Input
- Audio
- Serial Transfer Port

---

## Memory

The Game Boy has several distinct types of memory that each need to be emulated: Work RAM, Video RAM, Cartridge ROM, and External RAM.

Each type of memory can be thought of as a simple block of bytes of a fixed size. Your task is to represent each one in code and implement the ability to read and write individual bytes within them using a 16-bit address, since the Game Boy has a 16-bit address space.

The cartridge ROM is a special case — it is read-only, so it only needs a read method. Rather than initializing it as an empty block, you'll need to load it in from a ROM file on disk.

---

## Memory Mapping

Rather than representing all of the Game Boy's memory as one flat block, the hardware uses memory mapping. This means different address ranges correspond to different physical memory modules. It also enables bank switching, where the data behind an address range can be swapped out at runtime, effectively giving the system more usable memory than the address space alone would allow.

Your job is to implement a central memory bus or interconnect that owns all of the individual memory modules. When something reads or writes to an address, this interconnect is responsible for routing that request to the correct module based on the address range. The key address ranges to map are:

- ROM (cartridge)
- Video RAM
- External RAM
- Work RAM
- Echo RAM (mirrors Work RAM)
- OAM (sprite attribute table)
- High RAM

---

## CPU

The Game Boy CPU is a modified version of the Zilog Z80 and runs at approximately 4.19 MHz. It has eight 8-bit general purpose registers (A, B, C, D, E, F, H, L), a 16-bit stack pointer, and a 16-bit program counter.

Your register file should represent all of these. A useful C++ technique is using unions to allow 8-bit register pairs like B and C to also be accessed together as the 16-bit register BC.

The core of CPU emulation is the fetch/decode/execute cycle. On each step, the CPU reads the byte at the address stored in the program counter, increments the program counter, then decodes and executes the instruction that byte represents. Each instruction also takes a certain number of cycles to complete, which you need to track in order to keep the rest of the hardware in sync.

The Game Boy has hundreds of opcodes. A switch statement dispatching on the opcode byte is a natural way to implement this in C++. Each branch should carry out the instruction's logic and return the number of cycles it took. The Game Boy also has a secondary opcode table prefixed by 0xCB for extended instructions.

---

## Video Display

The Game Boy has a screen resolution of 160x144 pixels and a refresh rate of 60Hz. Since you know the CPU speed and the frame rate, you can calculate exactly how many CPU cycles should elapse per frame.

The screen is drawn line by line from top to bottom. After each line is drawn there is a horizontal blanking period, and after all 144 lines are drawn there is a vertical blanking period before the next frame begins. Your PPU (pixel processing unit) implementation needs to track which mode it is currently in: OAM scan, pixel transfer, H-Blank, or V-Blank. Certain system interrupts are triggered when the mode changes, which games rely on for effects like per-scanline scrolling.

The PPU maintains an internal framebuffer sized to the screen dimensions. At the end of each scanline, the current line is rendered into the framebuffer. Once a full frame has been accumulated, it is passed to your window/renderer to be displayed.

The Game Boy has three video layers, all composed of 8x8 pixel tiles: the background layer, the window layer (typically used for HUD elements like score and lives), and sprites. Each is stored differently in VRAM and rendered slightly differently, but all three need to be composited together to produce the final image.

---

## Joypad Input

The Game Boy has four directional buttons and four general buttons. The game reads their state by reading from a special memory-mapped register. Your joypad implementation needs to track the state of each button and correctly respond when that register is read, returning the appropriate bits depending on whether the game is querying directional or general buttons.

On your host system you'll be reading keyboard input (via SDL2), and your job is to translate those keypresses into updates to the joypad's internal state. A joypad interrupt should be triggered when a button is pressed.

---

## Bringing It All Together

The top-level emulator class should own both the Game Boy object and the window. The main loop should run at the correct speed, stepping the CPU enough times to fill a frame's worth of cycles, then rendering the resulting framebuffer to the screen and reading input, all within the roughly 16.6ms window that a 60Hz frame allows. If your calculations finish early, sleep the remaining time to avoid running faster than real hardware.

---

## Recommended Resources

- **Pan Docs** — the most comprehensive Game Boy hardware reference available
- **The Game Boy Programming Manual** — Nintendo's original developer documentation
- **Blargg's test ROMs** — essential for validating your CPU and hardware timing
- **The emudev subreddit** — a helpful community for when you get stuck

---

Good luck — it's a challenging project but an incredibly rewarding one!
