# Sisop-01-2025-IT08

Anggota IT08:
| Nama | NRP |
| ---------------------- | ---------- |
| Arya Bisma Putra Refman | 5027241036 |
| Moch. Rizki Nasrullah | 5027241038 |
| Nadia Fauziazahra Kusumastuti | 5027241094 |
----

## Soal 2
[Author: Nathan / etern1ty]

Anda merupakan seorang “Observer”, dari banyak dunia yang dibuat dari ingatan yang berbentuk “fragments” - yang berisi kemungkinan yang dapat terjadi di dunia lain. Namun, akhir-akhir ini terdapat anomali-anomali yang seharusnya tidak terjadi, perpindahan “fragments” di berbagai dunia, yang kemungkinan terjadi dikarenakan seorang “Seeker” yang berubah menjadi “Ascendant”, atau dalam kata lain, “God”. Tidak semua “Observer” menjadi “Player”, tetapi disini anda ditugaskan untuk ikut serta dalam menjaga equilibrium dari dunia-dunia yang terbuat dari “Arcaea”. 

A. “First Step in a New World” → Tugas pertama, dikarenakan kejadian “Axiom of The End” yang semakin mendekat, diperlukan sistem untuk mencatat “Player” aktif agar terpisah dari “Observer”. Buatlah dua shell script, `login.sh` dan `register.sh`, yang dimana database “Player” disimpan di `/data/player.csv`. Untuk register, parameter yang dipakai yaitu email, username, dan password. Untuk login, parameter yang dipakai yaitu email dan password.

```bash
nano login.sh && chmod +x login.sh
nano register.sh && chmod +x register.sh
```
- `nano login.sh && chmod +x login.sh`: Membuat script `login.sh` sekaligus mengubah permissionnya menjadi executable
- `nano register.sh && chmod +x register.sh`: Membuat script `register.sh` sekaligus mengubah permissionnya menjadi executable
<br>

B. “Radiant Genesis” → Sistem login/register untuk para "Player" tentunya memiliki constraint, yaitu validasi email dan password. 
Email harus memiliki format yang benar dengan tanda @ dan titik, sementara password harus memiliki minimal 8 karakter, setidaknya satu huruf kecil, satu huruf besar, dan satu angka untuk menjaga keamanan data di dunia “Arcaea”.

C. “Unceasing Spirit” → Karena diperlukan pengecekan keaslian “Player” yang aktif, maka diperlukan sistem untuk pencegahan duplikasi “Player”. Jadikan sistem login/register tidak bisa memakai email yang sama (email = unique), tetapi tidak ada pengecekan tambahan untuk username.

D. “The Eternal Realm of Light” → Password adalah kunci akses ke dunia Arcaea. Untuk menjaga keamanan "Player", password perlu disimpan dalam bentuk yang tidak mudah diakses. Gunakan algoritma hashing sha256sum yang memakai static salt (bebas).

```bash
nano terminal.sh && chmod +x terminal.sh
```
- `nano terminal.sh && chmod +x terminal.sh`: Membuat script `terminal.sh` sekaligus mengubah permissionnya menjadi executable
<br>

-- login.sh --
```bash
#!/bin/bash

DB_PATH="/data/player.csv"
SALT="static_salt"

while true; do
    read -p "Enter email: " email
    read -s -p "Enter password: " password
    echo ""

    hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')
    user_data=$(grep "^$email," "$DB_PATH")

    if [[ -z "$user_data" ]]; then
        echo "❌ Incorrect email or password! Please try again."
        continue
    fi

    stored_password=$(echo "$user_data" | cut -d',' -f3)

    if [[ "$hashed_password" == "$stored_password" ]]; then
        username=$(echo "$user_data" | cut -d',' -f2)
        echo "✅ Login successful! Welcome, $username"
        exit 0
    else
        echo "❌ Incorrect email or password! Please try again."
    fi
done
```
- `DB_PATH="/data/player.csv"` → Menentukan lokasi file database (player.csv) yang menyimpan informasi pengguna
- `SALT="static_salt"` → Mendefinisikan salt statis untuk meningkatkan keamanan hashing password
- `while true; do` → Memulai loop infinite yang akan terus berjalan sampai pengguna berhasil login
	- `read -p "Enter email: " email` → Meminta pengguna memasukkan email
	- `read -s -p "Enter password: " password` → Meminta pengguna memasukkan password secara silent mode (tanpa terlihat di layar)
	- `echo ""` → Mencetak baris kosong untuk estetika
	- `hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')` → Menggabungkan password dengan salt, kemudian mengenkripsinya menggunakan SHA-256
	- `user_data=$(grep "^$email," "$DB_PATH")` → Mencari baris dalam database (player.csv) yang memiliki email yang cocok di awal baris
	- `if [[ -z "$user_data" ]]; then` → Mengecek apakah email ditemukan dalam database
		- `echo "❌ Incorrect email or password! Please try again."` → Jika tidak ditemukan, tampilkan pesan error
 		- `continue` → Kembali ke awal loop untuk meminta input ulang
	- `stored_password=$(echo "$user_data" | cut -d',' -f3)` → Mengambil password terenkripsi yang tersimpan di database (kolom ke-3)
	- `if [[ "$hashed_password" == "$stored_password" ]]; then` → Membandingkan password yang dimasukkan dengan yang tersimpan di database
		- `username=$(echo "$user_data" | cut -d',' -f2)` → Mengambil username (kolom ke-2)
		- `echo "✅ Login successful! Welcome, $username"` → Menampilkan pesan login berhasil
		- `exit 0` → Menghentikan skrip dengan kode sukses
	- `else`
 		- `echo "❌ Incorrect email or password! Please try again."` → Jika password tidak cocok, tampilkan pesan error
<br>

-- register.sh --
```bash
#!/bin/bash

DB_PATH="/data/player.csv"
SALT="static_salt"

touch "$DB_PATH"

validate_email() {
    [[ "$1" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

validate_password() {
    [[ "$1" =~ [A-Z] ]] && [[ "$1" =~ [a-z] ]] && [[ "$1" =~ [0-9] ]] && [[ ${#1} -ge 8 ]]
}

while true; do
    read -p "Enter email: " email
    if ! validate_email "$email"; then
        echo "❌ Invalid email format!"
        continue
    fi

    if grep -q "^$email," "$DB_PATH"; then
        echo "❌ Email is already registered!"
        continue
    fi

    read -p "Enter username: " username
    if [[ -z "$username" ]]; then
        echo "❌ Username cannot be empty!"
        continue
    fi

    while true; do
        read -s -p "Enter password: " password
        echo ""
        if ! validate_password "$password"; then
            echo "❌ Password must have at least 8 characters, 1 uppercase letter, 1 lowercase letter, and 1 number!"
            continue
        fi
        break
    done

    hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')
    echo "$email,$username,$hashed_password" >> "$DB_PATH"

    echo "✅ Registration successful!"
    break
done
```
- `DB_PATH="/data/player.csv"` → Menentukan lokasi file database (player.csv) yang akan menyimpan informasi pengguna
- `SALT="static_salt"` → Mendefinisikan salt statis untuk meningkatkan keamanan hashing password
- `touch "$DB_PATH"` → Membuat file database jika belum ada
- `validate_email()` → Memvalidasi apakah email yang dimasukkan sesuai dengan format email yang benar
	- `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$` → Regex (email harus memiliki format `nama@domain.com`)
