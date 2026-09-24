# Memory Forensics A--Z --- Rekap Pembelajaran Hari Ini

## 1. `windows.info`

**Penjelasan:**\
Digunakan untuk mengetahui informasi dasar sistem operasi dari memory
dump.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.info
```

Spesifik:

``` bash
vol -f MemoryDump.mem windows.info | grep "Kernel"
```

`grep` digunakan untuk mengambil output yang mengandung kata tertentu.

  Column           Penjelasan
  ---------------- -------------------------------
  Kernel Base      Alamat dasar kernel Windows
  DTB              Directory Table Base
  Is64Bit          Menunjukkan arsitektur 64-bit
  MachineType      Tipe arsitektur mesin
  SystemTime       Waktu sistem pada memory dump
  NtSystemRoot     Lokasi instalasi Windows
  NtMajorVersion   Versi mayor Windows
  NtMinorVersion   Versi minor Windows

------------------------------------------------------------------------

## 2. `windows.pslist`

**Penjelasan:**\
Digunakan untuk melihat daftar proses yang aktif/terdaftar di memory.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.pslist
```

Spesifik:

``` bash
vol -f MemoryDump.mem windows.pslist | grep "oneetx.exe"
```

Mencari proses tertentu berdasarkan nama.

  Column          Penjelasan
  --------------- -------------------------------------
  PID             ID unik proses
  PPID            PID dari parent process
  ImageFileName   Nama executable proses
  Threads         Jumlah thread
  Handles         Jumlah handle
  SessionId       ID session
  CreateTime      Waktu proses dibuat
  ExitTime        Waktu proses berhenti jika tersedia

------------------------------------------------------------------------

## 3. `windows.pstree`

**Penjelasan:**\
Digunakan untuk melihat hubungan parent-child antarproses.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.pstree
```

Spesifik:

``` bash
vol -f MemoryDump.mem windows.pstree | grep -A 5 "oneetx.exe"
```

`-A 5` menampilkan 5 baris setelah baris yang cocok.

  Column    Penjelasan
  --------- --------------------------------
  PID       ID proses
  PPID      ID parent process
  Process   Nama proses
  Tree      Struktur hubungan parent-child

------------------------------------------------------------------------

## 4. `windows.psscan`

**Penjelasan:**\
Digunakan untuk melakukan scanning terhadap memory untuk menemukan
process object. Berguna untuk menemukan proses yang tidak terlihat pada
daftar proses biasa, termasuk artefak proses yang sudah terminated atau
tidak lagi terhubung secara normal.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.psscan
```

Spesifik:

``` bash
vol -f MemoryDump.mem windows.psscan | grep "oneetx.exe"
```

Mencari proses tertentu.

  Column          Penjelasan
  --------------- --------------------------------------------------
  PID             ID proses
  PPID            ID parent process
  ImageFileName   Nama executable
  Offset(V)       Alamat virtual object proses
  Threads         Jumlah thread
  Handles         Jumlah handle
  Wow64           Menunjukkan apakah proses berjalan sebagai WOW64
  CreateTime      Waktu proses dibuat
  ExitTime        Waktu proses berhenti

**Catatan:**\
`Wow64=True` bukan berarti malware. Itu menunjukkan proses 32-bit
berjalan pada Windows 64-bit.

------------------------------------------------------------------------

## 5. PID dan PPID

**Penjelasan:**\
PID adalah identitas sebuah proses, sedangkan PPID menunjukkan parent
dari proses tersebut.

**Penerapan perintah:**

Cari PID:

``` bash
vol -f MemoryDump.mem windows.pslist | grep "oneetx.exe"
```

Cari proses berdasarkan PID:

``` bash
vol -f MemoryDump.mem windows.pslist | grep -E "\s5896\s"
```

Cari child dari PID tertentu:

``` bash
vol -f MemoryDump.mem windows.pslist | grep -E "\s5896$"
```

Secara konsep, jika:

``` text
PID 5896 = oneetx.exe
PPID 8844
```

maka `8844` adalah parent dari `5896`.

  Column   Penjelasan
  -------- --------------------------
  PID      Identitas proses
  PPID     Identitas parent process

------------------------------------------------------------------------

## 6. `windows.cmdline`

