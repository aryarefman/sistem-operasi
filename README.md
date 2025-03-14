# Sisop-01-2025-IT08

Anggota IT08:
| Nama | NRP |
| ---------------------- | ---------- |
| Arya Bisma Putra Refman | 5027241036 |
| Moch. Rizki Nasrullah | 5027241038 |
| Nadia Fauziazahra Kusumastuti | 5027241094 |
----

## Soal 4
[Author: Amoes / winter]

Pada suatu hari, anda diminta teman anda untuk membantunya mempersiapkan diri untuk turnamen Pokemon “*Generation 9 OverUsed 6v6 Singles*” dengan cara membuatkan tim yang cocok untuknya. Tetapi, anda tidak memahami meta yang dimainkan di turnamen tersebut. Untungnya, seorang informan memberikan anda data pokemon_usage.csv yang bisa anda download dan analisis. 

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
nano pokemon_analysis.sh
```
- `nano`: Editor teks berbasis terminal di Linux
- `pokemon_analysis.sh`: Nama file skrip shell yang akan dibuat dan diedit menggunakan `nano`
- Perintah ini membuka editor `nano` untuk membuat atau mengedit skrip bernama `pokemon_analysis.sh`