- `validate_password()` → Memvalidasi apakah password memenuhi kriteria keamanan
	- `[[ "$1" =~ [A-Z] ]] && [[ "$1" =~ [a-z] ]] && [[ "$1" =~ [0-9] ]] && [[ ${#1} -ge 8 ]]` → Regex (password harus memiliki minimal 8 karakter, harus mengandung huruf besar (A-Z), harus mengandung huruf kecil (a-z), dan harus mengandung angka (0-9)
- `while true; do` → Memulai loop infinite untuk proses registrasi
	- `read -p "Enter email: " email` → Meminta input email dari pengguna
	- `if ! validate_email "$email"; then` → Memeriksa apakah format email valid
 		- `echo "❌ Invalid email format!"` → Jika tidak valid, tampilkan pesan "❌ Invalid email format!"
  		- `continue` → ulangi hingga berhasil 
	- `if grep -q "^$email," "$DB_PATH"; then` → Mengecek apakah email sudah terdaftar
		- `echo "❌ Email is already registered!"` → Jika sudah terdaftar, tampilkan pesan "❌ Email is already registered!" dan ulangi
		- `continue` → ulangi hingga berhasil
	- `read -p "Enter username: " username` → Meminta input username
	- `if [[ -z "$username" ]]; then` → Mengecek apakah username kosong
		- `echo "❌ Username cannot be empty!"` → Jika kosong, tampilkan pesan "❌ Username cannot be empty!"
		- `continue` → ulangi hingga berhasil
	- `while true; do` → Loop untuk memastikan password yang dimasukkan valid
		- `read -s -p "Enter password: " password` → Meminta input password dalam mode silent (tidak ditampilkan)
		- `if ! validate_password "$password"; then` → Memeriksa apakah password memenuhi kriteria keamanan
			- `echo "❌ Password must have at least 8 characters, 1 uppercase letter, 1 lowercase letter, and 1 number!"` → Jika tidak memenuhi, tampilkan pesan error
   			- `continue` → ulangi hingga berhasil
	- `hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')` → Mengenkripsi password dengan SHA-256 setelah ditambahkan salt
	- `echo "$email,$username,$hashed_password" >> "$DB_PATH"` → Menyimpan data pengguna (email, username, hashed password) ke dalam database
	- `echo "✅ Registration successful!"` → Menampilkan pesan sukses
	- `break` → Menghentikan loop registrasi setelah sukses
<br>
  
-- terminal.sh --
```bash
#!/bin/bash

clear

cat << "EOF"
   ___   ___  ________   _______     ______  __________________  ___
  / _ | / _ \/ ___/ _ | / __/ _ |   / __/\ \/ / __/_  __/ __/  |/  /
 / __ |/ , _/ /__/ __ |/ _// __ |  _\ \   \  /\ \  / / / _// /|_/ /
/_/ |_/_/|_|\___/_/ |_/___/_/ |_| /___/   /_/___/ /_/ /___/_/  /_/
⠀⠀⠀⠀⠀⠀⠀⠀⠀
EOF

echo "          ┌───────────────────────────────────────┐"
echo "          │            ARCAEA TERMINAL            │"
echo "          ├───────┬───────────────────────────────┤"
echo "          │  ID   │ OPTION                        │"
echo "          ├───────┼───────────────────────────────┤"
echo "          │   1   │ Register New Account          │"
echo "          │   2   │ Login to Existing Account     │"
echo "          │   3   │ Exit Arcaea Terminal          │"
echo "          └───────┴───────────────────────────────┘"

printf "\n> Enter option [1-3]: "
read choice

case $choice in
    1)
        bash register.sh
        ;;
    2)
        bash login.sh
        if [ $? -eq 0 ]; then
            bash ./scripts/manager.sh
        fi
        ;;
    3)
        echo -e "\n👋 Exiting Arcaea Terminal..."
        exit 0
        ;;
    *)
        echo -e "\n❌ Invalid choice. Please enter a number between 1 and 3."
        ;;
esac
```
- `clear` → Menghapus tampilan sebelumnya agar layar bersih sebelum menampilkan menu baru
- `cat << "EOF" ... EOF` → Menampilkan ASCII art sebagai header
- `echo "..."` → Menampilkan menu utama dengan opsi
	- Opsi yang tersedia:
		- 1 → Register akun baru
		- 2 → Login ke akun yang sudah ada
		- 3 → Keluar dari Arcaea Terminal
- `printf "\n> Enter option [1-3]: "` → Meminta input pengguna
	- `read choice` → Menyimpan pilihan pengguna dalam variabel choice
- `case $choice in` → Mengeksekusi perintah berdasarkan pilihan pengguna
	- `1)` `bash register.sh` → Menjalankan `register.sh` untuk registrasi akun baru
	- `2)` `bash login.sh` → Menjalankan `login.sh` untuk login pengguna
		- `if [ $? -eq 0 ]; then` → Jika login berhasil
  		- `bash ./scripts/manager.sh` → maka `manager.sh` akan dijalankan
	- `3)` `echo -e "\n👋 Exiting Arcaea Terminal..."` → Menampilkan pesan keluar dan mengakhiri skrip
	- `*)` `echo -e "\n❌ Invalid choice. Please enter a number between 1 and 3."` → Menampilkan pesan error jika input tidak valid
<br>

E. “The Brutality of Glass” → Setelah sukses login, "Player" perlu memiliki akses ke sistem pemantauan sumber daya. Sistem harus dapat melacak penggunaan CPU (dalam persentase) yang menjadi representasi “Core” di dunia “Arcaea”. Pastikan kalian juga bisa melacak “terminal” yang digunakan oleh “Player”, yaitu CPU Model dari device mereka.

Lokasi shell script: `./scripts/core_monitor.sh`

```bash
nano ./scripts/core_monitor.sh && chmod +x ./scripts/core_monitor.sh
```
<br>

F. “In Grief and Great Delight” → Selain CPU, “fragments” juga perlu dipantau untuk memastikan equilibrium dunia “Arcaea”. RAM menjadi representasi dari “fragments” di dunia “Arcaea”, yang dimana dipantau dalam persentase usage, dan juga penggunaan RAM sekarang. 

Lokasi shell script: `./scripts/frag_monitor.sh`

```bash
nano ./scripts/frag_monitor.sh && chmod +x ./scripts/frag_monitor.sh
```
<br>

G. “On Fate's Approach” → Pemantauan yang teratur dan terjadwal sangat penting untuk mendeteksi anomali. Crontab manager (suatu menu) memungkinkan "Player" untuk mengatur jadwal pemantauan sistem. 
- Hal yang harus ada di fungsionalitas menu:
	- Add/Remove CPU [Core] Usage
	- Add/Remove RAM [Fragment] Usage
	- View Active Jobs

Lokasi shell script: `./scripts/manager.sh`

```bash
nano ./scripts/manager.sh && chmod +x ./scripts/manager.sh
```
<br>

H. “The Disfigured Flow of Time” → Karena tentunya script yang dimasukkan ke crontab tidak mengeluarkan output di terminal, buatlah 2 log file, core.log dan fragment.log di folder ./log/, yang dimana masing-masing terhubung ke program usage monitoring untuk usage tersebut. 
- Format log:
	- CPU
		- `[YYYY-MM-DD HH:MM:SS] - Core Usage [$CPU%] - Terminal Model [$CPU_Model]`
	- RAM
		- `[YYYY-MM-DD HH:MM:SS] - Fragment Usage [$RAM%] - Fragment Count [$RAM MB] - Details [Total: $TOTAL MB, Available: $AVAILABLE MB]`
<br>

-- core_monitor.sh --
```bash
#!/bin/bash

LOG_FILE="$(dirname "$0")/../.logs/core.log"

timestamp=$(date "+%Y-%m-%d %H:%M:%S")
cpu_usage=$(awk '{u=$2+$4; t=$2+$4+$5; if (NR==1){u1=u; t1=t;} else print ($2+$4-u1) * 100 / (t-t1) "%"; }' \
<(grep 'cpu ' /proc/stat) <(sleep 1; grep 'cpu ' /proc/stat))
cpu_model=$(lscpu | grep "Model name" | cut -d':' -f2 | xargs)

echo "[$timestamp] - Core Usage [$cpu_usage] - Terminal Model [$cpu_model]" >> "$LOG_FILE"
```
- `LOG_FILE="$(dirname "$0")/../.logs/core.log"` → Menentukan lokasi file log di dalam direktori `.logs/core.log`
	- Variabel `LOG_FILE` menyimpan path file log tempat data akan dicatat
	- Menggunakan `$(dirname "$0")` untuk memastikan path relatif terhadap lokasi script
 - `timestamp=$(date "+%Y-%m-%d %H:%M:%S")` → Menggunakan perintah date untuk mendapatkan waktu dan tanggal saat ini dalam format `YYYY-MM-DD HH:MM:SS`
 - `cpu_usage=$(awk '{u=$2+$4; t=$2+$4+$5; if (NR==1){u1=u; t1=t;} else print ($2+$4-u1) * 100 / (t-t1) "%"; }' <(grep 'cpu ' /proc/stat) <(sleep 1; grep 'cpu ' /proc/stat))`
	- Mengambil informasi CPU dari `/proc/stat` sebelum dan sesudah jeda 1 detik
	- `awk` menghitung persentase penggunaan CPU berdasarkan perubahan nilai user dan system time dibandingkan dengan total time
 - `cpu_model=$(lscpu | grep "Model name" | cut -d':' -f2 | xargs)`
	- Menggunakan perintah `lscpu` untuk mendapatkan informasi model CPU
	- `grep "Model name"` menyaring hanya baris yang berisi model prosesor
	- `cut -d':' -f2` mengambil nilai setelah titik dua ':'
	- `xargs` membersihkan spasi yang tidak perlu.
 - `echo "[$timestamp] - Core Usage [$cpu_usage] - Terminal Model [$cpu_model]" >> "$LOG_FILE"`
 	- Mencetak hasil ke dalam file log yang telah ditentukan sebelumnya dengan format `[YYYY-MM-DD HH:MM:SS] - Core Usage [$CPU%] - Terminal Model [$CPU_Model]`
<br>

-- frag_monitor.sh --
```bash
#!/bin/bash

LOG_FILE="$(dirname "$0")/../.logs/fragment.log"

timestamp=$(date "+%Y-%m-%d %H:%M:%S")
ram_usage=$(free | awk '/Mem/ {printf "%.2f", $3/$2 * 100}')
total_ram=$(free -m | awk '/Mem/ {print $2}')
available_ram=$(free -m | awk '/Mem/ {print $7}')

echo "[$timestamp] - Fragment Usage [$ram_usage%] - Fragment Count [$available_ram MB] - Details [Total: $total_ram MB, Available: $available_ram MB]" >> "$LOG_FILE"
```
- `LOG_FILE="$(dirname "$0")/../.logs/core.log"` → Menentukan lokasi file log di dalam direktori `.logs/core.log`
	- Variabel `LOG_FILE` menyimpan path file log tempat data akan dicatat
	- Menggunakan `$(dirname "$0")` untuk memastikan path relatif terhadap lokasi script
- `timestamp=$(date "+%Y-%m-%d %H:%M:%S")` → Menggunakan perintah date untuk mendapatkan waktu dan tanggal saat ini dalam format `YYYY-MM-DD HH:MM:SS`
- `ram_usage=$(free | awk '/Mem/ {printf "%.2f", $3/$2 * 100}')`
 	- Menggunakan perintah `free` untuk mendapatkan informasi RAM
	- `awk` menghitung persentase RAM yang sedang digunakan dengan rumus `(RAM digunakan / RAM total) * 100`
	- Hasilnya diformat dengan dua angka di belakang desimal
- `total_ram=$(free -m | awk '/Mem/ {print $2}')`
 	- `free -m` menampilkan informasi RAM dalam satuan MB
  	- `awk '/Mem/ {print $2}'` mengambil total RAM  
- `available_ram=$(free -m | awk '/Mem/ {print $7}')`
	- `awk '/Mem/ {print $7}'` mengambil jumlah RAM yang masih tersedia
- `echo "[$timestamp] - Fragment Usage [$ram_usage%] - Fragment Count [$available_ram MB] - Details [Total: $total_ram MB, Available: $available_ram MB]" >> "$LOG_FILE"`
	- Mencetak hasil ke dalam file log yang telah ditentukan sebelumnya dengan format `[YYYY-MM-DD HH:MM:SS] - Fragment Usage [$RAM%] - Fragment Count [$RAM MB] - Details [Total: $TOTAL MB, Available: $AVAILABLE MB]`
<br>

-- manager.sh --
```bash
#!/bin/bash

SCRIPT_DIR=$(dirname "$(realpath "$0")")

while true; do
        clear

        cat << "EOF"

  ___             _        _      __  __
 / __|_ _ ___ _ _| |_ __ _| |__  |  \/  |__ _ _ _  __ _ __ _ ___ _ _
| (__| '_/ _ \ ' \  _/ _` | '_ \ | |\/| / _` | ' \/ _` / _` / -_) '_|
 \___|_| \___/_||_\__\__,_|_.__/ |_|  |_\__,_|_||_\__,_\__, \___|_|
                                                       |___/

     ┌───────────────────────────────────────────────────────┐
     │                  ARCAEA TERMINAL                      │
     ├───────┬───────────────────────────────────────────────┤
     │  ID   │ OPTION                                        │
     ├───────┼───────────────────────────────────────────────┤
     │   1   │ Add CPU - Core Monitor to Crontab             │
     │   2   │ Add RAM - Fragment Monitor to Crontab         │
     │   3   │ Remove CPU - Core Monitor from Crontab        │
     │   4   │ Remove RAM - Fragment Monitor from Crontab    │
     │   5   │ View All Scheduled Monitoring Jobs            │
     │   6   │ Exit Arcaea Terminal                          │
     └───────┴───────────────────────────────────────────────┘
EOF

        printf "\n> Enter option [1-6]: "
        read choice

        case $choice in
        1)
                (crontab -l 2>/dev/null; echo "* * * * * /bin/bash $SCRIPT_DIR/core_monitor.sh") | crontab -
                echo "✅ CPU Monitoring added to crontab."
                ;;
        2)
                (crontab -l 2>/dev/null; echo "* * * * * /bin/bash $SCRIPT_DIR/frag_monitor.sh") | crontab -
                echo "✅ RAM Monitoring added to crontab."
                ;;
        3)
                crontab -l | grep -v "$SCRIPT_DIR/core_monitor.sh" | crontab -
                echo "✅ CPU Monitoring removed from crontab."
                ;;
        4)
                crontab -l | grep -v "$SCRIPT_DIR/frag_monitor.sh" | crontab -
                echo "✅ RAM Monitoring removed from crontab."
                ;;
        5)
                echo "📋 Active crontab schedules:"
                crontab -l
                ;;
        6)
                echo "👋 Exiting..."
                exit 0
                ;;
        *)
                echo "❌ Invalid choice."
                ;;
        esac

    printf "\nPress Enter to continue..."
    read
