# 7. Memory and the VM

## 7.1 The VM

Fale is interpreted, on a VM with a GC, and the VM is written in Zig (D10).

## 7.2 The GC

The GC is generational and fully concurrent, modelled on Java's Generational ZGC (D20).

- New objects go in a small copying nursery with bump allocation; objects that survive move to an old area.
- Both generations are collected while the program runs, so there are no long pauses.
- It needs write barriers (for an old object getting a pointer to a new one) and read barriers (since objects move).
- The VM knows its roots exactly (its own stack, registers and globals), so stack maps aren't needed. Every pointer
  load and store goes through the VM's own code, so the barriers live in one place.

This is the hardest part of the project. Zig doesn't check memory safety at compile time, so GC bugs can slip into
release builds: the GC needs thorough tests (D10).

## 7.3 Sharing

`copy` and `share` (chapter 3) say what a program means; they don't free anything. The GC frees memory (D19).

## 7.4 Inside and outside

Base Fale only computes. Anything that touches the outside world, including the terminal, files, time, randomness,
the network and the program's surroundings, comes from the standard library, and only what the program's manifest allows can be used (D49, D50). Using
something the manifest doesn't allow is refused before running.

## 7.5 Embedding

Fale runs standalone or embedded (D51). Standalone, the `fale` program grants what the manifest lists. Embedded, the
host grants its own abilities. The VM keeps no global state, so several scripts can run side by side.
