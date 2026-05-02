# Capstone-Prateeksha
An Inventory that Accepts input and Stores information
# Hybrid Inventory Manager

A console-based inventory manager that blends a **C data layer** (binary file I/O) with a **C++ UI layer** (STL, classes, menus).

---

## Project Structure

```
hybrid_inventory/
├── include/
│   ├── inventory.h          # Item struct + C function declarations
│   └── InventoryManager.h   # C++ class declaration
├── src/
│   ├── inventory.c          # C backend (fread/fwrite/fseek)
│   ├── InventoryManager.cpp # C++ class implementation
│   └── main.cpp             # Entry point
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run

### Option A – Make (recommended)

```bash
# From the project root
make          # compiles everything into ./inventory
./inventory   # run the app
make clean    # remove build artefacts and inventory.dat
```

### Option B – CMake

```bash
mkdir build && cd build
cmake ..
cmake --build .
./inventory
```

**Requirements:** GCC ≥ 9 (or Clang ≥ 10), C11 + C++17 support, GNU Make or CMake ≥ 3.14.

---

## Features

| # | Feature |
|---|---------|
| 1 | **Add item** – validates ID, name, quantity, price before writing |
| 2 | **View item** – look up a single active record by ID |
| 3 | **Update item** – modify any field of an existing record in-place |
| 4 | **Delete item** – soft delete (sets `is_deleted = 1`); never shows again |
| 5 | **List all** – displays all active items sorted by ID using `std::sort` |

Data persists in `inventory.dat` (binary, same directory as the executable).

---

## Design Notes

* **C layer** (`inventory.c`): uses `fopen/fread/fwrite/fseek/fclose`. Every record occupies `sizeof(Item)` bytes at a fixed offset, so `fseek` jumps directly to any record for O(1) update or delete.
* **C++ layer** (`InventoryManager.cpp`): wraps C functions; uses `std::vector<Item>` to hold listed items and `std::sort` with a lambda to sort by ID before printing.
* `extern "C"` in the header ensures the C++ compiler uses C linkage when calling the C functions.

---

## Test Cases

- **TC-1 – Persistence across restarts**  
  Added items with IDs 1, 2, 3 (Widget, Gadget, Doohickey). Exited with option 6. Re-launched the app and chose *List all*. ✅ All three items appeared.

- **TC-2 – Duplicate ID rejection**  
  While items 1–3 exist, tried to add a new item with ID 2. ✅ App printed `[FAIL] Could not add item (duplicate ID or I/O error)` and the original item 2 remained unchanged.

- **TC-3 – Update persists after restart**  
  Updated item 1 (Widget): changed quantity from 10 to 99 and price from 9.99 to 4.50. Exited and restarted. Chose *View item* → ID 1. ✅ New quantity (99) and price (4.50) were shown.

- **TC-4 – Soft delete hides item**  
  Deleted item 2 (Gadget). Chose *List all* → only items 1 and 3 appeared. Chose *View item* → ID 2 → ✅ `[FAIL] Item with ID 2 not found`. Restarted; same result. ✅

- **TC-5 – Input validation**  
  On *Add item*, entered ID = −5 → re-prompted. Entered empty name → re-prompted. Entered quantity = −3 → re-prompted. Entered price = −1 → re-prompted. ✅ App never crashed; after valid input was provided, the item was stored correctly.
