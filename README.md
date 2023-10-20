import pyautogui
import time
import threading
import tkinter as tk
from tkinter import ttk

class AutoF5App:
    def __init__(self, master):
        self.master = master
        master.title("AutoF5 App")

        self.running = False
        self.auto_smooth = tk.BooleanVar()
        self.auto_smooth.set(False)  # По умолчанию авто-плавность отключена

        # Заголовок
        title_label = ttk.Label(master, text="AutoF5 Application", font=("Helvetica", 16, "bold"))
        title_label.grid(row=0, column=0, columnspan=2, pady=(10, 20))

        # Секция ввода параметров
        settings_frame = ttk.LabelFrame(master, text="Settings", padding=(10, 5))
        settings_frame.grid(row=1, column=0, padx=10, pady=10, sticky="ew")

        # Время выполнения
        ttk.Label(settings_frame, text="Total Time (seconds):").grid(row=0, column=0, sticky="e", pady=5)
        self.total_time_entry = ttk.Entry(settings_frame)
        self.total_time_entry.grid(row=0, column=1, pady=5)

        # Интервал
        ttk.Label(settings_frame, text="Interval (seconds):").grid(row=1, column=0, sticky="e", pady=5)
        self.interval_entry = ttk.Entry(settings_frame)
        self.interval_entry.grid(row=1, column=1, pady=5)

        # Клавиша
        ttk.Label(settings_frame, text="Key:").grid(row=2, column=0, sticky="e", pady=5)
        self.key_entry = ttk.Entry(settings_frame)
        self.key_entry.grid(row=2, column=1, pady=5)

        # Авто-плавность
        self.auto_smooth_checkbox = ttk.Checkbutton(settings_frame, text="Auto Smooth", variable=self.auto_smooth)
        self.auto_smooth_checkbox.grid(row=3, column=1, pady=5)

        # Секция управления
        control_frame = ttk.Frame(master)
        control_frame.grid(row=1, column=1, padx=10, pady=10)

        # Кнопка "Start"
        self.start_button = ttk.Button(control_frame, text="Start", command=self.start)
        self.start_button.grid(row=0, column=0, pady=5)

        # Кнопка "Stop"
        self.stop_button = ttk.Button(control_frame, text="Stop", command=self.stop)
        self.stop_button.grid(row=0, column=1, pady=5)

        # Секция статуса
        status_frame = ttk.LabelFrame(master, text="Status", padding=(10, 5))
        status_frame.grid(row=2, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")

        # Текстовое поле статуса
        self.status_text = tk.Text(status_frame, height=10, width=40, font=("Courier New", 12))
        self.status_text.pack(fill=tk.BOTH, expand=True)

        # Разделитель для статуса
        ttk.Separator(master, orient=tk.HORIZONTAL).grid(row=3, column=0, columnspan=2, sticky="ew", pady=10)

    def start(self):
        if not self.running:
            self.running = True
            self.start_button["state"] = "disabled"
            self.stop_button["state"] = "enabled"
            total_time = float(self.total_time_entry.get())
            interval = float(self.interval_entry.get())
            key = self.key_entry.get() or 'F5'  # Если поле не заполнено, используем 'F5'
            thread = threading.Thread(target=self.run_auto_f5, args=(total_time, interval, key))
            thread.start()

    def stop(self):
        self.running = False

    def run_auto_f5(self, total_time, interval, key):
        try:
            iterations = int(total_time / interval)
            for i in range(1, iterations + 1):
                if not self.running:
                    break
                pyautogui.press(key)
                if self.auto_smooth.get():
                    interval *= 1.001
                time.sleep(interval)
                self.update_status(f"Key {key} pressed, {i} times - completed successfully")
                if i >= 9:
                    self.delete_first_line()
            self.running = False
        except Exception as e:
            self.update_status(f"Error: {str(e)}")
        finally:
            self.start_button["state"] = "enabled"
            self.stop_button["state"] = "disabled"

    def update_status(self, message):
        self.status_text.insert(tk.END, message + "\n")

    def delete_first_line(self):
        self.status_text.delete("1.0", "2.0")

if __name__ == "__main__":
    root = tk.Tk()
    app = AutoF5App(root)
    root.mainloop()

