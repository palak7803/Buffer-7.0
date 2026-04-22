
# Military Warehouse Management System

A multi-level military supply chain management system written in Java.  
Refactored from a single-file design into a clean, modular package structure.

## Data Structures Used

| Data Structure     | Usage Area                | Purpose                                                  |
|------------------|--------------------------|----------------------------------------------------------|
| **HashMap**       | Password Storage          | Stores username → encrypted password                     |
| **HashMap**       | Login Attempts            | Tracks number of failed login attempts                   |
| **HashMap**       | Account Lock System       | Stores account lock duration                             |
| **HashMap**       | Warehouse Levels          | Maps warehouse → hierarchy level                         |
| **HashMap**       | Manager IDs               | Maps warehouse → manager ID                              |
| **HashMap**       | Warehouse Database        | Stores warehouse → warehouse object                      |
| **HashMap**       | Graph Representation      | Stores node → connected edges                            |
| **HashMap**       | Inventory Management      | Stores item name → item stock details                    |
| **ArrayList**     | Pending Requests          | Stores all supply requests                               |
| **ArrayList**     | Pending Deliveries        | Stores all delivery orders                               |
| **ArrayList**     | Graph Edges               | Stores connected nodes with distances                    |
| **ArrayList**     | Eligible Suppliers        | Temporary list of available suppliers                    |
| **ArrayList**     | Distance Results          | Used for sorting distances in path view                  |
| **LinkedList**    | Replenishment Queue       | Maintains FIFO request handling                          |
| **Queue**         | Replenishment Requests    | Ensures first request is processed first                 |
| **PriorityQueue** | FEFO Inventory Batches    | Processes earliest expiry stock first                    |
| **PriorityQueue** | Dijkstra Algorithm        | Selects minimum distance node first                      |
| **LinkedList**    | Path Tracking             | Stores shortest path route                               |
| **List Interface**| Delivery Path             | Stores supplier → receiver route                         |
| **StringBuilder** | Encryption/Decryption     | Efficient string manipulation                            |
| **Arrays**        | Fixed Item List           | Stores predefined inventory items                        |
| **Scanner**       | Input Handling            | Handles user input                                       |
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