**Penjelasan:**\
Digunakan untuk melihat command line yang digunakan ketika proses
dijalankan.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.cmdline
```

Spesifik berdasarkan PID:

``` bash
vol -f MemoryDump.mem windows.cmdline --pid 5896
```

Spesifik berdasarkan nama:

``` bash
vol -f MemoryDump.mem windows.cmdline | grep "oneetx.exe"
```

  Column    Penjelasan
  --------- ---------------------------------------------
  PID       ID proses
  Process   Nama proses
  Args      Command line/argument yang digunakan proses

------------------------------------------------------------------------

## 7. `windows.dlllist`

**Penjelasan:**\
Digunakan untuk melihat DLL yang dimuat oleh suatu proses.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.dlllist
```

Spesifik PID:

``` bash
vol -f MemoryDump.mem windows.dlllist --pid 5896
```

Cari DLL tertentu:

``` bash
vol -f MemoryDump.mem windows.dlllist --pid 5896 | grep -Ei "\.dll"
```

  Column    Penjelasan
  --------- -----------------
  PID       ID proses
  Process   Nama proses
  Base      Alamat awal DLL
  Size      Ukuran DLL
  Name      Nama/path DLL

------------------------------------------------------------------------

## 8. `windows.vadinfo`

**Penjelasan:**\
Digunakan untuk melihat Virtual Address Descriptor (VAD), yaitu
informasi mengenai region memory virtual milik suatu proses.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.vadinfo
```

Spesifik PID:

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid 5896
```

Cari permission memory:

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid 5896 | grep "PAGE_EXECUTE"
```

Pada kasus RedLine, region:

``` text
PAGE_EXECUTE_READWRITE
PrivateMemory = 1
File = N/A
```

menjadi region yang perlu diperhatikan karena memory tersebut writable
sekaligus executable dan tidak memiliki backing file.

  Column          Penjelasan
  --------------- -------------------------------------------
  Start VPN       Alamat awal region virtual memory
  End VPN         Alamat akhir region
  Tag             Tag VAD
  Protection      Permission memory
  CommitCharge    Memory yang sudah di-commit
  PrivateMemory   Menunjukkan private memory
  File            File yang menjadi backing memory jika ada

------------------------------------------------------------------------

## 9. Memory Protection

**Penjelasan:**\
Protection menunjukkan permission sebuah region memory.

**Penerapan perintah:**

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid 5896 | grep -E "PAGE_EXECUTE|PAGE_READWRITE"
```

Permission yang perlu dipahami:

``` text
PAGE_READONLY
PAGE_READWRITE
PAGE_EXECUTE_READ
PAGE_EXECUTE_READWRITE
PAGE_EXECUTE_WRITECOPY
```

`PAGE_EXECUTE_READWRITE` berarti region dapat dieksekusi, dibaca, dan
ditulis.

`PAGE_EXECUTE_WRITECOPY` berarti region executable dan menggunakan
mekanisme copy-on-write.

  Protection               Penjelasan
  ------------------------ -------------------------------------------
  PAGE_READONLY            Hanya dapat dibaca
  PAGE_READWRITE           Dapat dibaca dan ditulis
  PAGE_EXECUTE_READ        Dapat dieksekusi dan dibaca
  PAGE_EXECUTE_READWRITE   Dapat dieksekusi, dibaca, dan ditulis
  PAGE_EXECUTE_WRITECOPY   Executable dengan mekanisme copy-on-write

------------------------------------------------------------------------

## 10. `windows.malfind`

**Penjelasan:**\
Digunakan untuk membantu menemukan region memory yang memiliki
karakteristik mencurigakan, misalnya executable private memory.

**Penerapan perintah:**

Versi lama:

``` bash
vol -f MemoryDump.mem windows.malfind
```

Spesifik PID:

``` bash
vol -f MemoryDump.mem windows.malfind --pid 5896
```

Versi plugin baru:

``` bash
vol -f MemoryDump.mem windows.malware.malfind
```

**Catatan:**\
Pada environment yang digunakan hari ini, plugin mengalami error
rendering karena dependency Capstone:

``` text
AttributeError:
module 'capstone' has no attribute 'CS_ARCH_ARM64'
```

Jadi error tersebut merupakan masalah dependency/rendering plugin, bukan
bukti bahwa memory dump rusak.

  Column          Penjelasan
  --------------- ---------------------------------
  PID             ID proses
  Process         Nama proses
  Start VPN       Awal region memory
  End VPN         Akhir region memory
  Protection      Permission region
  CommitCharge    Jumlah memory yang di-commit
  PrivateMemory   Status private memory
  File            File backing memory
  Notes           Informasi indikasi mencurigakan

------------------------------------------------------------------------

## 11. `windows.netscan`

**Penjelasan:**\
Digunakan untuk menemukan network connection/socket yang masih dapat
ditemukan di memory.

**Penerapan perintah:**

Umum:

