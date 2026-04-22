
# Military Warehouse Management System

A multi-level military supply chain management system written in Java.  
Refactored from a single-file design into a clean, modular package structure.

---

## Data Structure

```
| Data Structure     | Used In Which Part         | Purpose                                                 |
| ------------------ | -------------------------- | ------------------------------------------------------- |
| **HashMap**        | Password storage           | Stores username → encrypted password                    |
| **HashMap**        | Login attempts             | Stores wrong attempts count                             |
| **HashMap**        | Lock system                | Stores account lock time                                |
| **HashMap**        | Warehouse Levels           | Stores warehouse name → level (Battalion, Brigade etc.) |
| **HashMap**        | Manager IDs                | Stores warehouse → manager ID                           |
| **HashMap**        | Warehouses Database        | Stores warehouse name → warehouse object                |
| **HashMap**        | Graph Representation       | Stores node → connected edges                           |
| **HashMap**        | Inventory inside Warehouse | Stores item name → ItemStock                            |
| **ArrayList**      | Pending Requests           | Stores all supply requests                              |
| **ArrayList**      | Pending Deliveries         | Stores all delivery orders                              |
| **ArrayList**      | Graph Edges                | Stores connected nodes and distances                    |
| **ArrayList**      | Eligible Suppliers         | Temporary list of suppliers                             |
| **ArrayList**      | Distance Results           | Used in view connections sorting                        |
| **LinkedList**     | Replenishment Queue        | FIFO request queue                                      |
| **Queue**          | Replenishment Requests     | First generated request handled first                   |
| **PriorityQueue**  | FEFO Inventory Batches     | Earliest expiry stock first                             |
| **PriorityQueue**  | Dijkstra Algorithm         | Minimum distance node first                             |
| **LinkedList**     | Path Tracking              | Stores shortest route path                              |
| **List Interface** | Delivery Path              | Stores supplier to receiver path                        |
| **StringBuilder**  | Encryption/Decryption      | Efficient string creation                               |
| **Arrays**         | Fixed item lists           | Predefined inventory items                              |
| **Scanner**        | Input handling             | User inputs from keyboard                               |

---

## Folder Structure

```
MilitaryWarehouse/
├── passwords.txt                  ← Default login credentials
├── compile_and_run.sh             ← One-shot build & run script
└── src/
    └── military/
        ├── main/
        │   └── Main.java          ← Entry point + menu loop
        ├── data/
        │   └── DataStore.java     ← All global state (HashMaps, counters, session)
        ├── models/
        │   ├── Batch.java         ← Single FEFO stock batch
        │   ├── ItemStock.java     ← Item with FEFO PriorityQueue of batches
        │   ├── Warehouse.java     ← Warehouse node with inventory map
        │   ├── SupplyRequest.java ← Manual or auto-replenishment request
        │   ├── DeliveryOrder.java ← Active shipment between warehouses
        │   ├── ReplenishmentRequest.java ← Queued replenishment record
        │   ├── Edge.java          ← Graph edge (road connection)
        │   ├── NodeDistance.java  ← Dijkstra priority-queue entry
        │   ├── DijkstraResult.java← Shortest path result
        │   └── SupplierResult.java← Best supplier + route
        └── services/
            ├── AuthService.java       ← Login, lockout, switchLevel
            ├── InventoryService.java  ← Warehouse setup, view, threshold check, append
            ├── RequestService.java    ← Request creation, viewing, supplier search
            ├── DeliveryService.java   ← Approve, complete deliveries, consume, replenishment
            ├── GraphService.java      ← Build graph, Dijkstra, viewConnections
            └── UtilityService.java    ← Encrypt/decrypt, category, RFID, expiry date
