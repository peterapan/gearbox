# ⚙️ GearBox — Enterprise Asset & Equipment Rental Registry

This repository is a finalized Information System project for managing high-value industrial assets and tools. It replaces the basic starter template to implement a complete database architecture and transaction workflow for commercial equipment logistics.

---

## Run it on Windows

From this folder, ensure your virtual environment is active, install the direct dependencies, and boot up the development server:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\.venv\Scripts\Activate.ps1
python -m pip install fastapi uvicorn
python -m uvicorn main:app --reload
```

Then open **http://127.0.0** in your web browser. The built-in Swagger UI page will let you interactively test the complete system transaction workflow without writing a separate client.

---

## Project Structure

Upholding the strict repository architecture constraints, the project files are organized as follows:
* `main.py` - Creates the FastAPI web application instance and maps all API route endpoints.
* `app/database.py` - Configures the SQLite database context manager and runs `initialize_database()`.
* `app/models.py` - Defines Object-Oriented Pydantic validation schemas embedded with raw SQL query commands.
* `gearbox.db` - The local SQLite database storage file generated automatically the first time the app starts.

---

## How the Object Models Function

The system utilizes structural Pydantic data classes inside `app/models.py` to enforce automated type validation and leverage clean Object-Oriented inheritance patterns:

```python
from pydantic import BaseModel, Field

# Defines the baseline validation fields required to register any asset
class EquipmentInput(BaseModel):
    item_name: str = Field(min_length=1)
    brand: str = Field(min_length=1)
    serial_number: str = Field(min_length=1)
    category: str = Field(min_length=1)

# Inherits input fields and extends them with active database parameters
class Equipment(EquipmentInput):
    id: int
    is_available: int = 1
```

---

## How the SQL Queries Function

The database connections are managed cleanly through a synchronous connection context manager in `app/database.py`. Raw SQL commands are executed safely using `?` variable binding placeholders to maintain complete database integrity:

```python
with database_connection() as connection:
    # Example table schema tracking structural assets
    connection.execute("""
        CREATE TABLE IF NOT EXISTS equipment (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            item_name TEXT NOT NULL,
            brand TEXT NOT NULL,
            serial_number TEXT NOT NULL,
            category TEXT NOT NULL,
            is_available INTEGER DEFAULT 1
        )
    """)
```

---

## Current Active System Endpoints

The information system exposes four core inventory management routes and a dedicated business transaction pipeline:

| Endpoint | Parameters / Payload | Purpose | Core CRUD / Process |
| :--- | :--- | :--- | :--- |
| **GET /** | None | Displays a system status welcome message confirming the API is active. | General Status |
| **GET /equipment** | None | Retrieves a comprehensive list of all machinery and power tools registered in the warehouse. | Read (Inventory CRUD) |
| **POST /equipment** | `EquipmentInput` | Validates structural fields and registers a new high-value industrial asset into the database. | Create (Inventory CRUD) |
| **PUT /equipment/{equipment_id}** | `equipment_id: int`, `EquipmentInput` | Targets a specific asset row by its primary key ID and updates its tracked attributes. | Update (Inventory CRUD) |
| **DELETE /equipment/{equipment_id}** | `equipment_id: int` | Removes an asset from the database. Blocks the deletion if the item has active contracts. | Delete (Inventory CRUD) |
| **POST /lease** | `RentalInput` | **Core Transaction Workflow:** Validates item availability, shifts availability state, and appends a contract log. | Transactional Workflow |

---

## Reuse This Pattern For System Testing

To verify the integrated data flow across your system, follow this optimal testing sequence inside the `/docs` UI dashboard:
1. **POST /equipment**: Submit a high-value tool payload (e.g., Concrete Mixer) to register your first warehouse record (`id: 1`).
2. **POST /lease**: Submit a rental contract for `equipment_id: 1` to deploy it to a project job site. Your code updates the item's availability and creates a log record.
3. **GET /equipment**: Verify that the tool's availability state has dynamically changed to `0` (Leased Out).
4. **POST /lease (Conflict Test)**: Attempt to deploy `equipment_id: 1` a second time. The backend logic safely intercepts the request and blocks double-booking tracking errors with a `409 Conflict` message.