``` bash
vol -f MemoryDump.mem windows.netscan
```

Filter PID:

``` bash
vol -f MemoryDump.mem windows.netscan | grep -E "\s5896\s"
```

Filter proses:

``` bash
vol -f MemoryDump.mem windows.netscan | grep "oneetx.exe"
```

`netscan` tidak menggunakan `--pid` seperti beberapa plugin lain,
sehingga filter output menggunakan `grep`.

Contoh yang ditemukan:

``` text
TCPv4  10.0.85.2  55462  77.91.124.20  80  CLOSED  5896  oneetx.exe
```

Dari sini: - `10.0.85.2` = local IP - `55462` = local port -
`77.91.124.20` = remote IP - `80` = remote port - `5896` = PID -
`oneetx.exe` = owner

  Column        Penjelasan
  ------------- ---------------------------------
  Offset        Alamat object network di memory
  Proto         Protocol seperti TCPv4/UDPv4
  LocalAddr     IP lokal
  LocalPort     Port lokal
  ForeignAddr   IP remote/tujuan
  ForeignPort   Port remote
  State         Status koneksi TCP
  PID           PID proses pemilik socket
  Owner         Nama proses
  Created       Waktu socket dibuat

------------------------------------------------------------------------

## 12. `--pid`

**Penjelasan:**\
`--pid` digunakan untuk membatasi analisis plugin ke proses tertentu.

**Penerapan perintah:**

``` bash
vol -f MemoryDump.mem windows.dlllist --pid 5896
```

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid 5896
```

``` bash
vol -f MemoryDump.mem windows.cmdline --pid 5896
```

Gunakan `--pid` ketika sudah menemukan proses yang ingin dianalisis
lebih dalam.

  Parameter      Penjelasan
  -------------- --------------------------------
  `--pid 5896`   Membatasi analisis ke PID 5896

**Catatan:**\
Tidak semua plugin mendukung `--pid`. Contohnya `windows.netscan`
difilter menggunakan `grep`.

------------------------------------------------------------------------

## 13. `grep`

**Penjelasan:**\
`grep` digunakan untuk mencari teks tertentu dari output command.

**Penerapan umum:**

``` bash
command | grep "kata"
```

Contoh:

``` bash
vol -f MemoryDump.mem windows.pslist | grep "oneetx.exe"
```

Case-insensitive:

``` bash
vol -f MemoryDump.mem windows.pslist | grep -i "oneetx.exe"
```

Multiple pattern:

``` bash
vol -f MemoryDump.mem windows.netscan | grep -Ei "77\.91\.124\.20|\.php"
```

Filter PID:

``` bash
vol -f MemoryDump.mem windows.netscan | grep -E "\s5896\s"
```

  Syntax          Penjelasan
  --------------- -------------------------------------------------------------
  `grep "text"`   Mencari text
  `grep -i`       Tidak membedakan huruf besar/kecil
  `grep -E`       Menggunakan extended regular expression
  `\|`            Pipe, mengirim output command pertama ke command berikutnya
  `\s`            Whitespace dalam regex
  `|` di regex    OR

------------------------------------------------------------------------

## 14. `strings`

**Penjelasan:**\
Digunakan untuk mengambil rangkaian karakter yang dapat dibaca manusia
dari file binary atau memory dump.

**Penerapan umum:**

``` bash
strings MemoryDump.mem
```

Cari IP:

``` bash
strings MemoryDump.mem | grep "77.91.124.20"
```

Cari PHP:

``` bash
strings MemoryDump.mem | grep -Ei "\.php"
```

Cari IP atau PHP:

``` bash
strings MemoryDump.mem | grep -Ei "77\.91\.124\.20|\.php"
```

  Command         Penjelasan
  --------------- -----------------------------------------------------
  `strings`       Mengambil printable strings
  `strings -el`   Mencari string encoded sebagai UTF-16 little-endian
  `grep`          Memfilter hasil

------------------------------------------------------------------------

## 15. `strings -el`

**Penjelasan:**\
Digunakan untuk mencari string UTF-16 little-endian. Ini penting pada
memory Windows karena banyak string Windows disimpan dalam format
tersebut.

**Penerapan perintah:**

``` bash
strings -el MemoryDump.mem
```

Mencari URL/PHP:

``` bash
strings -el MemoryDump.mem | grep -Ei "77\.91\.124\.20|\.php"
```

Pada kasus RedLine, command ini menemukan:

``` text
http://77.91.124.20/store/games/index.php
```

  Parameter          Penjelasan
  ------------------ ----------------------
  `-e`               Menentukan encoding
  `l`                Little-endian
  `MemoryDump.mem`   File yang dianalisis

------------------------------------------------------------------------

## 16. Analisis URL

**Penjelasan:**\
URL yang ditemukan dari memory dapat digunakan untuk menghubungkan
aktivitas proses dengan server remote.

**Penerapan perintah:**

``` bash
strings -el MemoryDump.mem | grep -Ei "https?://"
```

Spesifik PHP:

``` bash
strings -el MemoryDump.mem | grep -Ei "https?://.*\.php"
```

Pada challenge RedLine ditemukan:

``` text
http://77.91.124.20/store/games/index.php
```

  Bagian            Penjelasan
  ----------------- ----------------
  `http`            Protocol
  `77.91.124.20`    Host/IP remote
  `/store/games/`   Path
  `index.php`       File PHP

------------------------------------------------------------------------

## 17. Analisis Path Executable

**Penjelasan:**\
Path executable digunakan untuk mengetahui lokasi file program yang
dijalankan oleh proses.

**Penerapan umum:**

Cari executable:

``` bash
strings MemoryDump.mem | grep -Ei "\.exe"
```

Cari executable tertentu:

``` bash
strings MemoryDump.mem | grep "oneetx.exe"
```

Pada kasus RedLine ditemukan path device:

``` text
\Device\HarddiskVolume3\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe
```

Yang dipetakan menjadi:

``` text
C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe
```

  Bagian                 Penjelasan
  ---------------------- -----------------------
  `C:\`                  Drive Windows
  `Users\Tammam`         User profile
  `AppData\Local\Temp`   Lokasi temporary user
  `c3912af058`           Folder temporary
  `oneetx.exe`           Executable

------------------------------------------------------------------------

## 18. `Wow64`

**Penjelasan:**\
WOW64 adalah mekanisme Windows 64-bit untuk menjalankan aplikasi 32-bit.

**Penerapan:**

Ditemukan pada output `psscan`:

``` text
Wow64=True
```

Ini berarti proses tersebut merupakan proses 32-bit yang berjalan pada
Windows 64-bit.

**Catatan penting:**\
Jangan menggunakan `Wow64=True` sebagai indikator malware.

  Nilai     Penjelasan
  --------- -----------------------------------
  `True`    Proses 32-bit pada Windows 64-bit
  `False`   Bukan proses WOW64

------------------------------------------------------------------------

## 19. `tun2socks.exe` dan `Outline.exe`

**Penjelasan:**\
Dalam analisis VPN, penting membedakan aplikasi utama dengan helper
process.

**Penerapan:**

Dari process tree:

``` text
explorer.exe
└── Outline.exe
    ├── Outline.exe
    └── tun2socks.exe
