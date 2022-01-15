# myOWNgoogleDOC
## Wstęp
___

Aplikacja jest aplikacją sieciową działającą w sieci lokalnej. Jest ona stosowana do obsługi pliku tekstowego współbieżnie przez wielu użytkowników. W domyśle server jest włączony na systemie Linux, a klienci są użytkownikami systemu Windows. Dla uproszczenia użyliśmy jednego pliku globalnego. 

## Opis zastosowanych metod
___

### Każdy użytkownik w trakcie wykonywania programu tworzy 3 pliki txt. Są to kolejno:
* file1.txt - to co na pewno jest na serwerze; do tego pliku co 3sec pobierana jest zawartość pliku globalnego będącego na serwerze
* file2.txt - to co być może jest na serwerze; jest niezbędny do porównań (wyjaśnienie poniżej)
* file3.txt - workspace; na tym pliku pracuje użytkownik

### W trakcie programu wykonywane są 2 porównania:
* pomiędzy file1, oraz file2 - jeśli istnieje różnica, to znaczy że zawartość pliku globalnego na serwerze uległa zmianie - ustaw stan 'r'
* pomiędzy file2, oraz file3 - jeśli istnieje różnica, to znaczy że użytkownik chce podmienić zawartość pliku globalnego - ustaw stan 'w'
* jeżli nie ma żadnych różnic ustaw stan 'k'

### Stany:
* stan 'r' - read; napisz wszystkie pliki tym co przyszło z serwera (jest w file1)
* stan 'w' - write; zapisz na serwerze zawartość pliku workspace i skopiuj jej zawartość do file1, oraz file2
* stan 'k' - OK; nie rób nic; skanuj dalej

## Instrukcja obslugi
___

rychu

### Instalacja pluginu Notepad++

rychu

## Potencjalne drogi rozwoju aplikacji
___

* Aplikacja jest rodzajem problemu czytelników i pisarzy. Rozwiązaniem tego problemu byłoby użycie w odpowiednim miejscu semaforów blokujących. Bez tego może występować problem z brudnym zapisem. 
* Można dodać obsługę wielu plików na serwerze.