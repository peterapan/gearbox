from typing import List

from fastapi import FastAPI, HTTPException

from app.database import initialize_database
from app.models import Equipment, EquipmentInput, RentalContract, RentalInput

# runs once when the server starts, so the tables always exist
initialize_database()

app = FastAPI(
    title="GearBox: Commercial Tools & Construction Equipment Rental Registry",
    description="Tracks rental equipment and records lease contracts issued to contractors.",
)


@app.get("/")
def home():
    return {"message": "GearBox API is running."}


@app.get("/equipment", response_model=List[Equipment], tags=["Equipment Inventory Management"])
def list_equipment():
    return Equipment.get_all()


@app.post("/equipment", response_model=Equipment, status_code=201, tags=["Equipment Inventory Management"])
def create_equipment(item: EquipmentInput):
    same_serial = Equipment.get_by_serial(item.serial_number)

    if same_serial is not None:
        raise HTTPException(status_code=409, detail="Serial number already exists.")
    else:
        return Equipment.create(item)


@app.put("/equipment/{equipment_id}", response_model=Equipment, tags=["Equipment Inventory Management"])
def update_equipment(equipment_id: int, item: EquipmentInput):
    existing_item = Equipment.get_by_id(equipment_id)
    if existing_item is None:
        raise HTTPException(status_code=404, detail="Equipment not found.")

    same_serial = Equipment.get_by_serial(item.serial_number)
    if same_serial is not None and same_serial.id != equipment_id:
        raise HTTPException(status_code=409, detail="Serial number already belongs to another item.")

    return Equipment.update(equipment_id, item)


@app.delete("/equipment/{equipment_id}", tags=["Equipment Inventory Management"])
def delete_equipment(equipment_id: int):
    result = Equipment.delete(equipment_id)

    if result == "not_found":
        raise HTTPException(status_code=404, detail="Equipment not found.")
    elif result == "has_contracts":
        raise HTTPException(status_code=409, detail="Cannot delete equipment that has lease contracts.")
    else:
        return {"success": True, "message": "Equipment " + str(equipment_id) + " deleted."}


@app.post("/lease", status_code=201, tags=["Lease Operations"])
def issue_lease(payload: RentalInput):
    result = RentalContract.issue_lease(payload)

    if result["status"] == "not_found":
        raise HTTPException(status_code=404, detail="Equipment not found.")
    elif result["status"] == "not_available":
        raise HTTPException(status_code=409, detail="This equipment is already leased out.")
    else:
        contract = result["contract"]
        return {
            "success": True,
            "message": "Equipment " + str(contract.equipment_id) + " leased to "
            + contract.contractor_name + " for site " + contract.site_location + ".",
            "contract": contract,
        }
