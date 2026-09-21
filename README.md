# Nothing Phone (2) / Pong — Linux 5.10.270 — Baseline

Źródła kernela dla projektu Android 17 / LineageOS 24.0.
Changelog uporządkowany 21.09.2026. Stan kodu przed dokumentacją: `d9b84e9900f957c94156f190859dde4593e4308f`.

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

## Walidacja i ograniczenia

- Wcześniejszy stos SCSI/UFS PM był testowany na Pongu: w opisanym oknie 98 udanych wejść w suspend, bez zaobserwowanych błędów resume i UFS/SCSI. Nie był to test wymuszonych timeoutów ani wszystkich ścieżek awaryjnych.
- Ten wynik dotyczy wcześniejszego etapu stosu; nie należy przypisywać go automatycznie każdemu późniejszemu commitowi ani całej gałęzi eksperymentalnej.
- Podczas tego porządkowania zweryfikowano historię i zmiany dokumentacji; nie wykonywano nowego buildu ani testu urządzenia. Nazwa baseline nie stanowi deklaracji pełnej walidacji sprzętowej.

## Pochodzenie zmian

Historia Linux stable, Android Common, LineageOS i lokalnych zmian pozostaje zachowana.
Backporty pochodzą m.in. z upstreamu Linux, Qualcomma oraz drzew arter97 i innych
projektów wymienionych w opisach commitów. Zachowano autorów, źródłowe SHA i opisy
adaptacji; porządkowanie gałęzi nie squashuje ani nie przepisuje tej historii.

Oryginalne wskazówki dla kontrybutorów Android Common zachowano w
[`README.android-common.md`](README.android-common.md). Ogólna dokumentacja Linux
pozostaje w [`README`](README) i katalogu `Documentation/`.