done
```
- `SCRIPT_DIR=$(dirname "$(realpath "$0")")`
	- Mengambil path absolut dari direktori script yang sedang berjalan
	- Ini memastikan bahwa perintah yang dijalankan akan selalu menggunakan path yang benar
- `while true; do ... done`: Setelah memilih salah satu opsi, kembali memilih menu
- `cat << "EOF" ... EOF`
 	- Menampilkan menu interaktif dalam format ASCII art
	- Memberikan daftar opsi yang dapat dipilih oleh pengguna
- `printf "\n> Enter option [1-6]: " ... read choice`
 	- Meminta pengguna memasukkan pilihan dari menu
	- Disimpan dalam variabel `choice` untuk diproses lebih lanjut
- `crontab -l 2>/dev/null; echo "* * * * * /bin/bash $SCRIPT_DIR/core_monitor.sh") | crontab -`
	- Menambahkan script monitoring CPU agar dijalankan setiap menit oleh crontab
	- Jika crontab kosong, perintah akan tetap berjalan tanpa error dengan `2>/dev/null` untuk menangani error yang mungkin muncul
- `(crontab -l 2>/dev/null; echo "* * * * * /bin/bash $SCRIPT_DIR/frag_monitor.sh") | crontab -`
	- Menambahkan script monitoring RAM agar dijalankan setiap menit oleh crontab
	- Jika crontab kosong, perintah akan tetap berjalan tanpa error dengan `2>/dev/null` untuk menangani error yang mungkin muncul
- `crontab -l 2>/dev/null | grep -v "$SCRIPT_DIR/core_monitor.sh" | crontab -`
	- Menghapus hanya baris yang berisi `core_monitor.sh` dari crontab
	- Menggunakan `grep -v` untuk menyaring crontab dan memperbaruinya tanpa menghapus tugas lainnya
	- Jika crontab kosong, perintah tetap berjalan tanpa error
- `crontab -l | grep -v "$SCRIPT_DIR/frag_monitor.sh" | crontab -`
	- Menghapus hanya baris yang berisi `frag_monitor.sh` dari crontab
	- Menggunakan `grep -v` untuk menyaring crontab dan memperbaruinya tanpa menghapus tugas lainnya
	- Jika crontab kosong, perintah tetap berjalan tanpa error
- `crontab -l 2>/dev/null`
 	- Menampilkan semua tugas yang sedang berjalan di crontab
- `exit 0`
	- Mengakhiri eksekusi script dan kembali ke shell

--- OUTPUT ---
![image](https://github.com/user-attachments/assets/8d05e23e-998a-4067-9e48-8788cbe9050e)
<br>

- REGISTER
![image](https://github.com/user-attachments/assets/c43fb4a6-69e5-4d26-a56b-482d35beadb1)
<br>

- LOGIN
![image](https://github.com/user-attachments/assets/997c4578-64ff-4513-8532-379fc348c4ce)
<br>

- ADD CPU
![image](https://github.com/user-attachments/assets/0d8bb557-99ff-4d7b-93b6-7369ab24dff6)
<br>

![image](https://github.com/user-attachments/assets/6484e7a1-024f-40db-8438-03eecaa8142f)


- ADD RAM
![image](https://github.com/user-attachments/assets/b05c8c38-55bb-4bf4-bc05-692b6156b70b)
<br>

![image](https://github.com/user-attachments/assets/7f0755fe-4c07-44f0-8f14-969071b30a18)

- btop
![image](https://github.com/user-attachments/assets/04aadb03-ea43-4f71-929e-ca13c42a1268)
	- Jika kita bandingkan hasil btop dengan CPU dan RAM, dapat dilihat bahwa hasil monitoring CPU dan RAM sudah akurat
<br>
<br>

## Soal 4
[Author: Amoes / winter]

Pada suatu hari, anda diminta teman anda untuk membantunya mempersiapkan diri untuk turnamen Pokemon “*Generation 9 OverUsed 6v6 Singles*” dengan cara membuatkan tim yang cocok untuknya. Tetapi, anda tidak memahami meta yang dimainkan di turnamen tersebut. Untungnya, seorang informan memberikan anda data [pokemon_usage.csv](https://drive.google.com/file/d/1n-2n_ZOTMleqa8qZ2nB8ALAbGFyN4-LJ/view?usp=sharing) yang bisa anda download dan analisis. 

- Data tersebut memiliki banyak kolom:
    - Nama Pokemon
    - Usage% yang merupakan persentase Pokemon yang disesuaikan dengan Rank pengguna dan Winrate
    - Raw Usage yang merupakan jumlah mentah Pokemon dalam semua tim yang tercatat
    - Type1 dan Type2 Pokemon
    - Statistic Pokemon: HP, Atk, Def, SpAtk, SpDef, dan Speed

Untuk menganalisis data tersebut dengan baik, anda berpikiran untuk membuat script yang bernama pokemon_analysis.sh

```bash
wget "https://drive.usercontent.google.com/u/0/uc?id=1n-2n_ZOTMIeqa8qZ2nB8ALAbGFyN4-LJ&export=download" -O pokemon_usage.csv
```
- `wget`: Perintah dalam Linux untuk mengunduh file dari internet
- `"https://drive.usercontent.google.com/u/0/uc?id=1n-2n_ZOTMIeqa8qZ2nB8ALAbGFyN4-LJ&export=download"`: Link file yang akan diunduh dari Google Drive
- `-O pokemon_usage.csv`: Menentukan nama file hasil unduhan sebagai `pokemon_usage.csv`
- Perintah ini akan mengunduh file dari Google Drive dan menyimpannya dengan nama `pokemon_usage.csv`
<br>

```bash
nano pokemon_analysis.sh && chmod +x pokemon_analysis.sh
```
- `nano`: Editor teks berbasis terminal di Linux
- `pokemon_analysis.sh`: Nama file skrip shell yang akan dibuat dan diedit menggunakan `nano`
- `chmod +x pokemon_analysis.sh`: Mengatur script menjadi executable
- Perintah ini membuka editor `nano` untuk membuat atau mengedit skrip bernama `pokemon_analysis.sh`

dengan fitur sebagai berikut:

```bash
#!/bin/bash

LINE=$(printf '=%.0s' $(seq 1 $(tput cols)))