```

---

## How to Compile and Run

### Option A — Shell Script (Linux / Mac / Git Bash)
```bash
cd MilitaryWarehouse
bash compile_and_run.sh
```

### Option B — Manual Terminal Commands
```bash
cd MilitaryWarehouse
mkdir -p out
find src -name "*.java" > sources.txt
javac -d out -sourcepath src @sources.txt
java -cp out military.main.Main
```

### Option C — VS Code
1. Install the **Extension Pack for Java** from the VS Code marketplace.
2. Open the `MilitaryWarehouse/` folder.
3. VS Code auto-detects source roots. Press **Run** on `Main.java`.

### Option D — IntelliJ IDEA
1. File → Open → select `MilitaryWarehouse/`
2. Right-click `src/` → Mark Directory as → Sources Root
3. Right-click `Main.java` → Run

---

## Default Login Credentials

| Step | Value |
|------|-------|
| System password | `start123` |
| Battalion password | `bat123` |
| Brigade password | `brig123` |
| Corps password | `corp123` |
| Command password | `cmd123` |
| AOC password | `aoc123` |

**Manager IDs** follow the pattern set in `InventoryService.loadWarehouses()`:
- AOC: `AOCMGR500`
- COMMAND_1: `CMGR401`, COMMAND_2: `CMGR402`, etc.
- CORPS_1: `CRPMGR301`, CORPS_2: `CRPMGR302`, etc.
- BRIGADE_1: `BMGR201`, BRIGADE_2: `BMGR202`, etc.
- BATTALION_1: `BATMGR101`, BATTALION_2: `BATMGR102`, etc.

---

## Key System Features

| Feature | Where Implemented |
|---------|------------------|
| XOR + mirror encryption | `UtilityService.encrypt/decrypt` |
| Account lockout (3 attempts → 1 hr) | `AuthService.verifyEncryptedCredential` |
| FEFO batch inventory | `ItemStock` (PriorityQueue by expiry) |
| Dijkstra shortest path | `GraphService.dijkstra` |
| Hierarchy-aware supplier selection | `RequestService.findBestSupplier` |
| FEFO stock deduction at approval | `DeliveryService.approveRequest` |
| FEFO batch credit at delivery | `DeliveryService.completeDelivery` |
| Auto-replenishment after consume | `DeliveryService.consumeItem` |
| AOC-only stock append | `InventoryService.appendStockAOCOnly` |
| Auto-delete fulfilled requests | `RequestService.autoDeleteFulfilledRequest` |

---

## Method Migration Reference (from original buffer3.java)

| Original Method | Moved To |
|----------------|----------|
| `encrypt / decrypt / mirror` | `UtilityService` |
| `getCategory` | `UtilityService` |
| `generateRFID` | `UtilityService` |
| `getExpiryDateByItem` | `UtilityService` |
| `getLevelPriority` | `UtilityService` |
| `loadPasswordsFromFile` | `AuthService` |
| `isLocked / verifyEncryptedCredential` | `AuthService` |
| `authenticate` | `AuthService` |
| `switchLevel` | `AuthService` |
| `loadWarehouses` | `InventoryService` |
| `initializeInventory` | `InventoryService` |
| `viewInventory` | `InventoryService` |
| `checkThresholdAndWarn` | `InventoryService` |
| `appendStockAOCOnly` | `InventoryService` |
| `addEdge / buildFullGraph` | `GraphService` |
| `dijkstra` | `GraphService` |
| `viewConnections` | `GraphService` |
| `requestItem` | `RequestService` |
| `viewPendingRequests / viewMyRequestsOnly` | `RequestService` |
| `viewMyReplenishmentQueue / viewReplenishmentQueue` | `RequestService` |
| `findBestSupplier / getEligibleSuppliers` | `RequestService` |
| `autoDeleteFulfilledRequest` | `RequestService` |
| `approveRequest` | `DeliveryService` |
| `viewPendingDeliveries` | `DeliveryService` |
| `completeDelivery` | `DeliveryService` |
| `consumeItem` | `DeliveryService` |
| `checkAndQueueReplenishment` | `DeliveryService` |
| All static HashMaps / ArrayLists | `DataStore` |
| `inventoryMenu` | `Main.runMenuLoop` |
| `main` | `Main.main` |
