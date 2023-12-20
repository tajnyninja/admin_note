# grep

Komenda `grep "^\s*linux" /boot/grub/grub.cfg | grep -v "security=apparmor"` wykonuje następujące czynności w systemie Linux:

1. **grep "^\s*linux" /boot/grub/grub.cfg**: Ta część komendy wyszukuje linie w pliku `/boot/grub/grub.cfg`, które zaczynają się od słowa "linux", przy czym może zawierać dowolną ilość białych znaków na początku linii. Znak `^` oznacza początek linii, a `\s*` oznacza dowolną ilość białych znaków (np. spacji lub tabulatorów) przed słowem "linux".
  
2. **|**: To jest operator przekierowania potoku, który przekazuje wynik poprzedniej komendy do kolejnej komendy.
  
3. **grep -v "security=apparmor"**: Ta część komendy filtruje wynik poprzedniego polecenia, wykluczając (za pomocą opcji -v) linie zawierające frazę "security=apparmor".
  

W rezultacie, ta komenda przeszukuje plik konfiguracyjny GRUB (bootloadera) w poszukiwaniu linii rozpoczynających się od "linux" (z dowolną ilością białych znaków na początku) i następnie eliminuje linie, które zawierają "security=apparmor".

# readlink

Komenda `$(readlink -e /etc/issue)` służy do wyświetlenia rzeczywistej ścieżki pliku `/etc/issue`, jeśli jest ona skrótem lub dowiązaniem symbolicznym.

Oto wyjaśnienie krok po kroku:

1. **readlink -e /etc/issue**: Wykonuje polecenie `readlink` z opcją `-e`, które zwraca rzeczywistą ścieżkę pliku `/etc/issue`, jeśli jest ona dowiązaniem symbolicznym.
  
2. **$()**: Jest to składnia podstawienia poleceń w Bash. Oznacza to, że wynik polecenia `readlink -e /etc/issue` będzie używany jako argument dla polecenia zewnętrznego.
  

Ostatecznie, wykonanie tej komendy spowoduje wyświetlenie rzeczywistej ścieżki pliku `/etc/issue`, jeśli jest ona skrótem lub dowiązaniem symbolicznym.
