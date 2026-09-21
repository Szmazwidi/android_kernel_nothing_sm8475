# Nothing Phone (2) / Pong — Linux 5.10.270 — Experimental

Źródła kernela dla projektu Android 17 / LineageOS 24.0.
Changelog uporządkowany 21.09.2026. Stan kodu przed dokumentacją: `e457b869750d1dd52995c61b07d7d1418e448b83`.

## Gałęzie repozytorium

| Gałąź | Przeznaczenie |
| --- | --- |
| [`lineage-24.0`](../../tree/lineage-24.0) | Dotychczasowa baza forka; zachowana bez zmian podczas porządkowania. Zawiera już lokalne poprawki Ponga, nie jest deklarowana jako czysty upstream LineageOS. |
| [`pong-a17-5.10.270-baseline`](../../tree/pong-a17-5.10.270-baseline) | Punkt odniesienia: Linux 5.10.270, integracja Android Common i wcześniejsze poprawki PM/UFS. |
| [`pong-a17-5.10.270-experimental`](../../tree/pong-a17-5.10.270-experimental) | Baseline oraz skumulowane eksperymentalne poprawki i backporty. |

## Changelog bazowy — względem `lineage-24.0`

### Linux i Android Common

- Aktualizacja bazy z 5.10.257 do **5.10.270**, przez integrację Android Common 5.10.269 i wydania Linux stable 5.10.270.
- Włączenie zmian upstream dotyczących m.in. pamięci, systemów plików, sieci, Bluetooth i obsługi błędów. Nie wszystkie dotyczą sprzętu Ponga.
- Adaptacje integracji do istniejących interfejsów Androida i Qualcomma: m.in. MHI, platform shutdown, TCP oraz sprawdzanie długości pakietów QRTR.

### Pamięć masowa i usypianie

- Poprawki współpracy SCSI/UFS power management z obsługą błędów, w tym usunięcie zakleszczenia między PM a SCSI error handler.
- Śledzenie systemowego suspend/resume oraz poprawki przełączania trybu zasilania i timeoutów START STOP UNIT.
- UFS multi-clear: czyszczenie wielu poleceń oraz usunięcie wyścigu między przerwaniem a resetem kontrolera.
- Dostosowanie multi-clear do lokalnego modelu blokowania (`hba->host->host_lock`) i atomowego kończenia żądań.
- Zachowanie wcześniejszych zmian Ponga, w tym sekwencji zasilania CNSS2/QCA6490 i poprawki inicjalizacji tabeli częstotliwości termicznych.

Baseline jest punktem porównawczym projektu, nie samym czystym tagiem Linux 5.10.270.

## Changelog eksperymentalny — dodatkowo względem baseline

### Pamięć, zRAM i kompresja

- Aktualizacja biblioteki **LZ4 do 1.10.0**, wraz z adaptacją API i eksportami dla modułowego backendu zRAM.
- Backport nowszych implementacji **zsmalloc i zRAM**: zmiany blokowania i mapowania obiektów, osobne backendy kompresji, parametry algorytmów, obsługa słowników i priorytetów rekompresji.
- W konfiguracji Ponga włączony backend LZ4 i LZ4 jako domyślny algorytm zRAM. Ustawienia startowe Androida mogą wybrać inny algorytm; faktyczny stan należy sprawdzić na urządzeniu.
- Eksperymentalny **Kcompressd**: przenoszenie kwalifikującej się pracy swap z kswapd do workera, ograniczona kolejka oraz obsługa referencji stron, zatrzymywania workera i ścieżki awaryjnej. Włączony w konfiguracji Ponga.
- Dostosowanie przechowywania FIFO Kcompressd do Linux 5.10, z zachowaniem późniejszej lokalnej poprawki.
- Przebudowa synchronicznego swap I/O: pomocnicze funkcje odczytu/zapisu, bio na stosie dla odczytu synchronicznego, usunięcie starego `rw_page`, adaptacja mpage i zswap oraz domknięcie cyklu życia bio.
- Oznaczenie synchronicznego I/O zRAM wykorzystywane przez swapon i kwalifikację do Kcompressd.
- Usprawnienia iterowania po wektorach bio i pomocniczych operacji kopiowania.

### GPU i synchronizacja

- KGSL: bezpieczniejsze indeksowanie tablic GMU, sprawdzanie rozmiaru danych ioctl i atomowy stan deskryptorów pamięci.
- Poprawka raportowania GPUREADONLY w debugfs oraz pomylenia typów w ścieżce LPAC.
- Użycie sortowania kernela podczas łączenia dma-fence.