```

`tun2socks.exe` merupakan helper yang menangani bagian tunnel/network,
sedangkan `Outline.exe` merupakan aplikasi VPN utama.

Pada soal:

``` text
What is the name of the process responsible for the VPN connection?
```

Jawaban yang digunakan:

``` text
Outline.exe
```

  Process           Peran dalam analisis
  ----------------- -----------------------------
  `Outline.exe`     Aplikasi VPN utama
  `tun2socks.exe`   Helper untuk traffic/tunnel

------------------------------------------------------------------------

## 20. Korelasi PID

**Penjelasan:**\
PID digunakan sebagai penghubung antarartefak memory.

**Penerapan:**

Contoh:

``` text
PID 5896
    ↓
oneetx.exe
    ↓
netscan
    ↓
77.91.124.20
    ↓
URL
    ↓
path executable
```

Command yang dapat digunakan:

``` bash
vol -f MemoryDump.mem windows.pslist | grep "5896"
```

``` bash
vol -f MemoryDump.mem windows.cmdline --pid 5896
```

``` bash
vol -f MemoryDump.mem windows.dlllist --pid 5896
```

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid 5896
```

``` bash
vol -f MemoryDump.mem windows.netscan | grep -E "\s5896\s"
```

  Artefak     Yang dicari
  ----------- ------------------------
  `pslist`    Identitas proses
  `pstree`    Parent/child
  `cmdline`   Cara proses dijalankan
  `dlllist`   DLL yang dimuat
  `vadinfo`   Region memory
  `netscan`   Network connection

------------------------------------------------------------------------

## 21. `pslist` vs `psscan`

**Penjelasan:**\
Keduanya digunakan untuk analisis proses, tetapi cara menemukan proses
berbeda.

**Penerapan:**

``` bash
vol -f MemoryDump.mem windows.pslist
```

