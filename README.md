SOURCE CODE:
import java.util.*;

// Item class
class Item {
    private String itemId;
    private String name;

    public Item(String itemId, String name) {
        this.itemId = itemId;
        this.name = name;
    }

    public String getItemId() { return itemId; }
    public String getName() { return name; }
}

// Location class
class Location {
    private String locationId;
    private String description;

    public Location(String locationId, String description) {
        this.locationId = locationId;
        this.description = description;
    }

    public String getLocationId() { return locationId; }
    public String getDescription() { return description; }
}

// InventoryRecord class
class InventoryRecord {
    private Item item;
    private Location location;
    private int quantity;

    public InventoryRecord(Item item, Location location, int quantity) {
        this.item = item;
        this.location = location;
        this.quantity = quantity;
    }

    public Item getItem() { return item; }
    public Location getLocation() { return location; }
    public int getQuantity() { return quantity; }
    public void adjustQuantity(int delta) { this.quantity += delta; }
}

// PickTask class
class PickTask {
    private Item item;
    private Location location;
    private int quantity;
    private boolean picked;

    public PickTask(Item item, Location location, int quantity) {
        this.item = item;
        this.location = location;
        this.quantity = quantity;
        this.picked = false;
    }

    public Item getItem() { return item; }
    public Location getLocation() { return location; }
    public int getQuantity() { return quantity; }
    public boolean isPicked() { return picked; }
    public void setPicked(boolean picked) { this.picked = picked; }
}

// PickList class
class PickList {
    private List<PickTask> tasks = new ArrayList<>();

    public void addTask(PickTask task) { tasks.add(task); }
    public List<PickTask> getTasks() { return tasks; }
}

// Pack class
class Pack {
    private PickList pickList;
    private boolean packed;

    public Pack(PickList pickList) {
        this.pickList = pickList;
        this.packed = false;
    }

    public void confirmPacking() { this.packed = true; }
    public boolean isPacked() { return packed; }
    public PickList getPickList() { return pickList; }
}

// Shipment class
class Shipment {
    private Pack pack;
    private String carrier;
    private String trackingNo;
    private boolean shipped;

    public Shipment(Pack pack) {
        this.pack = pack;
        this.shipped = false;
    }

    public void closeShipment(String carrier, String trackingNo) {
        if (pack.isPacked()) {
            this.carrier = carrier;
            this.trackingNo = trackingNo;
            this.shipped = true;
        } else {
            System.out.println("Shipment cannot be closed. Pack not completed.");
        }
    }

    public boolean isShipped() { return shipped; }
    public String getManifest() {
        StringBuilder sb = new StringBuilder();
        sb.append("Carrier: ").append(carrier)
          .append(", Tracking No: ").append(trackingNo).append("\n");
        for (PickTask task : pack.getPickList().getTasks()) {
            sb.append(task.getItem().getName())
              .append(" - ").append(task.getQuantity()).append("\n");
        }
        return sb.toString();
    }
}

// Main Application
public class WarehouseOperations_WarehouseApp {
    private static Map<String, Item> items = new HashMap<>();
    private static Map<String, Location> locations = new HashMap<>();
    private static List<InventoryRecord> inventory = new ArrayList<>();
    private static PickList currentPickList = null;
    private static Pack currentPack = null;
    private static Shipment currentShipment = null;

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.println("\nWarehouse Operations Menu");
            System.out.println("1. Add Item");
            System.out.println("2. Add Location");
            System.out.println("3. Adjust Inventory");
            System.out.println("4. Create Pick List");
            System.out.println("5. Record Pick");
            System.out.println("6. Create Pack");
            System.out.println("7. Ship Order");
            System.out.println("8. Inventory Summary");
            System.out.println("9. Exit");
            System.out.print("Choose: ");
            int choice = sc.nextInt(); sc.nextLine();

