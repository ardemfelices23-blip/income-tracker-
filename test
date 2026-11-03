import flet as ft
import sqlite3
import datetime

DB_FILE = "income_tracker.db"

def init_db():
    conn = sqlite3.connect(DB_FILE)
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS records (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            date TEXT NOT NULL,
            job TEXT NOT NULL,
            location TEXT NOT NULL,
            price REAL NOT NULL
        )
    ''')
    conn.commit()
    conn.close()

def main(page: ft.Page):
    page.title = "Code on the Go - Tracker"
    page.vertical_alignment = ft.MainAxisAlignment.START
    page.padding = 20
    page.scroll = ft.ScrollMode.AUTO

    init_db()

    job_input = ft.TextField(label="Pangalan ng Job / Task", width=300)
    location_input = ft.TextField(label="Location / Lokasyon", width=300)
    price_input = ft.TextField(label="Price / Income (PHP)", keyboard_type=ft.KeyboardType.NUMBER, width=300)
    
    status_text = ft.Text("", color=ft.colors.GREEN)
    output_list = ft.ListView(expand=1, spacing=10, padding=10, auto_scroll=True)
    total_display = ft.Text("Total: ₱0.00", weight=ft.FontWeight.BOLD, size=16)

    def save_record(e):
        if not job_input.value or not location_input.value or not price_input.value:
            status_text.value = "Pakipunan ang lahat ng fields!"
            status_text.color = ft.colors.RED
            page.update()
            return

        try:
            price = float(price_input.value)
        except ValueError:
            status_text.value = "Mali ang presyo. Maglagay ng numero."
            status_text.color = ft.colors.RED
            page.update()
            return

        date_today = datetime.date.today().isoformat()

        conn = sqlite3.connect(DB_FILE)
        cursor = conn.cursor()
        cursor.execute("INSERT INTO records (date, job, location, price) VALUES (?, ?, ?, ?)",
                       (date_today, job_input.value, location_input.value, price))
        conn.commit()
        conn.close()

        status_text.value = "Tagumpay na nai-save ang record!"
        status_text.color = ft.colors.GREEN
        job_input.value = ""
        location_input.value = ""
        price_input.value = ""
        page.update()

    def load_daily(e):
        output_list.controls.clear()
        today = datetime.date.today().isoformat()
        conn = sqlite3.connect(DB_FILE)
        cursor = conn.cursor()
        cursor.execute("SELECT job, location, price FROM records WHERE date = ?", (today,))
        rows = cursor.fetchall()
        conn.close()

        total = 0
        if not rows:
            output_list.controls.append(ft.Text(f"Walang record ngayong araw ({today})."))
        else:
            for job, location, price in rows:
                output_list.controls.append(
                    ft.Card(content=ft.Container(content=ft.Column([
                        ft.Text(f"Job: {job}", weight=ft.FontWeight.BOLD),
                        ft.Text(f"Location: {location}"),
                        ft.Text(f"Price: ₱{price:,.2f}", color=ft.colors.GREEN_700),
                    ]), padding=10))
                )
                total += price
        total_display.value = f"TOTAL KITA NGAYONG ARAW: ₱{total:,.2f}"
        page.update()

    def load_weekly(e):
        output_list.controls.clear()
        today = datetime.date.today()
        start_of_week = today - datetime.timedelta(days=today.weekday())
        end_of_week = start_of_week + datetime.timedelta(days=6)

        conn = sqlite3.connect(DB_FILE)
        cursor = conn.cursor()
        cursor.execute("SELECT date, job, location, price FROM records WHERE date BETWEEN ? AND ? ORDER BY date DESC",
                       (start_of_week.isoformat(), end_of_week.isoformat()))
        rows = cursor.fetchall()
        conn.close()

        total = 0
        if not rows:
            output_list.controls.append(ft.Text("Walang record sa linggong ito."))
        else:
            for date, job, location, price in rows:
                output_list.controls.append(
                    ft.Card(content=ft.Container(content=ft.Column([
                        ft.Text(f"Petsa: {date}", size=12, color=ft.colors.GREY_700),
                        ft.Text(f"Job: {job}", weight=ft.FontWeight.BOLD),
                        ft.Text(f"Location: {location}"),
                        ft.Text(f"Price: ₱{price:,.2f}", color=ft.colors.GREEN_700),
                    ]), padding=10))
                )
                total += price
        total_display.value = f"TOTAL KITA NG LINGGONG ITO: ₱{total:,.2f}"
        page.update()

    def load_monthly(e):
        output_list.controls.clear()
        current_month = datetime.date.today().strftime("%Y-%m")

        conn = sqlite3.connect(DB_FILE)
        cursor = conn.cursor()
        cursor.execute("SELECT date, job, location, price FROM records WHERE date LIKE ? ORDER BY date DESC",
                       (f"{current_month}%",))
        rows = cursor.fetchall()
        conn.close()

        total = 0
        if not rows:
            output_list.controls.append(ft.Text(f"Walang record ngayong buwan ({current_month})."))
        else:
            for date, job, location, price in rows:
                output_list.controls.append(
                    ft.Card(content=ft.Container(content=ft.Column([
                        ft.Text(f"Petsa: {date}", size=12, color=ft.colors.GREY_700),
                        ft.Text(f"Job: {job}", weight=ft.FontWeight.BOLD),
                        ft.Text(f"Location: {location}"),
                        ft.Text(f"Price: ₱{price:,.2f}", color=ft.colors.GREEN_700),
                    ]), padding=10))
                )
                total += price
        total_display.value = f"TOTAL KITA NG BUWANG ITO: ₱{total:,.2f}"
        page.update()

    page.add(
        ft.Text("Code on the Go - Tracker", size=20, weight=ft.FontWeight.BOLD),
        job_input,
        location_input,
        price_input,
        ft.ElevatedButton("I-save ang Record", on_click=save_record, bgcolor=ft.colors.BLUE, color=ft.colors.WHITE),
        status_text,
        ft.Divider(),
        ft.Row([
            ft.ElevatedButton("Daily", on_click=load_daily),
            ft.ElevatedButton("Weekly", on_click=load_weekly),
            ft.ElevatedButton("Monthly", on_click=load_monthly),
        ], alignment=ft.MainAxisAlignment.SPACE_EVENWAY),
        total_display,
        output_list
    )

ft.app(target=main)