### FastRPC / DSP

- Walidacja zakresów stron przekazywanych do DSP i poprawka użycia zwolnionej pamięci w asynchronicznych licznikach wydajności.
- Oddzielna obsługa nakładających się buforów ION i non-ION; lokalna adaptacja klasyfikacji deskryptorów uwzględnia również fd 0.

### Sieć

- **BBRv3** wraz z wymaganymi zmianami próbkowania TCP, ECN, retransmisji i TSO oraz obsługą PLB.
- BBR jest dostępne jako algorytm `bbr`; **domyślnie pozostaje CUBIC**, a PLB domyślnie wyłączone.
- Zachowanie opcjonalności nowego callbacku dla programów BPF TCP struct_ops.
- CNSS2: awaryjny timer restartu w ścieżce odzyskiwania działania. Nie zastępuje wcześniejszej poprawki zasilania Wi-Fi.

### Zasilanie, USB i urządzenia wejściowe

- Blokowanie współdzielonego stanu w governorach Qualcomm LPM/cluster LPM; dodatkowa walidacja wyboru stanu cluster idle.
- Obsługa nieoczekiwanych przerwań zakończenia RPMh i poprawka licznika warm reset PMIC.
- USB: poprawki zakleszczenia podczas wybudzania DWC3, kontroli wskaźników przy suspend, unieważniania TD xHCI i ponawiania konfiguracji MIDI.
- Goodix: obsługa gestu podwójnego dotknięcia z lokalną adaptacją zachowującą dotychczasowe gesty wybudzania; ograniczenie nadmiernego logowania dotyku i haptyki.
- GENI I2C: bariery pamięci po zapisach rejestrów i poprawna obsługa nieoczekiwanych przerwań.

### Pozostałe poprawki

- Ochrona statystyk UFS przed dzieleniem przez zero.
- Zmiana blokady statystyk UID na rt_mutex i dodanie wykrywania kontencji rwlock.
- Tablica haszująca do globalnego wyszukiwania zegarów oraz wymagane adaptacje pomocniczych API.

## Zależności poza tym repozytorium

Pełny zestaw źródeł Androida obejmuje także osobne repozytorium
`Szmazwidi/android_kernel_nothing_sm8475-modules`.
Lokalny zestaw to commit `0a41fca1b13a0baaf42ccaccb08187f5e5c0cf5c`:

- poprawka wyścigu w DSI ISR;
- Wi-Fi qcacmn: kontrola granic scatterlist oraz HTC/HIF;
- zabezpieczenie tabeli chainmask i propagacja błędu jej alokacji.

Te zmiany **nie znajdują się w tym repozytorium kernela**. Porządkowanie jego gałęzi
nie publikuje automatycznie repozytorium modułów; wskazany SHA identyfikuje lokalny
zestaw integracyjny, nie gwarantuje jego dostępności na GitHubie.

## Walidacja i ograniczenia

- Najnowsze pakiety BBRv3, Kcompressd i swap/Wi-Fi/fences przeszły przegląd statyczny; pełny build i testy działania tej końcowej kombinacji pozostają do wykonania.
- Podczas porządkowania gałęzi nie uruchamiano kompilacji ani flashowania.
- Wcześniejsze testy PM i wcześniejszych paczek nie oznaczają walidacji obecnego HEAD.
- Do sprawdzenia po przyszłym buildzie: boot, Wi-Fi, presja pamięci, swap/writeback, suspend/resume oraz TCP z CUBIC i BBR.
- Zmiany struktur TCP oraz API pamięci/I/O wymagają zgodnych modułów. Nie zakładamy zgodności ze starymi modułami binarnymi.
- Nie ma jeszcze pomiarów potwierdzających wzrost FPS, poprawę czasu pracy na baterii ani zysk z Kcompressd.

## Pochodzenie zmian

Historia Linux stable, Android Common, LineageOS i lokalnych zmian pozostaje zachowana.
Backporty pochodzą m.in. z upstreamu Linux, Qualcomma oraz drzew arter97 i innych
projektów wymienionych w opisach commitów. Zachowano autorów, źródłowe SHA i opisy
adaptacji; porządkowanie gałęzi nie squashuje ani nie przepisuje tej historii.

Oryginalne wskazówki dla kontrybutorów Android Common zachowano w
[`README.android-common.md`](README.android-common.md). Ogólna dokumentacja Linux
pozostaje w [`README`](README) i katalogu `Documentation/`.