            switch (choice) {
                case 1: addItem(sc); break;
                case 2: addLocation(sc); break;
                case 3: adjustInventory(sc); break;
                case 4: createPickList(sc); break;
                case 5: recordPick(sc); break;
                case 6: createPack(); break;
                case 7: shipOrder(sc); break;
                case 8: inventorySummary(); break;
                case 9: System.exit(0);
                default: System.out.println("Invalid choice.");
            }
        }
    }

    private static void addItem(Scanner sc) {
        System.out.print("Enter item id: ");
        String id = sc.nextLine();
        if (items.containsKey(id)) {
            System.out.println("Item already exists.");
            return;
        }
        System.out.print("Enter item name: ");
        String name = sc.nextLine();
        items.put(id, new Item(id, name));
        System.out.println("Item added.");
    }

    private static void addLocation(Scanner sc) {
        System.out.print("Enter location id: ");
        String id = sc.nextLine();
        if (locations.containsKey(id)) {
            System.out.println("Location already exists.");
            return;
        }
        System.out.print("Enter description: ");
        String desc = sc.nextLine();
        locations.put(id, new Location(id, desc));
        System.out.println("Location added.");
    }

    private static void adjustInventory(Scanner sc) {
        System.out.print("Enter item id: ");
        String itemId = sc.nextLine();
        Item item = items.get(itemId);
        if (item == null) { System.out.println("Invalid item."); return; }

        System.out.print("Enter location id: ");
        String locId = sc.nextLine();
        Location loc = locations.get(locId);
        if (loc == null) { System.out.println("Invalid location."); return; }

        System.out.print("Enter quantity to adjust: ");
        int qty = sc.nextInt(); sc.nextLine();

        InventoryRecord record = findRecord(item, loc);
        if (record == null) {
            record = new InventoryRecord(item, loc, 0);
            inventory.add(record);
        }
        record.adjustQuantity(qty);
        System.out.println("Inventory adjusted.");
    }

    private static void createPickList(Scanner sc) {
        currentPickList = new PickList();
        while (true) {
            System.out.print("Enter item id (or 'done'): ");
            String itemId = sc.nextLine();
            if (itemId.equalsIgnoreCase("done")) break;

            Item item = items.get(itemId);
            if (item == null) { System.out.println("Invalid item."); continue; }

            System.out.print("Enter location id: ");
            String locId = sc.nextLine();
            Location loc = locations.get(locId);
            if (loc == null) { System.out.println("Invalid location."); continue; }

            System.out.print("Enter quantity: ");
            int qty = sc.nextInt(); sc.nextLine();

            InventoryRecord record = findRecord(item, loc);
            if (record == null || record.getQuantity() < qty) {
                System.out.println("Not enough stock.");
                continue;
            }

            record.adjustQuantity(-qty);
            currentPickList.addTask(new PickTask(item, loc, qty));
            System.out.println("Task added to pick list.");
        }

        System.out.println("Pick List created:");
        for (PickTask task : currentPickList.getTasks()) {
            System.out.println(task.getItem().getName() +
                               " from " + task.getLocation().getDescription() +
                               " - Qty: " + task.getQuantity());
        }
    }

    private static void recordPick(Scanner sc) {
        if (currentPickList == null) {
            System.out.println("No active pick list.");
            return;
        }
        for (PickTask task : currentPickList.getTasks()) {
            if (!task.isPicked()) {
                System.out.println("Picking: " + task.getItem().getName() +
                                   " - Qty: " + task.getQuantity());
                task.setPicked(true);
            }
        }
        System.out.println("All tasks marked as picked.");
    }

    private static void createPack() {
        if (currentPickList == null) {
            System.out.println("No pick list available.");
            return;
        }
        currentPack = new Pack(currentPickList);
        currentPack.confirmPacking();
        System.out.println("Pack created and confirmed.");
    }

    private static void shipOrder(Scanner sc) {
        if (currentPack == null || !currentPack.isPacked()) {
            System.out.println("No packed order to ship.");
            return;
        }
        currentShipment = new Shipment(currentPack);
        System.out.print("Enter carrier: ");
        String carrier = sc.nextLine();
        System.out.print("Enter tracking number: ");
        String trackingNo = sc.nextLine();
        currentShipment.closeShipment(carrier, trackingNo);
        if (currentShipment.isShipped()) {
            System.out.println("Shipment closed. Manifest:\n" + currentShipment.getManifest());
        }
    }

    private static void inventorySummary() {
        System.out.println("\nInventory Snapshot:");
        for (InventoryRecord rec : inventory) {
            System.out.println(rec.getItem().getName() +
                               " @ " + rec.getLocation().getDescription() +
                               " = " + rec.getQuantity());
        }
    }

    private static InventoryRecord findRecord(Item item, Location loc) {
        for (InventoryRecord rec : inventory) {
            if (rec.getItem().equals(item) && rec.getLocation().equals(loc)) {
                return rec;
            }
        }
        return null;
    }
}

