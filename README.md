# Inventory-management-in-c-by-vedant-wadive
It's a simple inventory managemnt system build in c++
# Inventory Manager

A console application that stores inventory items in a **binary file** (`inventory.dat`) and supports full CRUD via an interactive menu.

- **Data layer** — pure C (`inventory.c`): structs, `fread`/`fwrite`, `fseek` for in-place update/delete.  
- **UI / logic layer** — C++ (`InventoryManager.cpp`): class, `std::vector`, `std::sort`, validated input.

---

## File Structure

```
inventory_manager/
├── include/
│   ├── inventory.h          # Shared C struct + extern "C" API
│   └── InventoryManager.h   # C++ class declaration
├── src/
│   ├── inventory.c          # C backend (binary file I/O)
│   ├── InventoryManager.cpp # C++ menu & logic
│   └── main.cpp             # Entry point
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run

### Using Make (recommended)

```bash
# Build
make

# Run
./inventory_manager

# Build and run in one step
make run

# Clean build artifacts and data file
make clean
```

### Using CMake

```bash
mkdir build && cd build
cmake ..
cmake --build .
./inventory_manager
```

**Requirements:** `gcc` (C11), `g++` (C++17), `make` or `cmake ≥ 3.10`.

---

## Menu

```
  1  Add item
  2  View item by ID
  3  Update item
  4  Delete item
  5  List all items
  6  Exit
```

---

## Item Format (on disk)

| Field        | Type      | Notes                          |
|--------------|-----------|--------------------------------|
| `id`         | `int`     | Positive, unique               |
| `name`       | `char[40]`| Non-empty                      |
| `quantity`   | `int`     | ≥ 0                            |
| `price`      | `float`   | ≥ 0.00                         |
| `is_deleted` | `int`     | 0 = active, 1 = soft-deleted   |

Records are stored back-to-back in `inventory.dat`. Update and delete use `fseek` to modify the correct record in-place without rewriting the whole file.

---

## Test Cases

- **Add + persist**: Added items 1 (Widget A, qty 10, $9.99), 2 (Gadget B, qty 5, $24.50), 3 (Doohickey C, qty 100, $1.25). Exited. Re-ran the program and chose *List all* — all 3 items still appeared. ✅

- **Update persists**: Updated item 2's name to "Gadget B Pro", quantity to 50, price to $29.99. Exited. Re-ran and chose *View item* for ID 2 — new values shown correctly. ✅

- **Soft delete**: Deleted item 3. *List all* showed only 2 items (Widget A, Gadget B Pro); Doohickey C was absent. *View item* for ID 3 returned "not found". ✅

- **Duplicate ID rejection**: Attempted to add a new item with ID 1 (already exists). Received `✗ Failed. The ID may already exist or be invalid.` — no duplicate was written. ✅

- **Input validation**: Entered −5 for quantity → re-prompted. Entered −1.0 for price → re-prompted. Entered 0 and 5.00 respectively → item saved successfully. Program never crashed. ✅

---

## Design Notes

- **`extern "C"` guard** in `inventory.h` allows the C API to be called from C++ without name-mangling issues.
- **Soft delete** keeps the file append-only for new records; deletions are O(n) scans with an in-place flag flip.
- **`std::vector`** buffers up to 1 024 active records for the list view; **`std::sort`** orders them by `id` before printing.
- The C backend exposes exactly the five functions specified: `add_item`, `get_item`, `update_item`, `delete_item`, `list_items`.