``` bash
vol -f MemoryDump.mem windows.psscan
```

Gunakan keduanya untuk membandingkan proses yang terlihat secara normal
dengan process object yang ditemukan melalui scanning memory.

  Plugin     Fungsi
  ---------- --------------------------------------------
  `pslist`   Melihat proses yang terdaftar/aktif
  `psscan`   Scan memory untuk menemukan process object

------------------------------------------------------------------------

## 22. Process Tree

**Penjelasan:**\
Process tree membantu mengetahui proses mana yang menjalankan proses
lain.

**Penerapan:**

``` bash
vol -f MemoryDump.mem windows.pstree
```

Contoh:

``` text
explorer.exe
└── Outline.exe
    └── tun2socks.exe
```

Untuk investigasi malware, hubungan parent-child dapat membantu memahami
bagaimana sebuah executable dijalankan.

  Elemen   Penjelasan
  -------- ---------------------------------------
  Parent   Proses yang menjalankan/menjadi induk
  Child    Proses yang dibuat oleh parent
  PID      ID proses
  PPID     ID parent

------------------------------------------------------------------------

## 23. Workflow Dasar Memory Forensics

**Penjelasan:**\
Analisis memory sebaiknya dilakukan secara bertahap, bukan langsung
mencari malware.

**Penerapan perintah:**

``` bash
vol -f MemoryDump.mem windows.info
```

``` bash
vol -f MemoryDump.mem windows.pslist
```

``` bash
vol -f MemoryDump.mem windows.pstree
```

``` bash
vol -f MemoryDump.mem windows.psscan
```

``` bash
vol -f MemoryDump.mem windows.cmdline
```

``` bash
vol -f MemoryDump.mem windows.dlllist --pid <PID>
```

``` bash
vol -f MemoryDump.mem windows.vadinfo --pid <PID>
```

``` bash
vol -f MemoryDump.mem windows.malware.malfind --pid <PID>
```

``` bash
vol -f MemoryDump.mem windows.netscan
```

``` bash
strings -el MemoryDump.mem | grep -Ei "http|\.php|\.exe"
```

  Tahap       Tujuan
  ----------- ------------------------------
  `info`      Identifikasi OS
  `pslist`    Daftar proses
  `pstree`    Hubungan proses
  `psscan`    Scan process object
  `cmdline`   Argument proses
  `dlllist`   DLL
  `vadinfo`   Memory region
  `malfind`   Kandidat memory mencurigakan
  `netscan`   Network
  `strings`   String/artefak teks

------------------------------------------------------------------------

## 24. Contoh Alur Investigasi RedLine

**Penjelasan:**\
Pada challenge RedLine, proses investigasi dilakukan dengan
menghubungkan beberapa artefak.

**Penerapan:**

``` text
MemoryDump.mem
      ↓
windows.info
      ↓
Windows 10 x64
      ↓
pslist / psscan
      ↓
oneetx.exe — PID 5896
      ↓
pstree / PPID
      ↓
vadinfo
      ↓
PAGE_EXECUTE_READWRITE
      ↓
netscan
      ↓
77.91.124.20
      ↓
strings -el
      ↓
http://77.91.124.20/store/games/index.php
      ↓
path executable
      ↓
C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe
```

  ------------------------------------------------------------------------------------------------
  Artefak                             Hasil penting
  ----------------------------------- ------------------------------------------------------------
  Process                             `oneetx.exe`

  PID                                 `5896`

  Network                             `77.91.124.20`

  PHP URL                             `http://77.91.124.20/store/games/index.php`

  Executable                          `oneetx.exe`

  Full path                           `C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe`
  ------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 25. Prinsip Analisis yang Perlu Diingat

**Penjelasan:**\
Memory forensics bukan sekadar menjalankan banyak plugin. Yang paling
penting adalah menghubungkan hasil antarplugin.

**Penerapan:**

``` text
Process
  ↓
PID
  ↓
PPID / Parent
  ↓
Command Line
  ↓
DLL
  ↓
Memory Region
  ↓
Network
  ↓
URL / IP
  ↓
File / Path
```

  Konsep         Yang perlu dicari
  -------------- ------------------------------------------
  Process        Apa yang sedang berjalan?
  PID            Identitas prosesnya apa?
  PPID           Siapa parent-nya?
  Command line   Bagaimana proses dijalankan?
  DLL            Library apa yang dimuat?
  VAD            Memory region seperti apa yang dimiliki?
  Network        Berkomunikasi dengan siapa?
  URL            Mengakses resource apa?
  Path           Executable berasal dari mana?
