from datetime import datetime
from typing import List

from pydantic import BaseModel, Field

from app.database import database_connection


# ---------- Equipment ----------

class EquipmentInput(BaseModel):
    item_name: str = Field(..., min_length=1, max_length=100)
    brand: str = Field(..., min_length=1, max_length=50)
    serial_number: str = Field(..., min_length=3, max_length=50)
    category: str = Field(..., min_length=1, max_length=50)


class Equipment(EquipmentInput):
    id: int
    is_available: int = 1  # 1 = available, 0 = leased out

    @classmethod
    def get_all(cls) -> List["Equipment"]:
        with database_connection() as connection:
            rows = connection.execute("SELECT * FROM equipment").fetchall()

        all_equipment = []
        for row in rows:
            item = cls(
                id=row["id"],
                item_name=row["item_name"],
                brand=row["brand"],
                serial_number=row["serial_number"],
                category=row["category"],
                is_available=row["is_available"],
            )
            all_equipment.append(item)
        return all_equipment

    @classmethod
    def get_by_id(cls, equipment_id: int):
        with database_connection() as connection:
            row = connection.execute(
                "SELECT * FROM equipment WHERE id = ?", (equipment_id,)
            ).fetchone()

        if row is None:
            return None
        else:
            return cls(
                id=row["id"],
                item_name=row["item_name"],
                brand=row["brand"],
                serial_number=row["serial_number"],
                category=row["category"],
                is_available=row["is_available"],
            )

    @classmethod
    def get_by_serial(cls, serial_number: str):
        with database_connection() as connection:
            row = connection.execute(
                "SELECT * FROM equipment WHERE serial_number = ?", (serial_number,)
            ).fetchone()

        if row is None:
            return None
        else:
            return cls(
                id=row["id"],
                item_name=row["item_name"],
                brand=row["brand"],
                serial_number=row["serial_number"],
                category=row["category"],
                is_available=row["is_available"],
            )

    @classmethod
    def create(cls, equipment_input: EquipmentInput):
        with database_connection() as connection:
            cursor = connection.execute(
                "INSERT INTO equipment (item_name, brand, serial_number, category) VALUES (?, ?, ?, ?)",
                (
                    equipment_input.item_name,
                    equipment_input.brand,
                    equipment_input.serial_number,
                    equipment_input.category,
                ),
            )
            new_id = cursor.lastrowid
        return cls.get_by_id(new_id)

    @classmethod
    def update(cls, equipment_id: int, equipment_input: EquipmentInput):
        existing_item = cls.get_by_id(equipment_id)
        if existing_item is None:
            return None

        with database_connection() as connection:
            connection.execute(
                "UPDATE equipment SET item_name = ?, brand = ?, serial_number = ?, category = ? WHERE id = ?",
                (
                    equipment_input.item_name,
                    equipment_input.brand,
                    equipment_input.serial_number,
                    equipment_input.category,
                    equipment_id,
                ),
            )
        return cls.get_by_id(equipment_id)

    @classmethod
    def delete(cls, equipment_id: int) -> str:
        existing_item = cls.get_by_id(equipment_id)
        if existing_item is None:
            return "not_found"

        with database_connection() as connection:
            row = connection.execute(
                "SELECT COUNT(*) AS total FROM rental_contracts WHERE equipment_id = ?",
                (equipment_id,),
            ).fetchone()

            if row["total"] > 0:
                return "has_contracts"

            connection.execute("DELETE FROM equipment WHERE id = ?", (equipment_id,))
            return "deleted"


# ---------- Rental contracts ----------

class RentalInput(BaseModel):
    equipment_id: int = Field(..., gt=0)
    contractor_name: str = Field(..., min_length=1, max_length=100)
    site_location: str = Field(..., min_length=1, max_length=150)
    project_manager: str = Field(..., min_length=1, max_length=100)


class RentalContract(RentalInput):
    id: int
    lease_date: str

    @classmethod
    def issue_lease(cls, payload: RentalInput) -> dict:
        with database_connection() as connection:
            # start the transaction and lock the database for writing
            connection.execute("BEGIN IMMEDIATE")

            item = connection.execute(
                "SELECT id, is_available FROM equipment WHERE id = ?",
                (payload.equipment_id,),
            ).fetchone()

            if item is None:
                return {"status": "not_found"}

            if item["is_available"] != 1:
                return {"status": "not_available"}

            connection.execute(
                "UPDATE equipment SET is_available = 0 WHERE id = ?",
                (payload.equipment_id,),
            )

            lease_date = datetime.now().isoformat(timespec="seconds")

            cursor = connection.execute(
                "INSERT INTO rental_contracts "
                "(equipment_id, contractor_name, site_location, project_manager, lease_date) "
                "VALUES (?, ?, ?, ?, ?)",
                (
                    payload.equipment_id,
                    payload.contractor_name,
                    payload.site_location,
                    payload.project_manager,
                    lease_date,
                ),
            )
            contract_id = cursor.lastrowid

        contract = RentalContract(
            id=contract_id,
            equipment_id=payload.equipment_id,
            contractor_name=payload.contractor_name,
            site_location=payload.site_location,
            project_manager=payload.project_manager,
            lease_date=lease_date,
        )
        return {"status": "success", "contract": contract}