print_centered() {
    local termwidth=$(tput cols)
    local padding=$(((termwidth - ${#1}) / 2))
    printf "%*s%s%*s\n" $padding "" "$1" $padding ""
}

show_help_ascii() {
    cat << "EOF"
⠀⠀⠀⠀⠀⠀⣤⣶⢦⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀           ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠈⠙⢾⡱⣟⢿⣳⣶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠪⡷⣭⣟⣿⣿⣿⣷⣶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠳⢟⣾⣿⣿⣿⣿⣿⣿⣿⣷⣶⣤⣄⡀⠀⠀⠀⠀⠀⠀⢀⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀      ______ _____ _   __ ________  ________ _   _⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠿⠽⠿⠿⠿⣿⣿⣿⣿⣿⣿⣿⣷⣦⣤⣀⠀⠀⢸⡿⣷⣄⡤⠶⡤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀ | ___ \  _  | | / /|  ___|  \/  |  _  | \ | |⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠛⠻⢿⣿⣿⣿⣿⣿⣶⣾⣿⣯⣷⣾⣷⣷⣧⣝⣓⠶⢤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀| |_/ / | | | |/ / | |__ | .  . | | | |  \| |⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⣻⣿⣿⣿⣿⣿⣿⣽⣫⢿⣿⡿⣿⣿⣿⣷⣮⣼⣗⣲⣒⠲⠤⠤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀|  __/| | | |    \ |  __|| |\/| | | | | . ` |⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⠶⣏⣷⣿⣿⣿⣿⡿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣿⣿⣿⣿⣿⣿⡿⣜⠢⡍⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀| |   \ \_/ / |\  \| |___| |  | \ \_/ / |\  |⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⡴⣁⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣩⣿⣿⡥⢣⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀\_|    \___/\_| \_/\____/\_|  |_/\___/\_| \_/⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡴⣳⠛⢼⡻⣏⡯⣽⡹⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠿⢙⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⠮⣵⠋⣴⢯⣳⣳⢟⡲⣝⠶⣹⣻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢿⣿⡿⠿⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⢏⣽⠃⣷⣛⢮⣵⣛⢮⣳⣽⣞⡵⣣⠟⣿⣿⢿⡻⣿⢿⣿⣏⠉⠁⠀⠀⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀ ⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣏⢾⠃⣼⡳⣭⡟⡮⣧⣿⣿⣿⣿⣿⡵⢫⡝⣿⣯⠷⣭⢏⡟⣿⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀ ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡞⡼⡏⢰⣯⢟⢧⣻⣿⣿⣿⣿⣿⠿⢿⣿⡳⡜⡹⣿⣿⣷⡟⣾⡱⢿⣆⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀ ⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⢯⡽⣠⡿⢙⡾⢯⣿⣿⣿⣿⡿⠁⠀⠀⠙⣷⡹⠴⡹⣿⣟⣿⢶⣫⢟⣿⣾⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀ ⠀ ⠀⠀⠀⠀⠀⠀⠀⢸⣺⢟⢵⡵⢹⣙⢯⣿⣿⣿⣿⠃⠀⠀⠀⠀⠈⢷⣩⢳⢹⣿⣻⣧⣛⢾⣿⡷⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣑⣲⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀ ⠀⠀⠀⠀⠀⠀⠀⠀⣠⢞⣱⡟⢹⡝⣢⢫⢶⡹⢿⣿⣿⡀⠀⠀⠀⠀⠀⠈⢧⢎⡳⣜⢿⣽⡾⣿⣿⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⢷⣹⣥⡀⠙⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠾⠥⢺⡟⣷⠸⡫⡜⣫⢮⡝⣯⢿⣿⢧⠀⠀⠀⠀⠀⠀⠀⠉⠓⣿⣮⠿⢽⣮⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣿⣿⣿⣿⣦⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⡿⣼⠐⢴⣹⢣⡗⢾⣩⢞⡹⢯⣧⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣤⣾⣿⡉⢉⣛⣻⣿⡛⠢⢄⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢹⡞⢇⠸⣧⡛⣼⢣⠟⣮⡝⣧⢛⡷⣄⠀⠀⠀⠀⠀⠀⠀⠘⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠸⣿⣿⣿⣿⣿⣿⣿⣿⣷⣤⣼⣿⠆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢏⣸⡆⠻⣝⢷⣋⣟⡲⣝⡎⡷⡹⢜⡳⢤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⣿⣿⣿⡿⠁⠀⠉⠛⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡰⣻⣧⠀⠀⠀⠀⠀⠈⢧⣹⣶⠙⣾⠵⣌⠵⢮⣝⢲⡙⡎⣕⣣⢿⠳⡍⢯⣙⢲⠲⡤⢤⣀⣤⢤⡤⣄⡀⠀⠀⠀⠀⠀⠀⠀⣿⢿⣱⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣞⣿⣿⣿⣧⣀⣀⣠⣴⣶⣾⣧⣻⢧⡈⠻⣭⡳⣆⡉⢒⡙⣴⡿⡙⢎⡱⢎⡱⣘⠦⢣⡙⠢⠍⢭⠳⣝⣖⣻⣆⡀⠀⠀⠀⠀⢠⡟⣧⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⣠⣿⣿⣿⣿⣿⣿⣿⣿⣿⠿⣿⣿⣿⣿⣮⢟⣦⡌⠳⢷⡸⡠⢼⡟⡖⡹⣘⠴⢣⡱⢮⠼⡑⡎⠽⣘⠦⣡⣿⣿⢣⠝⣫⢷⣦⣄⣠⢯⡝⣮⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⢀⣼⣻⣿⣿⣿⣿⣿⣿⣿⣿⣏⣿⣏⠉⢿⢯⣽⣿⣼⡹⢷⣄⡙⠣⣿⡹⢔⢣⣱⣮⣷⣾⣮⣶⣩⣼⣷⣾⣿⣿⠿⡿⣿⣾⣥⡚⢬⡙⠿⣎⡷⣃⣾⣥⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⣠⣿⣿⣷⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⣿⣾⣦⠈⣝⢿⣿⣿⣿⣧⡯⡽⢶⣿⣷⣯⣼⡿⡹⢀⢆⢣⣛⣿⣿⡛⡽⡘⢦⡙⣔⠣⡭⣛⢿⣷⣯⣜⠢⢝⣻⣿⣿⣿⣷⣦⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⣷⣿⡷⡿⠛⠙⣿⣽⣿⣧⣿⡽⣻⣿⢖⣑⠢⡘⢦⣓⢾⣿⡵⢣⡝⣢⢕⠪⡕⢲⡡⢞⢈⣻⣿⡿⠿⠟⡛⠿⣿⣿⣿⣿⣿⡦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣞⣧⣴⡿⠀⠀⠀⠉⢳⣷⣿⣿⠹⣏⣷⢳⢎⣿⣿⣝⢧⣛⠶⣩⠳⣜⣡⠞⣴⣿⠟⡉⠆⠱⡈⢌⠣⢝⠻⣿⣿⣿⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢀⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡟⠛⢡⣿⣿⣿⣿⡀⠀⠀⠀⠀⠀⠀⠸⣿⡿⢎⡽⣚⣾⣿⣿⣿⣿⣿⣿⣾⣽⣣⢟⢦⣣⣾⣿⢣⠓⣌⡘⡐⠐⡂⢡⠊⢣⣓⢻⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣦⣿⣿⣿⣿⣷⢳⣤⣀⠀⠀⠀⠀⢠⡟⡱⣏⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣾⣿⡟⣆⠣⡝⢢⠗⣌⠱⠠⢡⠘⢠⠘⢧⡚⣧⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠙⠿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠉⠛⢿⣗⢮⡜⣹⢒⠦⣼⡑⣜⣱⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣏⡌⠳⣭⡜⣡⣫⡜⣢⠑⣂⠡⢂⡉⠔⢫⡖⣳⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⢾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠛⠛⠛⠋⠉⠁⠀⠀⠀⠈⠳⢝⡦⣏⡚⡴⢻⣖⡟⠉⠛⠿⣿⣿⣿⡿⢿⡿⢿⡿⣿⢿⣿⣿⣿⣿⣦⡠⢙⣤⢦⣙⡑⢯⡄⢊⠤⠐⡉⠤⢹⡒⣿⣷⣦⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⢼⣝⡾⣿⠏⠀⠀⠀⠀⠀⠉⠻⢽⡷⣾⣥⣻⣼⣿⠟⠋⠉⠺⣿⢢⣿⣋⠶⣥⢛⣦⢙⣦⠌⡁⢆⠑⡂⢟⡴⣻⣿⣿⣦⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠑⠫⠷⠋⠁⠀⠀⠀⠀⠘⢿⣳⢎⡟⣴⢫⡔⢧⡘⣦⠑⢢⠈⡔⢊⢶⡹⣿⣿⣿⣿⣷⣦⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⢸⣎⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢻⢮⡝⢶⡓⣮⢓⡗⣼⠑⢢⠑⢨⠘⣦⢛⡟⣿⣿⣿⣿⣿⣿⢷⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠘⣿⡎⣿⣿⣿⣿⣿⣿⣿⣿⣿⡟⣿⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣟⢞⣣⠟⣼⣩⢾⣸⢍⠢⡘⠤⡙⢴⢫⣽⣯⣛⢿⡿⢿⢯⣿⣯⣟⣷⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣤
⠀⠀⠀⢻⣿⢸⣿⣿⣿⣿⣿⣿⣻⢷⣻⢼⡹⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣠⡤⣤⣤⡴⣒⡢⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠸⢯⣜⠻⡰⢧⢇⡾⢎⡔⢡⠒⣍⢎⡗⣾⠈⠛⠷⣾⣿⣷⣶⣬⣛⢾⣽⡾⣽⣦⢤⡀⠀⠀⠀⠀⠀⠀⣀⣀⣤⣴⣶⡾⣟⢷⣿⠃
⠀⠀⠀⠈⢿⣎⣿⡿⣟⣯⣷⢿⣽⣻⣽⢎⡷⣹⢳⣄⡀⠀⠀⠀⠀⠀⠀⠀⣰⣿⠿⡿⢟⣛⣛⠻⣩⢙⠭⣻⡬⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡿⣦⡉⢳⣛⣥⢾⣹⠲⣌⠧⡹⣌⠞⡼⡗⠀⠀⠀⠀⠉⠙⠻⠿⣛⡷⡝⢿⣳⢯⣟⡿⣿⣶⣤⣴⣾⣿⣽⣿⣻⡽⣶⣻⡼⣿⠃⠀
⠀⠀⠀⠀⠈⢧⣻⡿⣽⣻⣞⣯⣟⣷⣻⣮⡱⢏⡷⣭⣛⣦⣀⠀⠀⠀⠀⢠⣿⣏⢯⣓⠯⣖⣣⠷⣱⢎⡵⢢⢻⣽⣧⡀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣞⡳⣝⡶⢋⣰⡽⢎⡳⣥⠻⣔⢫⡕⢮⣹⢳⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⠚⠹⢿⣮⣽⣳⣿⢯⠿⡽⢯⣟⣾⢷⣿⣳⣯⡿⠃⠀⠀
⠀⠀⠀⠀⠀⠈⢦⡙⢷⠟⢊⠓⠻⢾⣳⢯⣷⣡⠛⡲⣝⡶⣹⣛⡶⣤⣤⡾⡿⣜⣧⣫⣝⣮⡵⢯⢷⣮⢷⢯⢷⠻⣓⣷⣤⣀⣀⣤⣤⠶⣚⣫⣭⣭⢭⣉⠰⣔⢯⡜⣏⡳⣭⢳⢎⡳⣜⠳⣜⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠙⠛⠺⣿⣽⣧⢿⡼⣯⢾⡷⣯⣿⠷⠖⠀
⠀⠀⠀⠀⠀⠀⠀⠻⡆⣿⢻⢿⡷⣦⣌⠛⢶⣏⡷⣅⠲⣘⠱⣋⣶⡟⣧⢻⡽⣹⢎⢷⡹⢶⡙⢯⠞⡬⢋⠞⣩⣳⣿⢻⡼⣹⠞⣪⡵⣞⢯⡳⢞⣉⠲⣌⡧⣹⢮⡝⣎⢷⣙⢮⢣⠗⣮⢛⡞⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⢿⣻⣷⣭⡳⣝⡞⣽⣆⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠈⠾⣯⢞⡵⣫⢟⡿⣦⡙⢾⣭⣛⠶⣣⣼⡟⣼⢣⣏⠶⣍⡞⣬⢓⣣⠚⡤⢋⠴⣉⠎⣼⣿⢧⢻⣴⢣⣞⢷⡹⢎⡳⣜⡖⣮⢛⡼⣡⡟⣧⢻⡜⣧⣛⢮⢏⡾⣱⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠛⠾⣿⣾⣼⣡⠿⣧⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠻⣜⢧⡻⣜⣳⢻⣆⠱⣯⢽⣡⣿⡝⡶⣫⡜⣏⣞⡹⣜⡏⣶⢫⢗⣫⢖⡳⣌⢿⣯⠯⠾⠇⢾⡭⢞⡭⣏⠷⣎⣽⠶⢋⣶⠯⣝⣎⢧⡻⣴⣫⢞⣳⠞⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠛⠯⢷⣿⣧
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠳⣽⡜⣧⢻⠂⣬⡙⣒⣿⣽⡵⣧⢿⣜⣞⣷⣻⣞⡿⢯⠿⣛⣿⣿⣿⢯⣷⣬⡴⢶⣌⣛⡛⣒⣛⣛⣩⣤⢾⡻⣥⢻⡜⣮⢷⣻⠷⣹⠞⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠛⠶⣿⣿⣿⣿⣻⣟⣿⣯⣿⣽⣯⣿⣷⣿⣿⣿⣿⣿⣿⣿⢏⡾⣹⣷⣝⢧⣫⠽⣹⢭⢞⣭⢳⡹⢮⡵⣯⢷⣻⢯⡯⠗⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠙⠛⠿⠿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⣾⣵⣧⣿⣿⣾⣵⡿⣧⣿⢾⡾⣟⣿⣟⡿⠽⠛⠊⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⢿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣿⣽⣿⣾⣷⣯⣟⡝⠛⠉⠉⠉⠉⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠻⣼⡷⡞⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⣟⣿⣟⣿⣽⠳⣌⣳⡿⣽⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠛⠛⠛⠿⠿⠿⠿⠯⠿⠿⠽⠿⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀

EOF
}

# Fungsi untuk ASCII Art setiap fitur
show_info_ascii() {
    cat << "EOF"
                                                    ⢀⣴⡆
                                                   ⢀⣾⣿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⣿⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡾⠋⠉⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⠃⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⣠⠖⠲⢤⡖⠒⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⠀⢀⡏⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⡏⠀⠀⠀⠀⠀⢀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⢀⣀⠀⠀⢸⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⠸⣄⠀⠁⣠⠞⠉⠀⠀⠀⠀⠀⠀⠀⠀⢀⣀⣀⣠⣤⣤⣤⣤⠀⠀
                                        ⠀⡞⠉⠻⠁⢹⠀⠀⡏⠀⠀⠀⠀⢸⠃⠀⠀⠀⠀⠀⠀⠀⠀⠹⣶⠋⠀⠀⠀⠀⣀⡤⠴⠒⠊⠉⠉⠀⠀⣿⣿⣿⠿⠋⠀⠀
                                        ⠀⠳⢤⡀⠀⡞⠁⠀⡇⠀⠀⢀⡠⠼⠴⠒⠒⠒⠒⠦⠤⠤⣄⣀⠀⢀⣠⠴⠚⠉⠀⠀⠀⠀⠀⠀⠀⠀⣼⠿⠋⠁⠀⠀⠀⠀
                                        ⠀⠀⠀⠈⠷⡏⠀⠀⣇⠔⠂⠈⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢨⠿⠋⠀⠀⠀⠀⠀⠀⠀⠀⣀⡤⠖⠋⠁⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⢰⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⠤⠒⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⢀⡟⠀⣠⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⢻⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⡤⠤⢴
                                        ⠀⠀⠀⠀⠀⠀⣸⠁⣾⣿⣀⣽⡆⠀⠀⠀⠀⠀⠀⠀⢠⣾⠉⢿⣦⠀⠀⠀⢸⡀⠀⠀⢀⣠⠤⠔⠒⠋⠉⠉⠀⠀⠀⠀⢀⡞
                                        ⠀⠀⠀⠀⠀⢀⡏⠀⠹⠿⠿⠟⠁⠀⠰⠦⠀⠀⠀⠀⠸⣿⣿⣿⡿⠀⠀⠀⢘⡧⠖⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡼⠀
                                        ⠀⠀⠀⠀⠀⣼⠦⣄⠀⠀⢠⣀⣀⣴⠟⠶⣄⡀⠀⠀⡀⠀⠉⠁⠀⠀⠀⠀⢸⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⠁⠀
                                        ⠀⠀⠀⠀⢰⡇⠀⠈⡇⠀⠀⠸⡾⠁⠀⠀⠀⠉⠉⡏⠀⠀⠀⣠⠖⠉⠓⢤⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⠃⠀⠀
                                        ⠀⠀⠀⠀⠀⢧⣀⡼⠃⠀⠀⠀⢧⠀⠀⠀⠀⠀⢸⠃⠀⠀⠀⣧⠀⠀⠀⣸⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡰⠃⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠈⢧⡀⠀⠀⠀⠀⠘⣆⠀⠀⠀⢠⠏⠀⠀⠀⠀⠈⠳⠤⠖⠃⡟⠀⠀⠀⢾⠛⠛⠛⠛⠛⠛⠛⠛⠁⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⠙⣆⠀⠀⠀⠀⠈⠦⣀⡴⠋⠀⠀⠀⠀⠀⠀⠀⠀⢀⣼⠙⢦⠀⠀⠘⡇⠀⠀⠀⠀⠀⠀⢀⣀⠀⠀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⢠⡇⠙⠦⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⠴⠋⠸⡇⠈⢳⡀⠀⢹⡀⠀⠀⠀⢀⡞⠁⠉⣇⣀⠀⠀
                                        ⠀⠀⠀⠀⠀⠀⠀⡼⣀⠀⠀⠈⠙⠂⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠀⠀⠀⠀⣷⠴⠚⠁⠀⣀⣷⠀⠀⠀⢠⠇⠀⠀⠀⠀⠀⣳
                                        ⠀⠀⠀⠀⠀⠀⡴⠁⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣆⡴⠚⠉⠉⠀⠀⠀⠀⢸⠃⣀⣠⠤⠤⠖⠋
                                        ⣼⢷⡆⠀⣠⡴⠧⣄⣇⠀⠀⠀⠀⡴⠚⠙⠲⠞⠛⠙⡆⠀⠀⠀⠀⠀⢀⡇⣠⣽⢦⣄⢀⣴⣶⠀⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⡿⣼⣽⡞⠁⠀⠀⠀⢹⡀⠀⠀⠀⢹⠀⠀⠀⠀⠀⠀⣸⠀⠀⠀⠀⠀⣼⠉⠁⠀⠀⢠⢟⣿⣿⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⣷⠉⠁⢳⠀⠀⠀⠀⠈⣧⠀⠀⠀⠀⠙⢦⠀⠀⠀⡠⠁⠀⠀⠀⠀⣰⠃⠀⠀⠀⠀⠏⠀⠀⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠹⡆⠀⠈⡇⠀⠀⠀⠀⠘⣆⠀⠀⠀⠀⠀⠹⣧⠞⠁⠀⠀⠀⠀⣰⠃⠀⠀⠀⠀⠀⠀⠀⣸⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⢳⡀⠀⠙⠀⠀⠀⠀⠀⠘⣆⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⣰⠃⠀⠀⠀⠀⢀⡄⠀⢠⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⢳⡀⣰⣀⣀⣀⠀⠀⠀⠘⣦⣀⠀⠀⠀⡇⠀⠀⠀⢀⡴⠃⠀⠀⠀⠀⠀⢸⡇⢠⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
                                        ⠀⠀⠀⠉⠉⠀⠀⠈⠉⠉⠉⠙⠻⠿⠾⠾⠻⠓⢦⠦⡶⡶⠿⠛⠛⠓⠒⠒⠚⠛⠛⠁
                ,-.----.       ,----..          ,--.                    ____      ,----..            ,--.
                \    /  \     /   /   \     ,--/  /|    ,---,.        ,'  , `.   /   /   \         ,--.'|
                |   :    \   /   .     : ,---,': / '  ,'  .' |     ,-+-,.' _ |  /   .     :    ,--,:  : |
                |   |  .\ : .   /   ;.  \:   : '/ / ,---.'   |  ,-+-. ;   , || .   /   ;.  \,`--.'`|  ' :
                .   :  |: |.   ;   /  ` ;|   '   ,  |   |   .' ,--.'|'   |  ;|.   ;   /  ` ;|   :  :  | |
                |   |   \ :;   |  ; \ ; |'   |  /   :   :  |-,|   |  ,', |  ':;   |  ; \ ; |:   |   \ | :
                |   : .   /|   :  | ; | '|   ;  ;   :   |  ;/||   | /  | |  |||   :  | ; | '|   : '  '; |
                ;   | |`-' .   |  ' ' ' ::   '   \  |   :   .''   | :  | :  |,.   |  ' ' ' :'   ' ;.    ;
                |   | ;    '   ;  \; /  ||   |    ' |   |  |-,;   . |  ; |--' '   ;  \; /  ||   | | \   |
                :   ' |     \   \  ',  / '   : |.  \'   :  ;/||   : |  | ,     \   \  ',  / '   : |  ; .'
                :   : :      ;   :    /  |   | '_\.'|   |    \|   : '  |/       ;   :    /  |   | '`--'
                |   | :       \   \ .'   '   : |    |   :   .';   | |`-'         \   \ .'   '   : |
                `---'.|        `---`     ;   |,'    |   | ,'  |   ;/              `---`     ;   |.'
                  `---`                  '---'      `----'    '---'                         '---'
                                               .--.--.      ,---,         ,----..       ,---,.
                                         ,--, /  /    '.   '  .' \       /   /   \    ,'  .' |
                                       ,'_ /||  :  /`. /  /  ;    '.    |   :     : ,---.'   |
                                  .--. |  | :;  |  |--`  :  :       \   .   |  ;. / |   |   .'
                                ,'_ /| :  . ||  :  ;_    :  |   /\   \  .   ; /--`  :   :  |-,
                                |  ' | |  . . \  \    `. |  :  ' ;.   : ;   | ;  __ :   |  ;/|
                                |  | ' |  | |  `----.   \|  |  ;/  \   \|   : |.' .'|   :   .'
                                :  | | :  ' ;  __ \  \  |'  :  | \  \ ,'.   | '_.' :|   |  |-,
                                |  ; ' |  | ' /  /`--'  /|  |  '  '--'  '   ; : \  |'   :  ;/|
                                :  | : ;  ; |'--'.     / |  :  :        '   | '/  .'|   |    \
                                '  :  `--'   \ `--'---'  |  | ,'        |   :    /  |   :   .'
                                :  ,      .-./           `--''           \   \ .'   |   | ,'
                                 `--`----'                                `---`     `----'
		                                    ____            ____
		  .--.--.                         ,'  , `.        ,'  , `.   ,---,       ,-.----.
		 /  /    '.          ,--,      ,-+-,.' _ |     ,-+-,.' _ |  '  .' \      \    /  \        ,---,
		|  :  /`. /        ,'_ /|   ,-+-. ;   , ||  ,-+-. ;   , || /  ;    '.    ;   :    \      /_ ./|
		;  |  |--`    .--. |  | :  ,--.'|'   |  ;| ,--.'|'   |  ;|:  :       \   |   | .\ :,---, |  ' :
		|  :  ;_    ,'_ /| :  . | |   |  ,', |  ':|   |  ,', |  '::  |   /\   \  .   : |: /___/ \.  : |
		 \  \    `. |  ' | |  . . |   | /  | |  |||   | /  | |  |||  :  ' ;.   : |   |  \ :.  \  \ ,' '
		  `----.   \|  | ' |  | | '   | :  | :  |,'   | :  | :  |,|  |  ;/  \   \|   : .  / \  ;  `  ,'
		  __ \  \  |:  | | :  ' ; ;   . |  ; |--' ;   . |  ; |--' '  :  | \  \ ,';   | |  \  \  \    '
		 /  /`--'  /|  ; ' |  | ' |   : |  | ,    |   : |  | ,    |  |  '  '--'  |   | ;\  \  '  \   |
		'--'.     / :  | : ;  ; | |   : '  |/     |   : '  |/     |  :  :        :   ' | \.'   \  ;  ;
		  `--'---'  '  :  `--'   \;   | |`-'      ;   | |`-'      |  | ,'        :   : :-'      :  \  \
		            :  ,      .-./|   ;/          |   ;/          `--''          |   |.'         \  ' ;
		             `--`----'    '---'           '---'                          `---'            `--`
EOF
}

show_sort_ascii() {
    cat << "EOF"
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⠖⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡤⢤⡀⠀⠀⠀⠀⢸⠀⢱⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠳⡀⠈⠢⡀⠀⠀⢀⠀⠈⡄⠀⠀⠀⠀⠀⠀⠀⠀⡔⠦⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡤⠊⡹⠀⠀⠘⢄⠀⠈⠲⢖⠈⠀⠀⠱⡀⠀⠀⠀⠀⠀⠀⠀⠙⣄⠈⠢⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡠⠖⠁⢠⠞⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⠀⠀⠀⢱⠀⠀⠀⠀⠀⠀⠀⠀⠈⡆⠀⠀⠉⠑⠢⢄⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⡠⠚⠁⠀⠀⠀⡇⠀⠀⠀⠀⠀⢀⠇⠀⡤⡀⠀⠀⠀⢀⣼⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⢠⣾⣿⣷⣶⣤⣄⣉⠑⣄⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⢀⠞⢁⣴⣾⣿⣿⡆⢇⠀⠀⠀⠀⠀⠸⡀⠀⠂⠿⢦⡰⠀⠀⠋⡄⠀⠀⠀⠀⠀⠀⠀⢰⠁⣿⣿⣿⣿⣿⣿⣿⣿⣷⣌⢆⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⡴⢁⣴⣿⣿⣿⣿⣿⣿⡘⡄⠀⠀⠀⠀⠀⠱⣔⠤⡀⠀⠀⠀⠀⠀⠈⡆⠀⠀⠀⠀⠀⠀⡜⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣮⢣⠀⠀⠀⠀⠀
				⠀⠀⠀⡼⢠⣾⣿⣿⣿⣿⣿⣿⣿⣧⡘⢆⠀⠀⠀⠀⠀⢃⠑⢌⣦⠀⠩⠉⠀⡜⠀⠀⠀⠀⠀⠀⢠⠃⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣧⣣⡀⠀⠀⠀
				⠀⠀⢰⢃⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣦⠱⡀⠀⠀⠀⢸⠀⠀⠓⠭⡭⠙⠋⠀⠀⠀⠀⠀⠀⠀⡜⢰⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⡱⡄⠀⠀
				⠀⠀⡏⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣇⢃⠀⠀⠀⢸⠀⠀⠀⠀⢰⠀⠀⠀⠀⠀⠀⠀⢀⠜⢁⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⠘⣆⠀
				⠀⢸⢱⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡘⣆⠀⠀⡆⠀⠀⠀⠀⠘⡄⠀⠀⠀⠀⡠⠖⣡⣾⠁⣸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣧⢸⠀
				⠀⡏⣾⣿⣿⣿⣿⡿⡛⢟⢿⣿⣿⣿⣿⣿⣿⣧⡈⢦⣠⠃⠀⠀⠀⠀⠀⢱⣀⠤⠒⢉⣾⡉⠻⠋⠈⢘⢿⣿⣿⣿⣿⠿⣿⣿⠏⠉⠻⢿⣿⣿⣿⣿⡘⡆
				⢰⡇⣿⣿⠟⠁⢸⣠⠂⡄⣃⠜⣿⣿⠿⠿⣿⣿⡿⠦⡎⠀⠀⠀⠀⠀⠒⠉⠉⠑⣴⣿⣿⣎⠁⠠⠂⠮⢔⣿⡿⠉⠁⠀⠹⡛⢀⣀⡠⠀⠙⢿⣿⣿⡇⡇
				⠘⡇⠏⠀⠀⠀⡾⠤⡀⠑⠒⠈⠣⣀⣀⡀⠤⠋⢀⡜⣀⣠⣤⣀⠀⠀⠀⠀⠀⠀⠙⢿⡟⠉⡃⠈⢀⠴⣿⣿⣀⡀⠀⠀⠀⠈⡈⠊⠀⠀⠀⠀⠙⢿⡇⡇
				⠀⠿⠀⠀⠀⠀⠈⠀⠉⠙⠓⢤⣀⠀⠁⣀⡠⢔⡿⠊⠀⠀⠀⠀⠙⢦⡀⠀⠐⠢⢄⡀⠁⡲⠃⠀⡜⠀⠹⠟⠻⣿⣰⡐⣄⠎⠀⠀⠀⠀⠀⠀⠀⠀⢣⡇
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠉⠁⠀⡜⠀⠀⠀⠀⠀⠀⠀⠀⠱⡀⠀⠀⠀⠙⢦⣀⢀⡴⠁⠀⠀⠀⠀⠉⠁⢱⠈⢆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⢱⠀⠀⠀⠀⠈⢏⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠈⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡠⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⠱⡄⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡜⠀⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⠀⠀⠀⠀⠀⠘⣆⠀⠀⠀⠀⠀⠀⣰⠃⠀⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡾⠀⠀⠘⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠸⠁⠀⠀⠀⠀⠀⠀⠸⡄⠀⠀⠀⢀⡴⠁⠀⠀⢀⠇⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢧⠀⠀⠀⠘⢆⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⣧⣠⠤⠖⠋⠀⠀⠀⠀⡸⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠢⡀⠀⠀⠀⠳⢄⠀⠀⠀⠀⠀⠀⠀⢣⠀⠀⠀⠀⠀⠀⠀⠀⡏⠀⠀⠀⠀⠀⠀⢀⡴⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⡠⠊⠈⠁⠀⠀⠀⡔⠛⠲⣤⣀⣀⣀⠀⠈⢣⡀⠀⠀⠀⠀⠀⢸⠁⠀⠀⠀⢀⡠⢔⠝⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠐⢈⠤⠒⣀⠀⠀⠀⠀⣀⠟⠀⠀⠀⠑⠢⢄⡀⠀⠀⠈⡗⠂⠀⠀⠀⠙⢦⠤⠒⢊⡡⠚⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠆⠒⣒⡁⠬⠦⠒⠉⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠒⢺⢠⠤⡀⢀⠤⡀⠠⠷⡊⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⠣⡀⡱⠧⡀⢰⠓⠤⡁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠁⠀⠈⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				                                            ,----,
				                ,----..                   ,/   .`|
				  .--.--.      /   /   \  ,-.----.      ,`   .'  :
				 /  /    '.   /   .     : \    /  \   ;    ;     /
				|  :  /`. /  .   /   ;.  \;   :    \.'___,/    ,'
				;  |  |--`  .   ;   /  ` ;|   | .\ :|    :     |
				|  :  ;_    ;   |  ; \ ; |.   : |: |;    |.';  ;
				 \  \    `. |   :  | ; | '|   |  \ :`----'  |  |
				  `----.   \.   |  ' ' ' :|   : .  /    '   :  ;
				  __ \  \  |'   ;  \; /  |;   | |  \    |   |  '
				 /  /`--'  / \   \  ',  / |   | ;\  \   '   :  |
				'--'.     /   ;   :    /  :   ' | \.'   ;   |.'
				  `--'---'     \   \ .'   :   : :-'     '---'
				                `---`     |   |.'
				                          `---'
EOF
}

show_search_ascii() {
    cat << "EOF"
			⠀   ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣠⠤⠤⠤⠤⣄⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⠶⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠉⠙⠢⣄⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡴⠋⠁⠀⠀⠀⠀⢀⣀⡐⢄⠀⠀⠀⠀⠀⠀⠈⠳⣄⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡞⠁⠀⠀⠀⠀⠀⡜⠁⠀⣿⡌⠀⠀⠀⠀⠀⠀⠀⠀⠈⢆⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡾⠀⠀⠀⠀⠀⠀⣸⣷⣤⣾⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠊⣼⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⢤⡀⠀⠀⠀⢰⡇⠀⠀⠀⠀⠀⠀⣿⣿⣿⣿⣿⠇⠀⠀⠀⠀⠀⠀⠀⠀⡜⣼⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡎⠀⠉⠲⣄⣀⣼⡇⠀⠀⠀⠀⠀⠀⠻⠿⣿⣟⡼⠀⠀⠀⠀⠀⠀⠀⠀⠀⣿⣿⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠉⠉⠁⠀⡏⠑⠌⠓⢬⣧⠀⠀⠀⠀⠘⢄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢿⠿⡀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣇⠀⠀⠀⠇⠀⠀⠀⠀⠙⣆⠀⠀⠀⠀⠀⠈⠉⠓⠒⠲⠤⢤⣀⠀⠂⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀
			⠀⠀⠀⠀⢀⣠⠤⠖⠒⠒⠒⠦⢤⡀⠀⠀⠀⠀⠀⢸⡄⠀⠀⠀⠀⠀⠀⠀⠀⠈⢦⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠲⠤⠤⠒⠋⢉⠟⠀⠀⠀⠀
			⠀⠀⢀⡴⠋⠁⠀⠀⠀⠀⠀⠀⠀⠙⢦⠀⠀⠀⢠⡞⠹⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⣦⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡠⠋⠀⠀⠀⠀⠀
			⠀⣠⠟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢇⠀⢠⡟⠀⠀⠹⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⡏⠈⠑⠢⢤⣀⣀⠀⠀⠀⠀⢀⣀⡤⠖⠯⣀⠀⠀⠀⠀⠀⠀
			⢀⡟⠀⠀⠀⠀⠠⠴⠤⣀⠀⠀⠀⠀⠀⢸⣠⡟⠀⠀⠀⠀⢹⣄⠀⠀⠀⠀⠀⠀⢀⣼⡁⠀⠀⠀⠀⠀⠈⠉⠉⠉⠉⢻⠀⠀⠀⠀⠀⠉⠢⣄⣀⡀⠀
			⢸⡇⠀⠀⠀⠀⠀⠀⠀⠘⡆⠀⠀⠀⠀⢈⣿⡇⠀⠀⠀⠀⢸⠉⢢⣀⡀⢀⣀⣴⠟⠀⠙⢦⡀⠀⠀⠀⠀⠀⠀⠀⠀⠸⡆⠀⠀⠀⠀⠀⠀⠀⠀⢇⡀
			⠘⣇⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⢸⡇⣷⠀⠀⠀⢀⡞⠀⢰⠏⠉⠉⠁⢸⡀⠀⠀⠀⠈⠓⠶⠤⣤⣄⣀⣠⡤⠴⡇⠀⠀⠀⠀⠀⠀⠀⠀⡔⠁
			⠀⠹⣆⠀⠀⠀⠀⠀⢀⡼⠁⠀⠀⠀⠀⢸⠁⠸⡆⠀⣠⠞⠀⢀⡞⠀⠀⠀⠀⠘⡇⠀⠀⠀⠀⠀⠀⠀⠀⢸⠃⠀⠀⢰⣧⣀⣀⡀⠀⢀⣀⣠⠴⠃⠀
			⠀⠀⠹⡓⠦⠤⠤⠖⠋⠀⠀⠀⠀⠀⠀⢸⠀⠀⠹⡴⠁⠀⢠⠞⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⠀⠀⠀⡞⠀⠀⠀⣸⠀⠀⠉⠉⠉⠉⠀⠀⠀⠀⠀
			⠀⠀⠀⠘⢆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⠀⢸⢁⡠⠴⢧⡀⠀⠀⠀⠀⣀⠔⠳⣄⠀⠀⠀⠀⠀⠀⡼⠁⠀⠀⢠⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠑⢄⠀⠀⠀⠀⠀⠀⠀⠀⠘⣇⣠⡿⠋⠀⠀⠀⠙⢦⣀⡠⠞⠁⠀⠀⠈⠙⠶⣤⣀⡀⣰⠃⠀⠀⣠⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠙⢦⣀⠀⠀⠀⠀⠀⠀⣸⠏⠀⠀⠀⠀⠀⠀⠈⢻⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣹⠋⠉⠉⣹⠏⠙⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠑⠲⢤⣄⣀⣠⡏⠀⠀⠀⠀⠀⠀⠀⠀⠈⣇⠀⠀⠀⠀⠀⠀⠀⠀⡰⠃⢀⣤⠞⠁⠀⠀⠘⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⡽⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣻⠶⠤⠤⠤⠤⠤⢤⣞⡥⠖⠋⠀⠀⠀⠀⠀⠀⢹⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⠟⠒⠀⠀⠒⠒⠺⢯⡁⠀⠀⠀⠀⠀⠀⠀⠀⠀⣾⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⠎⠀⠀⠀⠀⠀⠀⠀⠀⠀⡴⠃⠀⠀⠀⠀⠀⠀⠀⠀⠙⢦⡀⠀⠀⠀⠀⠀⠀⠀⢄⣈⠆⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠳⢄⡀⠀⠀⠀⠀⠀⠀⢀⠞⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⣄⠀⠀⠀⠀⢀⣠⠴⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢇⣀⡤⠖⢄⠀⣰⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠳⠒⠒⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠀⠀⠀⠈⠙⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			                                                                   ,--,
			  .--.--.       ,---,.   ,---,       ,-.----.     ,----..        ,--.'|
			 /  /    '.   ,'  .' |  '  .' \      \    /  \   /   /   \    ,--,  | :
			|  :  /`. / ,---.'   | /  ;    '.    ;   :    \ |   :     :,---.'|  : '
			;  |  |--`  |   |   .':  :       \   |   | .\ : .   |  ;. /|   | : _' |
			|  :  ;_    :   :  |-,:  |   /\   \  .   : |: | .   ; /--` :   : |.'  |
			 \  \    `. :   |  ;/||  :  ' ;.   : |   |  \ : ;   | ;    |   ' '  ; :
			  `----.   \|   :   .'|  |  ;/  \   \|   : .  / |   : |    '   |  .'. |
			  __ \  \  ||   |  |-,'  :  | \  \ ,';   | |  \ .   | '___ |   | :  | '
			 /  /`--'  /'   :  ;/||  |  '  '--'  |   | ;\  \'   ; : .'|'   : |  : ;
			'--'.     / |   |    \|  :  :        :   ' | \.''   | '/  :|   | '  ,/
			  `--'---'  |   :   .'|  | ,'        :   : :-'  |   :    / ;   : ;--'
			            |   | ,'  `--''          |   |.'     \   \ .'  |   ,/
			            `----'                   `---'        `---`    '---'
EOF
}

show_filter_ascii() {
    cat << "EOF"
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⡴⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⣾⡄
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⣾⣷⣾⣿⣃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⢹⣧
			⠀⠀⠀⠀⠀⠀⠀⣀⣤⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⣿⠋⠀⢸⣿
			⠀⠀⠀⢀⣠⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣧⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⡿⠁⠀⠀⢸⣿
			⠀⢀⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣶⣤⡀⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣿⡿⠁⠀⠀⠀⣿⣿
			⢀⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣍⢤⣬⣥⣾⣿⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⣿⣿⠁⠀⠀⠀⢰⣿⡏
			⣸⣿⣿⣿⣿⣿⣿⡟⠛⠻⠿⠿⠿⠿⣿⡿⠇⢨⣍⣙⣛⣠⣿⣿⣿⣿⣿⣿⣿⣷⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣾⣿⡏⠀⠀⠀⢀⣾⣿⠁
			⣿⣿⣿⣿⣿⣿⣿⣧⡘⢶⣶⣶⣶⣶⣶⣤⣤⣤⣤⣭⣉⣉⣉⣛⠛⠿⠿⣿⣿⣿⣿⣷⣄⠀⠀⠀⠀⢀⣠⣴⣦⣤⣶⣶⣿⣿⣧⣤⣤⣄⠉⢻⡇⠀⠀⠀⣼⣿⡏⠀
			⣿⣿⣿⣿⣿⣿⠿⣿⣿⣄⠙⢿⣿⣄⠀⠀⠀⠀⠈⠉⠙⠛⠻⠿⣿⣿⣶⣤⡉⠻⣿⣿⣿⡆⠀⢀⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⣥⣤⣭⣁⣀⠀⣸⣿⡟⠀⠀
			⢻⣿⣿⣿⣿⣿⡄⢬⣙⠻⢰⣤⡙⠻⣷⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⠻⣿⣷⣌⠻⢿⡇⢠⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣯⣍⡁⠿⠋⠀⠀⠀
			⠸⣿⡏⣉⡻⢿⣿⡜⣿⣿⣿⣿⣿⣶⣄⡙⠻⣷⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⣷⣄⢁⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣛⣛⣛⠛⠂⠀⠀⠀⠀
			⠀⢻⡇⣿⣿⣶⣬⣙⠈⢿⣿⣿⣿⣿⣿⣿⣶⣌⡙⠻⢿⣶⣤⣄⡀⠀⠀⠀⠀⠀⠀⠈⢻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⡀⠀⠀⠀⠀
			⠀⠈⠁⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣤⣈⠙⠻⢿⣿⣶⣦⣄⣀⣀⣤⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⠀⠀
			⠀⠀⠀⠈⠻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠗⢀⣀⣉⠙⠛⠛⠛⠿⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡟⣉⠻⣿⣿⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠙⠻⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠟⢉⣠⣾⣿⣿⠟⣁⣤⣴⣶⣶⢸⣿⣿⣿⣿⣿⣿⣿⢟⣛⠻⣿⣿⣿⣿⣿⣿⣿⣿⠁⣿⠇⠸⣿⡀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠛⠿⢿⣿⣿⣿⣿⡿⠃⣴⣿⣿⣿⠟⣡⣾⣿⣿⣿⣿⣿⢸⣿⣿⣿⣿⣿⣿⢣⣿⣿⠀⠘⣿⣿⣿⣿⣿⣿⣿⠀⠀⠀⠀⣿⡇⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠀⢀⣾⣿⣿⣿⠏⣼⡿⢟⣿⣿⣿⣿⣿⠀⣿⣿⣿⣿⣿⣿⠘⠛⠋⠀⠀⢹⣿⣿⣿⣿⣿⣿⣇⠠⠤⢃⣿⡇⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⣿⣿⣿⣿⣀⡅⣠⣿⣿⣿⣿⣿⣿⣇⢹⣿⣿⣿⣿⣿⠰⣷⣀⣠⡄⣸⣿⣿⣿⣿⣿⠿⢻⣷⣶⣿⣿⡇⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣿⣿⣿⣿⣿⣿⢡⣿⣿⣿⣿⣿⣿⣿⣿⡄⢻⣿⣿⣿⣿⣷⣬⣉⣉⣴⣿⣿⣿⣿⣿⣿⣿⠿⢟⣿⣿⠟⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣿⣿⣿⣿⣿⣿⡇⢸⡇⣿⣿⣿⣿⣿⡿⢹⣿⣦⡙⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣛⣋⣭⣶⣾⡿⢟⣡⣾⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢻⣿⣿⣿⣿⣿⣇⢈⣥⢸⣿⣿⣿⣿⡇⣾⣿⣿⣿⣶⣬⣙⠻⠿⣿⣿⣿⣿⣿⣿⠿⠿⢛⣉⣥⣶⣿⡿⠟⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⣿⣿⣿⡿⠈⢿⣿⣿⣿⡀⣿⣿⣿⣿⣿⣿⣿⣿⣶⣦⣤⣤⣤⣤⣴⣶⣾⣿⣿⢿⣿⣿⠁⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⣿⣿⣿⣿⣿⣿⣿⠡⣿⣌⢻⣿⣿⣧⢹⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢸⣿⠃⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣿⣿⣿⣿⣿⣿⣿⠇⠀⢿⣿⣷⡙⠛⣿⡇⠹⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⠛⠁⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢹⣿⣿⣿⣿⣿⣿⠀⠀⠈⢿⣿⣿⣶⠈⣰⣷⣬⡻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠛⠟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣿⣿⣿⣿⣿⠀⠀⠀⠀⠈⠉⠋⢠⣿⣿⣿⣿⣮⡛⢿⠏⠻⢿⣿⣿⣿⣿⢟⣴⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⢿⠆⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⣿⣿⡆⠀⠀⠀⢉⡛⠟⣱⣿⣿⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠁⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢻⣿⣿⣿⣿⣿⠀⠀⠀⠀⠈⣿⣾⣿⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣿⣿⣿⣿⣿⡄⠀⠀⠀⠀⢹⣿⣿⣿⣿⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⣿⣿⣿⣿⡄⠀⠀⠀⠀⠉⠛⠋⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
			⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠿⣿⡟⢿⠟⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
				                     ,--,           ,----,
				                  ,---.'|         ,/   .`|
				    ,---,.   ,---,|   | :       ,`   .'  :   ,---,.,-.----.
				  ,'  .' |,`--.' |:   : |     ;    ;     / ,'  .' |\    /  \
				,---.'   ||   :  :|   ' :   .'___,/    ,',---.'   |;   :    \
				|   |   .':   |  ';   ; '   |    :     | |   |   .'|   | .\ :
				:   :  :  |   :  |'   | |__ ;    |.';  ; :   :  |-,.   : |: |
				:   |  |-,'   '  ;|   | :.'|`----'  |  | :   |  ;/||   |  \ :
				|   :  ;/||   |  |'   :    ;    '   :  ; |   :   .'|   : .  /
				|   |   .''   :  ;|   |  ./     |   |  ' |   |  |-,;   | |  \
				'   :  '  |   |  ';   : ;       '   :  | '   :  ;/||   | ;\  \
				|   |  |  '   :  ||   ,/        ;   |.'  |   |    \:   ' | \.'
				|   :  \  ;   |.' '---'         '---'    |   :   .':   : :-'
				|   | ,'  '---'                          |   | ,'  |   |.'
				`----'                                   `----'    `---'
EOF
}

if [[ ! -f "$1" ]]; then
    echo "Error: File not Found!"
    exit 1
fi

FILE="$1"
COMMAND="$2"
OPTION="$3"
```
- `#!/bin/bash`: Shebang line yang menunjukkan bahwa skrip ini akan dijalankan menggunakan Bash shell
<br>

- `LINE=$(printf '=%.0s' $(seq 1 $(tput cols)))`:
    - `tput cols` → Mengambil jumlah kolom terminal saat ini
    - `seq 1 $(tput cols)` → Membuat deret angka dari 1 hingga jumlah kolom terminal
    - `printf '=%.0s' $(seq 1 $(tput cols))` → Mencetak karakter '=' sebanyak jumlah kolom terminal tanpa output tambahan
    - Variabel LINE akan berisi garis horizontal '=' sepanjang terminal
<br>

- `print_centered()` → Mendefinisikan fungsi untuk mencetak teks di tengah terminal
    - `local termwidth=$(tput cols)` → Mengambil jumlah kolom terminal
    - `local padding=$(((termwidth - ${#1}) / 2))` → Menghitung jumlah spasi di kiri dan kanan agar teks berada di tengah
    - `printf "%*s%s%*s\n" $padding "" "$1" $padding ""` → Mencetak teks dengan spasi di kiri dan kanan agar berada di tengah
<br>

- `show_help_ascii()` → Fungsi untuk menampilkan teks ASCII (help)
- `show_info_ascii()` → Fungsi untuk menampilkan teks ASCII (info)
- `show_sort_ascii()` → Fungsi untuk menampilkan teks ASCII (sort)
- `show_search_ascii()` → Fungsi untuk menampilkan teks ASCII (search)
- `show_filter_ascii()` → Fungsi untuk menampilkan teks ASCII (filter)
- `cat << "EOF" ... EOF` → Tempat menulis teks ASCII
<br>

- `if [[ ! -f "$1" ]]; then` → Jika kolom pertama($1)/file tidak ada
- `echo "Error: File not Found!"` → Tampilkan error
- `exit 1` → Menghentikan script
- `fi` → Menutup if
<br>

- Contoh: `./script.sh pokemon_usage.csv --info dark`
- `FILE="$1"` → Mengisi variabel `FILE` dengan argumen pertama (`pokemon_usage.csv`)
- `COMMAND="$2"` → Mengisi variabel `COMMAND` dengan argumen kedua (`--info`)
- `OPTION="$3"` → Mengisi variable `OPTION` dengan argumen ketiga (`dark`)
<br>

A. Melihat summary dari data → Untuk mengetahui Pokemon apa yang sedang membawa teror kepada lingkungan “*8eneration 9 OverUsed*” anda berusaha untuk membuat sebuah fitur untuk menampilkan nama Pokemon dengan Usage% dan RawUsage paling tinggi.

```bash
case "$COMMAND" in
    -i|--info)
        show_info_ascii
        USAGE_POKEMON=$(awk -F',' 'NR>1 { if ($2 > maxUsage) { maxUsage = $2; maxPokemon = $1 } } END { print maxPokemon }' "$FILE")
        USAGE_PERCENT=$(awk -F',' 'NR>1 { if ($2 > maxUsage) { maxUsage = $2 } } END { printf "%.5f", maxUsage }' "$FILE")
        RAW_POKEMON=$(awk -F',' 'NR>1 { if ($3 > maxRaw) { maxRaw = $3; maxRawPokemon = $1 } } END { print maxRawPokemon }' "$FILE")
        RAW_USAGE=$(awk -F',' 'NR>1 { if ($3 > maxRaw) { maxRaw = $3 } } END { print maxRaw }' "$FILE")

        print_centered "$LINE"
        print_centered "📜 USAGE SUMMARY OF $FILE"
        print_centered "$LINE"
        print_centered "🔹 Highest Adjusted Usage 🔹"
        print_centered "🦁 $USAGE_POKEMON with $USAGE_PERCENT%"
        print_centered ""
        print_centered "🔸 Highest Raw Usage 🔸"
        print_centered "🐯 $RAW_POKEMON with $RAW_USAGE uses"
        print_centered "$LINE"
        exit 0
        ;;
```
- `case "$COMMAND" in` → Memulai struktur case untuk menangani berbagai perintah yang diberikan sebagai argumen kedua (COMMAND)
- `i|--info)` → Jika argumen kedua adalah `-i` atau `--info`, maka perintah di dalam blok ini akan dijalankan
	- `show_info_ascii` → Memanggil fungsi untuk menampilkan teks ASCII terkait informasi
 	- `USAGE_POKEMON=$(awk -F',' 'NR>1 { if ($2 > maxUsage) { maxUsage = $2; maxPokemon = $1 } } END { print maxPokemon }' "$FILE")` → Menggunakan awk untuk mencari Pokémon dengan Usage% tertinggi pada dataset
		- `-F','` → Menentukan pemisah kolom sebagai koma (CSV)
		- `NR>1` → Melewati baris pertama (header)
		- `if ($2 > maxUsage) { maxUsage = $2; maxPokemon = $1 }` → Menyimpan Pokémon dengan nilai Usage% tertinggi
		- `END { print maxPokemon }` → Mencetak nama Pokémon dengan Usage% tertinggi
	- `USAGE_PERCENT=$(awk -F',' 'NR>1 { if ($2 > maxUsage) { maxUsage = $2 } } END { printf "%.5f", maxUsage }' "$FILE")` → Menggunakan awk untuk mencari persentase Usage% tertinggi dalam dataset (5 desimal)
	- `RAW_POKEMON=$(awk -F',' 'NR>1 { if ($3 > maxRaw) { maxRaw = $3; maxRawPokemon = $1 } } END { print maxRawPokemon }' "$FILE")` → Menggunakan awk untuk mencari Pokémon dengan RawUsage tertinggi pada dataset
	- `RAW_USAGE=$(awk -F',' 'NR>1 { if ($3 > maxRaw) { maxRaw = $3 } } END { print maxRaw }' "$FILE")` → Menggunakan awk untuk mencari nilai RawUsage tertinggi dalam dataset
- `print_centered "$LINE"` → Mencetak garis pemisah secara terpusat
- `print_centered "📜 USAGE SUMMARY OF $FILE"` → Menampilkan judul laporan
- `print_centered "$LINE"` → Mencetak garis pemisah
- `print_centered "🔹 Highest Adjusted Usage 🔹"` → Menampilkan subjudul
- `print_centered "🦁 $USAGE_POKEMON with $USAGE_PERCENT%"` → Menampilkan Pokémon dengan Usage% tertinggi
- `print_centered ""` → Mencetak baris kosong untuk pemisahan visual
- `print_centered "🔸 Highest Raw Usage 🔸"` → Menampilkan subjudul
- `print_centered "🐯 $RAW_POKEMON with $RAW_USAGE uses"` → Menampilkan Pokémon dengan RawUsage tertinggi
- `print_centered "$LINE"` → Mencetak garis pemisah
- `exit 0` → Menghentikan skrip dengan kode sukses
<br>

B. Mengurutkan Pokemon berdasarkan data kolom → Untuk memastikan bahwa anda mengetahui kondisi lingkungan “*8eneration 9 OverUsed*”, anda berusaha untuk membuat sebuah fitur untuk sort berdasarkan:
- Usage%
- RawUsage
- Nama
- HP
- Atk
- Def
- Sp.Atk
- Sp.Def
- Speed
 
Sort dilakukan dengan urutan descending untuk semua angka selain nama, yang diurutkan secara alphabetical. Output harus sesuai dengan format csv yang diberikan.

```bash
    -s|--sort)
    	show_sort_ascii
    	if [[ -z "$OPTION" ]]; then
        	echo "Error: No sorting option provided"
        	exit 1
    	fi
    	print_centered "$LINE"
    	print_centered "📜 SORTED POKEMON LIST BASED ON $OPTION"
    	print_centered "$LINE"

    	awk -F',' -v col="$OPTION" '
    	NR==1 {
        	for (i=1; i<=NF; i++) if (tolower($i) == tolower(col)) colnum = i;
        	if (!colnum) { print "Error: Column not found"; exit 1 }
        	print;
        	next;
    	}
    	{ print | "sort -t, -k" colnum "nr" }' "$FILE" | column -s ',' -t

    	print_centered "$LINE"
    	exit 0
    	;;
```
- `-s|--sort)` → Jika argumen kedua adalah `-s` atau `--sort`, maka blok perintah ini akan dijalankan
- `show_sort_ascii` → Memanggil fungsi untuk menampilkan teks ASCII terkait sorting
- `if [[ -z "$OPTION" ]]; then` → Mengecek apakah argumen ketiga `OPTION` kosong
	- `echo "Error: No sorting option provided"` → Jika tidak ada opsi sorting yang diberikan, tampilkan pesan error		
 	- `exit 1` → Hentikan skrip dengan kode error
- `print_centered "$LINE"` → Mencetak garis pemisah secara terpusat
- `print_centered "📜 SORTED POKEMON LIST BASED ON $OPTION"` → Menampilkan judul laporan berdasarkan opsi sorting yang diberikan
- `print_centered "$LINE"` → Mencetak garis pemisah
- `awk -F',' -v col="$OPTION" '` → Menggunakan awk untuk mencari kolom yang sesuai dengan OPTION dan mengurutkan datanya
	- `NR==1 {` → Jika sedang memproses baris pertama (header)
		- `for (i=1; i<=NF; i++) if (tolower($i) == tolower(col)) colnum = i;` → Loop untuk mencari indeks kolom yang sesuai dengan OPTION (case-insensitive)
		- `if (!colnum) { print "Error: Column not found"; exit 1 }` → Jika tidak ditemukan, tampilkan error dan hentikan skrip
		- `print;` → Cetak header
		- `next;` → Lompat ke baris berikutnya
	- `}` → Jika bukan header, cetak setiap baris dan teruskan ke perintah sort
- `sort -t, -k" colnum "nr"` → Mengurutkan data berdasarkan kolom yang ditemukan (colnum), menggunakan koma sebagai pemisah, dan sorting numerik (nr)
- `| column -s ',' -t` → Memformat output agar lebih rapi dengan pemisah kolom yang sesuai
- `print_centered "$LINE"` → Mencetak garis pemisah
- `exit 0` → Menghentikan skrip
<br>

C. Mencari nama Pokemon tertentu → Setelah mengetahui kondisi lingkungan “*Generation 9 OverUsed*”, anda ingin mencari tahu statistik penggunaan dari beberapa Pokemon yang mungkin dapat bertanding baik melawan sebagian besar Pokemon yang ada di Top 10 usage. Oleh karena itu, anda membuat fitur search berdasarkan nama Pokemon. Pastikan agar search yang dimasukkan tidak memunculkan hasil yang tidak diinginkan (seperti memunculkan semua Grass type ketika mengetik search “Grass”), dan output harus sesuai dengan format csv yang diberikan dengan sort Usage%.

```bash
    -sh|--search)
    	show_search_ascii
    	if [[ -z "$OPTION" ]]; then
        	echo "Error: No search term provided"
        	exit 1
    	fi
    	print_centered "$LINE"
    	print_centered "🔍 SEARCH RESULTS FOR: $OPTION"
    	print_centered "$LINE"

    	awk -F',' -v term="$OPTION" 'NR==1 || tolower($1) ~ tolower(term)' "$FILE" | sort -t',' -k2,2nr | column -s ',' -t

    	print_centered "$LINE"
    	exit 0
    	;;
```
- `-sh|--search)` → Jika argumen kedua adalah `-sh` atau `--search`, maka blok perintah ini akan dijalankan
	- `show_search_ascii` → Memanggil fungsi untuk menampilkan teks ASCII terkait pencarian
 	- `if [[ -z "$OPTION" ]]; then` → Mengecek apakah argumen ketiga `OPTION` kosong
		- `echo "Error: No search term provided"` → Jika tidak ada istilah pencarian, tampilkan pesan error
		- `exit 1` → Hentikan skrip dengan kode error
	- `print_centered "$LINE"` → Mencetak garis pemisah secara terpusat
	- `print_centered "🔍 SEARCH RESULTS FOR: $OPTION"` → Menampilkan judul hasil pencarian berdasarkan istilah yang diberikan
	- `print_centered "$LINE"` → Mencetak garis pemisah
 	- `awk -F',' -v term="$OPTION" 'NR==1 || tolower($1) ~ tolower(term)' "$FILE"` → Menggunakan awk untuk mencari Pokémon yang cocok dengan istilah pencarian
		- `-F','` → Menentukan koma sebagai pemisah kolom (CSV)
		- `-v term="$OPTION"` → Menyimpan istilah pencarian ke dalam variabel term
		- `NR==1` → Selalu mencetak baris pertama (header)
		- `tolower($1) ~ tolower(term)` → Mencari istilah dalam kolom pertama (nama Pokémon) secara case-insensitive
	- `| sort -t',' -k2,2nr` → Mengurutkan hasil pencarian berdasarkan olom kedua (Usage%) secara numerik dari terbesar ke terkecil
	- `| column -s ',' -t` → Memformat output agar lebih rapi dengan pemisah kolom yang sesuai
	- `print_centered "$LINE"` → Mencetak garis pemisah
	- `exit 0` → Menghentikan skrip
 <br>

D. Mencari Pokemon berdasarkan filter nama type → Agar dapat membuat tim yang baik, anda perlu memikirkan kombinasi yang baik dari beberapa Pokemon, hal ini disebut sebagai “core” oleh komunitas Pokemon! Oleh karena itu, anda berpikiran untuk membuat fitur filter berdasarkan Type sebuah Pokemon. Output harus sesuai dengan format csv yang diberikan dengan sort Usage%

```bash
    -f|--filter)
        show_filter_ascii
        if [[ -z "$OPTION" ]]; then
            echo "Error: No filter option provided"
            exit 1
        fi
        print_centered "$LINE"
        print_centered "🌿 FILTERED POKEMON LIST FOR TYPE: $OPTION"
        print_centered "$LINE"
        awk -F',' -v type="$OPTION" 'NR==1 || tolower($4) == tolower(type) || tolower($5) == tolower(type)' "$FILE" |sort -t',' -k2,2nr | column -s ',' -t
        print_centered "$LINE"
        exit 0
        ;;
```
- `-f|--filter)` → Jika argumen kedua adalah `-f` atau `--filter`, maka blok perintah ini akan dijalankan
	- `show_filter_ascii` → Memanggil fungsi untuk menampilkan teks ASCII terkait filter
	- `if [[ -z "$OPTION" ]]; then` → Mengecek apakah argumen ketiga `OPTION` kosong
		- `echo "Error: No filter option provided"` → Jika tidak ada opsi filter yang diberikan, tampilkan pesan error
		- `exit 1` → Hentikan skrip dengan kode error
	- `print_centered "$LINE"` → Mencetak garis pemisah secara terpusat
	- `print_centered "🌿 FILTERED POKEMON LIST FOR TYPE: $OPTION"` → Menampilkan judul daftar Pokémon yang difilter berdasarkan tipe tertentu
	- `print_centered "$LINE"` → Mencetak garis pemisah
	- `awk -F',' -v type="$OPTION" 'NR==1 || tolower($4) == tolower(type) || tolower($5) == tolower(type)' "$FILE"` → Menggunakan awk untuk memfilter Pokémon berdasarkan tipe yang diberikan
		- `-F','` → Menentukan koma sebagai pemisah kolom (CSV)
		- `-v type="$OPTION"` → Menyimpan tipe Pokémon yang ingin difilter ke dalam variabel type
		- `NR==1` → Selalu mencetak baris pertama (header)
		- `tolower($4) == tolower(type) || tolower($5) == tolower(type)` → Mencocokkan tipe Pokémon pada kolom ke-4 atau ke-5 (karena beberapa Pokémon memiliki dua tipe), dengan pencocokan case-insensitive
	- `| sort -t',' -k2,2nr` → Mengurutkan hasil filter berdasarkan kolom kedua (Usage%) secara numerik dari terbesar ke terkecil
	- `| column -s ',' -t` → Memformat output agar lebih rapi dengan pemisah kolom yang sesuai
	- `print_centered "$LINE" → Mencetak garis pemisah`
	- `exit 0` → Menghentikan skrip dengan kode sukses
   <br>

E. Error handling → Pastikan program yang anda buat mengecek semua kesalahan pengguna agar dapat memberikan kejelasan kepada pengguna pada setiap kasus.

```bash
if [[ ! -f "$1" ]]; then
    echo "Error: File not Found!"
    exit 1
fi
```
- Mengatasi error jika file tidak ditemukan

```bash
if [[ -z "$OPTION" ]]; then
	echo "Error: No sorting option provided"
	exit 1
fi
```
- Mengatasi error jika argumen kedua tidak ditemukan

```bash
    *)
        show_info_ascii
        print_centered "$LINE"
        print_centered "❌ ERROR: Invalid Command"
        print_centered "$LINE"
        exit 1
        ;;
```
- Mengatasi error jika argumen ketiga tidak ditemukan
<br>

F. Help screen yang menarik → Untuk memberikan petunjuk yang baik pada pengguna program, anda berpikir untuk membuat sebuah help screen yang muncul ketika mengetik -h atau --help sebagai command yang dijalankan. Kriteria yang harus ada dalam help screen pada program ini adalah:
- ASCII Art yang menarik! Gunakan kreativitas anda untuk mencari/membuat art yang cocok untuk program yang sudah anda buat!
- Penjelasan setiap command dan sub-command

```bash
    -h|--help)
        show_help_ascii
        print_centered "$LINE"
        print_centered "📜 HELP SCREEN"
        print_centered "$LINE"
        cat << EOF
	Usage: ./pokemon_analysis.sh <file_name> [options]

	Options:
	  -h, --help            Display this help message.
	  -i, --info            Display the highest adjusted and raw usage.
	  -s, --sort <method>   Sort the data by the specified column.
	                        name    Sort by Pokemon name.
	                        usage   Sort by Adjusted Usage.
	                        raw     Sort by Raw Usage.
	                        hp      Sort by HP.
	                        atk     Sort by Attack.
	                        def     Sort by Defense.
	                        spatk   Sort by Special Attack.
	                        spdef   Sort by Special Defense.
	                        speed   Sort by Speed.
	  -sh, --search <name>     Search for a specific Pokemon sorted by usage.
	  -f, --filter <type>   Filter by type of Pokemon sorted by usage.
EOF
        print_centered "$LINE"
        exit 0
        ;;
```
- `-h|--help)` → Jika argumen kedua adalah `-h` atau `--help`, maka blok perintah ini akan dijalankan
	- `show_help_ascii` → Memanggil fungsi untuk menampilkan teks ASCII terkait bantuan
	- `print_centered "$LINE"` → Mencetak garis pemisah secara terpusat
	- `print_centered "📜 HELP SCREEN"` → Menampilkan judul "HELP SCREEN"
	- `print_centered "$LINE"` → Mencetak garis pemisah
	- `cat << EOF ... EOF` → Menampilkan blok teks bantuan kepada pengguna
		- `Usage: ./pokemon_analysis.sh <file_name> [options]` → Menjelaskan cara menggunakan skrip
		- `Options:` → Daftar opsi yang tersedia dalam skrip
		- `-h, --help` → Menampilkan pesan bantuan
		- `-i, --info` → Menampilkan Pokémon dengan Adjusted Usage dan Raw Usage tertinggi
		- `-s, --sort <method>` → Mengurutkan data berdasarkan kolom tertentu
			- `name` → Mengurutkan berdasarkan nama Pokémon
			- `usage` → Mengurutkan berdasarkan Adjusted Usage
			- `raw` → Mengurutkan berdasarkan Raw Usage
			- `hp` → Mengurutkan berdasarkan HP
			- `atk` → Mengurutkan berdasarkan Attack
			- `def` → Mengurutkan berdasarkan Defense
			- `spatk` → Mengurutkan berdasarkan Special Attack
			- `spdef` → Mengurutkan berdasarkan Special Defense
			- `speed` → Mengurutkan berdasarkan Speed
		- `-sh, --search <name>` → Mencari Pokémon berdasarkan nama dan mengurutkan hasilnya berdasarkan Usage
		- `-f, --filter <type>` → Memfilter Pokémon berdasarkan tipe dan mengurutkan hasilnya berdasarkan Usage
	- `print_centered "$LINE"` → Mencetak garis pemisah
	- `exit 0` → Menghentikan skrip
