# FastTagingSyaiful

Program ini digunakan untuk menambahkan penanda (placemark) di Google Earth Pro secara otomatis saat berada dalam mode Street View. Pengguna dapat menambahkan penanda dengan klik kanan atau menulis nama penanda secara manual dengan klik tengah. Program ini juga dapat dihentikan dengan menekan tombol [ESC](file:///c%3A/Users/SYAIFUL%20WACHID/Desktop/FastTagingSyaiful.py#148%2C280-148%2C280).

Berikut adalah penjelasan rinci dan urut dari kode Python di atas dalam bahasa Indonesia:

1. **Import Library**:
   ```python
   import pyautogui
   import time
   import tkinter as tk
   from tkinter import messagebox
   from pynput import mouse, keyboard
   import pygetwindow as gw
   ```
   Bagian ini mengimpor library yang diperlukan:
   - `pyautogui`: Untuk mengendalikan mouse dan keyboard secara otomatis.
   - `time`: Untuk memberikan jeda dalam proses.
   - `tkinter`: Untuk membuat dialog box GUI (Graphical User Interface).
   - `pynput`: Untuk mendeteksi dan mendengarkan input mouse dan keyboard.
   - `pygetwindow`: Untuk mengelola jendela aplikasi.

2. **Variabel Global**:
   ```python
   stop_script = False
   manual_naming = False
   mouse_position = (0, 0)  # Posisi mouse default
   ```
   Mendefinisikan variabel global yang akan digunakan untuk mengontrol alur program:
   - `stop_script`: Bendera untuk menghentikan skrip.
   - `manual_naming`: Bendera untuk menentukan apakah penamaan dilakukan secara manual.
   - `mouse_position`: Menyimpan posisi mouse.

3. **Fungsi `show_messagebox`**:
   ```python
   def show_messagebox(title, message):
       root = tk.Tk()
       root.withdraw()  # Menyembunyikan jendela utama
       root.attributes("-topmost", True)  # Memastikan messagebox berada di atas
       messagebox.showinfo(title, message)
       root.destroy()
   ```
   Fungsi ini menampilkan kotak pesan dengan judul dan pesan yang diberikan.

4. **Fungsi `show_input_dialog`**:
   ```python
   def show_input_dialog(title, message):
       def on_submit():
           nonlocal user_input
           user_input = entry.get()
           root.quit()
           root.destroy()
   
       user_input = None
       root = tk.Tk()
       root.title(title)
       root.geometry("380x180")
       root.attributes("-topmost", True)
   
       window_width = 380
       window_height = 180
       screen_width = root.winfo_screenwidth()
       screen_height = root.winfo_screenheight()
       position_right = int(screen_width/2 - window_width/2)
       position_down = int(screen_height/2 - window_height/2)
       root.geometry(f"{window_width}x{window_height}+{position_right}+{position_down}")
   
       label = tk.Label(root, text=message)
       label.pack(pady=10)
   
       entry = tk.Entry(root)
       entry.pack(pady=5)
       entry.focus_set()
   
       submit_button = tk.Button(root, text="Submit", command=on_submit)
       submit_button.pack(pady=10)
   
       root.update_idletasks()
       dialog_window = gw.getWindowsWithTitle(title)
       if dialog_window:
           dialog_window[0].activate()
           print(f"Dialog box '{title}' activated")
   
       root.mainloop()
       return user_input
   ```
   Fungsi ini menampilkan kotak dialog input di mana pengguna dapat memasukkan teks dan mengirimkannya.

5. **Fungsi `wait_for_mouse_click`**:
   ```python
   def wait_for_mouse_click():
       print("Waiting for right or middle mouse click...")
   
       def on_click(x, y, button, pressed):
           global manual_naming, mouse_position
           if pressed:
               if button == mouse.Button.right:
                   print(f"Right-click detected at position: ({x}, {y})")
                   mouse_position = (x, y)
                   manual_naming = False
                   return False  # Stop listener
               elif button == mouse.Button.middle:
                   print(f"Middle-click detected at position: ({x}, {y})")
                   mouse_position = (x, y)
                   manual_naming = True
                   return False  # Stop listener
   
       with mouse.Listener(on_click=on_click) as listener:
           listener.join()
   ```
   Fungsi ini menunggu klik kanan atau tengah pada mouse dan menyimpan posisi mouse yang diklik.

6. **Fungsi `add_placemark`**:
   ```python
   def add_placemark(counter):
       try:
           print("Sending Ctrl+Shift+P")
           pyautogui.hotkey('ctrl', 'shift', 'p')
   
           time.sleep(0.5)
   
           current_mouse_position = pyautogui.position()
           print(f"Current mouse position: {current_mouse_position}")
   
           target_position = (1178, 534)
           print(f"Moving mouse to position: {target_position}")
           pyautogui.moveTo(target_position)
   
           print(f"Dragging placemark to position: {current_mouse_position}")
           pyautogui.mouseDown(button='left')
           pyautogui.moveTo(current_mouse_position)
           pyautogui.mouseUp(button='left')
   
           print("Ensuring dialog box is active")
           dialog_title = "Google Earth - New Placemark"
           dialog_window = gw.getWindowsWithTitle(dialog_title)
           if dialog_window:
               dialog_window[0].activate()
               print(f"Dialog box '{dialog_title}' found and activated")
               time.sleep(0.5)
           else:
               print(f"Dialog box '{dialog_title}' not found")
   
           if manual_naming:
               print("Waiting for user to manually enter placemark name...")
               user_input = show_input_dialog("Info", "Please enter placemark name manually and press Submit to continue.")
               if user_input:
                   print(f"Entered placemark name: {user_input}")
                   pyautogui.typewrite(user_input)
           else:
               print(f"Naming placemark: {counter}")
               pyautogui.typewrite(str(counter))
   
           print("Pressing Enter to close the dialog")
           pyautogui.press('enter')
   
       except Exception as e:
           print(f"Error in add_placemark: {e}")
   ```
   Fungsi ini menambahkan penanda (placemark) di Google Earth:
   - Menekan tombol `Ctrl+Shift+P` untuk membuka dialog penambahan placemark.
   - Menunggu 0,5 detik untuk memastikan dialog terbuka.
   - Menggeser posisi mouse ke koordinat target dan mengklik untuk menetapkan placemark.
   - Jika `manual_naming` diaktifkan, meminta pengguna untuk memasukkan nama placemark secara manual.
   - Jika tidak, memberikan nama berdasarkan nomor counter.
   - Menekan tombol Enter untuk menutup dialog.

7. **Fungsi `on_press`**:
   ```python
   def on_press(key):
       global stop_script
       try:
           if key == keyboard.Key.esc:
               stop_script = True
               show_messagebox("Info", "The AUTO MARKING SCRIPT process has been stopped. Restart the program to start the MARKING process again.\n\nIf you need a specific program to make your work easier, contact me via WhatsApp or LinkedIn.")
               return False  # Stop listener
       except AttributeError:
           pass
   ```
   Fungsi ini menghentikan skrip jika tombol ESC ditekan dan menampilkan pesan bahwa skrip telah berhenti.

8. **Fungsi `main`**:
   ```python
   def main():
       global stop_script
       counter = 1
       show_messagebox("Info", "This program is used for <TAGGING> HOMPASS NUMBER while in STREETVIEW MODE directly in GOOGLE EARTH PRO.\n\nUse 'RIGHT CLICK' to start <MARKING> or 'MIDDLE CLICK' to manually enter placemark name.\nPress ESC to exit.\n\nThis program is created by:\nSyaiful Wachid > Senior Project Designer\n@PT.Fiberhome Indonesia\nAs an urgent ondesk survey solution\nLinkedIn Account: Syaiful Wachid\nCP:082230696953")
       while not stop_script:
           print("Waiting for right or middle click to add placemark...")
           wait_for_mouse_click()
           if stop_script:
               break
           print(f"Adding placemark with name: {counter}")
           add_placemark(counter)
           if not manual_naming:
               counter += 1
           print(f"Placemark {counter - 1} added, waiting for next click...")
   ```
   Fungsi utama yang menjalankan logika utama program:
   - Menampilkan kotak pesan dengan instruksi kepada pengguna.
   - Menunggu klik kanan atau tengah untuk menambahkan penanda.
   - Menambahkan penanda menggunakan `add_placemark` dan meningkatkan counter jika `manual_naming` tidak diaktifkan.

9. **Blok `if __name__ == "__main__"`**:
   ```python
   if __name__ == "__main__":
       time.sleep(5)
       print("Python script started")
       with keyboard.Listener(on_press=on_press) as listener:
           main()
           listener.join()
   ```
   Blok ini memastikan bahwa skrip hanya berjalan jika dijalankan sebagai program utama:
   - Memberi jeda 5 detik untuk memastikan Google Earth Pro aktif.
   - Menampilkan pesan debug.
   - Memulai listener keyboard untuk mendeteksi input ESC dan menjalankan fungsi `main`.

Itulah penjelasan rinci dan urut dari kode Python di atas.

Contoh Alur Kerja
1. Program dimulai dan menunggu 5 detik.
2. Menampilkan pesan informasi.
3. Menunggu klik kanan atau tengah dari mouse.
4. Jika klik kanan, menambahkan penanda dengan nama counter.
5. Jika klik tengah, menunggu input nama penanda dari pengguna.
6. Menekan Enter untuk menutup dialog.
7. Jika tombol ESC ditekan, program berhenti dan menampilkan pesan informasi.

Program ini sangat berguna untuk menandai lokasi secara otomatis di Google Earth Pro, terutama saat melakukan survei atau tagging dalam mode Street View.
Berikut Source Code Lengkapnya :

```python
import pyautogui
import time
import tkinter as tk
from tkinter import messagebox
from pynput import mouse, keyboard
import pygetwindow as gw

# Global flag to stop the script
stop_script = False
manual_naming = False
mouse_position = (0, 0)  # Default mouse position

def show_messagebox(title, message):
    root = tk.Tk()
    root.withdraw()  # Hide the main window
    root.attributes("-topmost", True)  # Ensure the messagebox is on top
    messagebox.showinfo(title, message)
    root.destroy()

def show_input_dialog(title, message):
    def on_submit():
        nonlocal user_input
        user_input = entry.get()
        root.quit()
        root.destroy()

    user_input = None
    root = tk.Tk()
    root.title(title)
    root.geometry("380x180")
    root.attributes("-topmost", True)

    # Calculate position to center the window
    window_width = 380
    window_height = 180
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
    position_right = int(screen_width/2 - window_width/2)
    position_down = int(screen_height/2 - window_height/2)
    root.geometry(f"{window_width}x{window_height}+{position_right}+{position_down}")

    label = tk.Label(root, text=message)
    label.pack(pady=10)

    entry = tk.Entry(root)
    entry.pack(pady=5)
    entry.focus_set()

    submit_button = tk.Button(root, text="Submit", command=on_submit)
    submit_button.pack(pady=10)

    # Pastikan jendela dialog aktif
    root.update_idletasks()
    dialog_window = gw.getWindowsWithTitle(title)
    if dialog_window:
        dialog_window[0].activate()
        print(f"Dialog box '{title}' diaktifkan")

    root.mainloop()
    return user_input

def wait_for_mouse_click():
    print("Menunggu klik kanan atau tengah mouse...")

    def on_click(x, y, button, pressed):
        global manual_naming, mouse_position
        if pressed:
            if button == mouse.Button.right:
                print(f"Klik kanan terdeteksi pada posisi: ({x}, {y})")
                mouse_position = (x, y)
                manual_naming = False
                return False  # Stop listener
            elif button == mouse.Button.middle:
                print(f"Klik tengah terdeteksi pada posisi: ({x}, {y})")
                mouse_position = (x, y)
                manual_naming = True
                return False  # Stop listener

    with mouse.Listener(on_click=on_click) as listener:
        listener.join()

def add_placemark(counter):
    # Tekan Ctrl+Shift+P untuk menambahkan penanda
    print("Mengirim Ctrl+Shift+P")
    pyautogui.hotkey('ctrl', 'shift', 'p')

    # Tunggu 1 detik agar dialog muncul
    time.sleep(0.5)

    # Ambil dan rekam posisi mouse saat ini
    current_mouse_position = pyautogui.position()
    print(f"Posisi mouse saat ini: {current_mouse_position}")

    # Pindahkan posisi mouse ke titik koordinat (1178, 534)
    target_position = (1178, 534) #x=1178, y=534
    print(f"Memindahkan mouse ke posisi: {target_position}")
    pyautogui.moveTo(target_position)

    # Klik kiri dan tahan, lalu geser ke posisi mouse yang terekam
    print(f"Menggeser placemark ke posisi: {current_mouse_position}")
    pyautogui.mouseDown(button='left')
    pyautogui.moveTo(current_mouse_position)
    pyautogui.mouseUp(button='left')

    # Pastikan dialog box aktif
    print("Memastikan dialog box aktif")
    dialog_title = "Google Earth - New Placemark"  # Sesuaikan dengan judul dialog box yang muncul
    dialog_window = gw.getWindowsWithTitle(dialog_title)
    if dialog_window:
        dialog_window[0].activate()
        print(f"Dialog box '{dialog_title}' ditemukan dan diaktifkan")

        # Tambahkan penundaan untuk memastikan dialog box aktif
        time.sleep(0.5)
    else:
        print(f"Dialog box '{dialog_title}' tidak ditemukan")

    if manual_naming:
        print("Menunggu pengguna untuk menulis nama placemark secara manual...")
        user_input = show_input_dialog("Info", "Silakan tulis nama placemark secara manual\n& tekan Submit untuk melanjutkan.")
        if user_input:
            print(f"Nama placemark yang dimasukkan: {user_input}")
            pyautogui.typewrite(user_input)
    else:
        # Beri nama placemark dengan angka counter
        print(f"Memberi nama placemark: {counter}")
        pyautogui.typewrite(str(counter))

    # Tekan Enter untuk menutup dialog
    print("Menekan Enter untuk menutup dialog")
    pyautogui.press('enter')

def on_press(key):
    global stop_script
    try:
        if key == keyboard.Key.esc:
            stop_script = True
            show_messagebox("Info", "Proses SCRIPT AUTO MARKING telah dihentikan, jalankan ulang program jika ingin memulai proses MARKING lagi\n\nJika anda mau REQ pembuatan program tertentu untuk mempermudah pekerjaan,\nSilahkan hubungi saya di via >>Whatsapp OR Linkedin")
            return False  # Stop listener
    except AttributeError:
        pass

def main():
    global stop_script
    counter = 1
    show_messagebox("Info", "Program ini di gunakan untuk <TAGGING> NOMOR HOMPASS saat berada pada MODE STREETVIEW secara langsung pada aplikasi GOOGLE EARTH PRO\n\nGunakan 'KLIK KANAN' untuk memulai <MARKING> atau 'KLIK TENGAH' untuk menulis nama placemark secara manual\nTekan ESC untuk keluar\n\nProgram ini dibuat oleh: \nSyaiful Wachid > Senior Project Designer\n@PT.Fiberhome Indonesia\nSebagai solusi ondesk survey saat urgent\nLinkedIn Account: Syaiful Wachid\nCP:082230696953")  # Show messagebox once
    while not stop_script:
        print("Menunggu klik kanan atau tengah untuk menambahkan penanda...")
        wait_for_mouse_click()
        if stop_script:
            break
        print(f"Menambahkan penanda dengan nama: {counter}")
        add_placemark(counter)
        if not manual_naming:
            counter += 1
        print(f"Penanda {counter - 1} ditambahkan, menunggu klik berikutnya...")

if __name__ == "__main__":
    # Tunggu beberapa detik untuk memastikan Google Earth Pro aktif
    time.sleep(5)

    # Debug Message
    print("Skrip Python Dimulai")

    # Start keyboard listener
    with keyboard.Listener(on_press=on_press) as listener:
        main()
        listener.join()
 ```
