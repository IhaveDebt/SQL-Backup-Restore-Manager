# backup_manager.py
import psycopg2
import json

DB_PARAMS = {
    "dbname": "backup_db",
    "user": "postgres",
    "password": "password",
    "host": "localhost",
    "port": 5432
}

def backup_data(filename="backup.json"):
    conn = psycopg2.connect(**DB_PARAMS)
    cur = conn.cursor()
    cur.execute("SELECT * FROM users;")
    rows = cur.fetchall()
    with open(filename, "w") as f:
        json.dump(rows, f)
    cur.close()
    conn.close()
    print(f"Backup saved to {filename}")

def restore_data(filename="backup.json"):
    conn = psycopg2.connect(**DB_PARAMS)
    cur = conn.cursor()
    with open(filename) as f:
        rows = json.load(f)
        for r in rows:
            cur.execute("INSERT INTO users(id, name, email) VALUES (%s, %s, %s) ON CONFLICT DO NOTHING;", r)
    conn.commit()
    cur.close()
    conn.close()
    print(f"Data restored from {filename}")

if __name__ == "__main__":
    backup_data()
    restore_data()
