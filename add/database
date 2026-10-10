import sqlite3
from contextlib import contextmanager

DATABASE_NAME = "gearbox.db"


@contextmanager
def database_connection():
    connection = sqlite3.connect(DATABASE_NAME)
    connection.row_factory = sqlite3.Row
    connection.execute("PRAGMA foreign_keys = ON")
    try:
        yield connection
        connection.commit()
    except Exception:
        connection.rollback()
        raise
    finally:
        connection.close()


def initialize_database():
    with database_connection() as connection:
        connection.execute(
            """
            CREATE TABLE IF NOT EXISTS equipment (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                item_name TEXT NOT NULL,
                brand TEXT NOT NULL,
                serial_number TEXT NOT NULL,
                category TEXT NOT NULL,
                is_available INTEGER DEFAULT 1
            )
            """
        )
        connection.execute(
            """
            CREATE TABLE IF NOT EXISTS rental_contracts (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                equipment_id INTEGER,
                contractor_name TEXT NOT NULL,
                site_location TEXT NOT NULL,
                project_manager TEXT NOT NULL,
                lease_date TEXT NOT NULL,
                FOREIGN KEY (equipment_id) REFERENCES equipment(id)
            )
            """
        )
