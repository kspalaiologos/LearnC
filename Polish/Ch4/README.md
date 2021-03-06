
# Struktura programu w C

## Słowa kluczowe C11

```
auto      extern   short     while
break     float    signed    _Alignas
case      for      sizeof    _Alignof
char      goto     static    _Atomic
const     if       struct    _Bool
continue  inline   switch    _Complex
default   int 	   typedef   _Generic
do        long     union     _Imaginary
double 	  register unsigned  _Noreturn
else      restrict void      _Static_assert
enum      return   volatile  _Thread_local
```

Nie musisz znać wszystkich z wymienionych powyżej słów kluczowych. Możesz odwołać się do tej tabeli, jeśli tylko będzie Ci potrzebna.

## Biblioteki

Biblioteka to zbiór narzędzi, które zostały wydzielone po to, aby dało się z nich korzystać w wielu programach. Ułatwiają one programowanie poprzez rozszerzanie aplikacji. Każda biblioteka posiada swoje pliki nagłówkowe, które zawierają deklaracje funkcji oraz często zawarte są w nich komentarze, jak używać danej funkcji.

Biblioteki dzielimy według sposobu wykorzystania, autora i rozszerzenia.

Według wykorzystania biblioteki mogą być statyczne, (Windows: .lib; Unix .a, można zaliczyć do tego pliki obiektowe odpowiednio .obj i .o) lub dynamiczne (Unix: .so, ścieżki do plików w /etc/ld.so.conf oraz $LD_LIBRARY_PATH; Windows: .dll)

Według autora biblioteki mogą być standardowe (dołączane do 99% kopii kompilatora i systemu operacyjnego) lub niestandardowe (dołączane rzadko lub często; mogą być specyficzne dla platformy, w części przypadków są nieprzenośne).

I finalnie, według rozszerzenia (dla Uniksopodobnych):
 * `.a`   - biblioteka statyczna;
 * `.bin` - plik binarny, często plik wykonywalny bez widocznego nagłówka formatu pliku wykonywalnego.
 * `.fw`  - pliki skompilowanego firmware
 * `.ko`  - moduły jądra
 * `.o`   - pliki obiektowe
 * `.so`  - dynamicznie linkowane biblioteki współdzielone
 
GCC najczęściej szuka bibliotek na komputerach pod kontrolą systemu unikspodobnego w:
 * /usr/local i target/include
 * /usr/include
 
Linkowanie statyczne polega w skrócie, dołączeniu kodu funkcji do programu na etapie kompilacji.

Linkowanie dynamiczne polega na dołączaniu do programu jedynie odpowiednich odwołań do funkcji bibliotecznych. Kod funkcji **nie** jest dołączany do programu na etapie kompilacji. Przy uruchomieniu programu, biblioteka jest natomiast ładowana do pamięci i udostępnia odpowiednie funkcje pracującemu programowi. Uwaga: Jeśli używasz bibliotek linkowanych dynamicznie, spodziewaj się możliwości, w której natrafisz na tzw. ***DLL Hell*** (komplikacje, które pojawiają się przy korzystaniu z różnych wersji bibliotek dynamicznych)

Podstawową zaletą linkowania dynamicznego jest to, że jest wymagany jeden egzemplarz biblioteki w pamięci dla wszystkich programów które z niej korzystają. Wadą jest to, że program linkowany dynamicznie jest zależny od dostępności zewnętrznym plików (bibliotek) i nie będzie działał poprawnie jeżeli ich nie znajdzie.

Aby możliwe było linkowanie dynamiczne biblioteka musi być specjalnie skonstruowana. Biblioteki dynamiczne pozwalają na linkowanie dynamiczne i statyczne. Zdecydowana większość używanych bibliotek to biblioteki dynamiczne, niezwykle rzadko zdarzają się przypadki bibliotek nadających się jedynie do linkowania statycznego. 

## Budowa pliku nagłówkowego biblioteki

Najczęściej pliki nagłówkowe mają następującą postać:

```
#ifndef _FILE_H
#define _FILE_H

/* ... */

#endif
```

Takie rozwiązanie zapobiega dołączaniu pliku wielokrotnie w jednej jednostce. Jeśli teoretycznie nagłówki dołączałyby się rekursywnie, to zapobiega niezliczonym błędom. W plikach nagłówkowych często umieszcza się definicje typów, z których korzysta biblioteka, makra i deklaracje funkcji.

## Tworzenie prostej biblioteki

Zakładając że biblioteka będzie posiadać wyłącznie jedną funkcję, możemy stworzyć ją w następujący sposób:

```
#ifndef _NAZWA_AUTOR_H
#define _NAZWA_AUTOR_H

#include <stdio.h>

void hello(void) {
    puts("Hello World!");
}

#endif

```

Należy pamiętać, o podaniu void w liście argumentów funkcji nie przyjmujących argumentów. W prototypie nie ma informacji na temat tego jakie argumenty funkcja przyjmuje, jeśli nawiasy są puste.

Plik ten należy zapisać jako "hello.h".

UWAGA: Jeśli spróbujesz dołączyć ten plik w dwóch oddzielnych plikach i połączyć, otrzymasz błąd kompilacji. Dzieje się tak, ponieważ wewnątrz jednej aplikacji znajduje się kilka funkcji nazwanych tak samo (`hello` i `hello`). 

Praktyczne użycie:

```
#include "hello.h"
 
int main(void) {
   hello();
}
```

Zakładając że nazwa powyższego pliku to `main.c`, komenda kompilacji wygląda "normalnie":

```
gcc main.c -o main
./main
```

=>

```
Hello World!
```

Zalecam Ci, aby implementacje funkcji w plikach nagłówkowych w dużych projektach (czego osobiście nie zalecam) były statyczne. Pomoże zapobiec to konfliktom w czasie linkowania. Poprawiona wersja biblioteki:

```
#ifndef _NAZWA_AUTOR_H
#define _NAZWA_AUTOR_H

#include <stdio.h>

static void hello(void) {
    puts("Hello World!");
}

#endif

```

## Preprocesor

Preprocesor to program analizujący kod źródłowy w poszukiwaniu dyrektyw preprocesora. Na podstawie tych instrukcji generuje on kod w "czystym" języku C, który następnie jest kompilowany przez kompilator. Ponieważ za pomocą preprocesora można sterować kompilatorem daje on możliwości, które nie były dotąd znane w innych językach programowania.

W języku C wszystkie linie zaczynające się od symbolu "#" są dyrektywami preprocesora. Nie są elementami języka C i nie podlegają bezpośrednio procesowi kompilacji.

Preprocesory są najczęściej wbudowane w kompilator. Istnieją również samodzielne preprocesory (GNU M4, Cog, czy chociażby mój PCP). Aby zobaczyć kod wytworzony przez preprocesor użyj przełącznika `-E`.

```
gcc test.c -E -o test.i
```

Ten plik może być dość duży, więc nie będę wstawiał tutaj mojego wyniku wykonywania tego polecenia.

Dyrektywy preprocesora to wyrażenia rozpoczynające się "#". Dyrektywa kończy się końcem linii, jednak można przenieść ją do następnej linii znakiem "\".

```
#define DODAJ(a,b) \
  a+b
```

### `#include`

To jedna z częściej wykorzystywanych dyrektyw, którą już znasz. Sprawia, że plik podany między nawiasami kwadratowymi (jeśłi jest nagłówkiem z include path) lub cudzysłowiem (jeśli znajduje się w lokalizacji relatywnej do pliku .c/.h) zostaje dopisany z całą swoją zawartością w miejsce dyrektywy.

### `#define`

Ta dyrektywa pozwala na zdefiniowanie stałej, makra, słowa kluczowego i funkcji. Przykład:

```
#define LICZBA 42
#include <stdio.h>

int main(void) {
    return LICZBA;
}
```

Zwróci 42. Można również definiować stałą jako "nic":

```
#define LICZBA
#include <stdio.h>

int main(void) {
    return LICZBA-1;
}
```

Zwróci -1. Przykładem makra może być:

```
#define SUMA(x,y) ((x)+(y))
```

Należy pamiętać, że parametry najbezpieczniej jest podawać w nawiasie, a całe makro również wrzucić w nawias. Zapobiegnie to powstaniu różnych dziwnych sytuacji. Przykładem takiej "niespodzianki" jest klasyk:

```
#include <stdio.h>

#define SIX 1+5
#define NINE 8+1

int main(void) {
    printf("%d * %d = %d\n", SIX, NINE, SIX * NINE);
    return 0;

}
```

... który na STDOUT wypisze `6 * 9 = 42` zamiast `6 * 9 = 54`. Przyjrzyjmy się programowi dokładniej. Jeśli odetniemy z wyjścia nagłówek stdio i wszystkie jego zależności, jak i makra wstawiane przez preprocesor, kod będzie wyglądał tak:

```
int main(void) {
    printf("%d * %d = %d", 1+5, 8+1, 1+5*8+1);
    return 0;
}
```

Uprośćmy to lekko:

```
int main(void) {
    printf("%d * %d = %d", 6, 9, 1+5*8+1);
    return 0;
}
```

Widać że pierwsze dwie liczby się zgadzają, ale potem mamy zaskoczenie: `1+5*8+1`. Znając kolejność wykonywania działań, jest to `1+40+1`, czyli `42`. Jak temu zapobiec? Bardzo prosto. Spójrzmy jeszcze raz na stary program:

```
#include <stdio.h>

#define SIX (1+5)
#define NINE (8+1)

int main(void) {
    printf("%d * %d = %d\n", SIX, NINE, SIX * NINE);
    return 0;

}
```

Teraz na STDOUT zostanie wypisany prawidłowy wynik. Przykład ten pochodzi z książki K&R i ma demonstrować niewłaściwe użycie preprocesora.

### `#undef`

Ta dyrektywa odwraca działanie `#define`. Przykład:

```
#define SOMETHING 3
const int sth = SOMETHING; /* sth = 3 */
#undef SOMETHING
const int sth2 = SOMETHING; /* error */
```

### `#if #else #elif #endif` - kompilacja warunkowa

Powyższe dyrektywy procesora uzależniają przebieg późniejszej kompilacji od stałych znanych w czasie procesu wstępnego przetwarzania kodu źródłowego:

 * `#if` sprawia że blok aż do `#endif`/`#else`/`#elif`, jeśli nie jest prawdziwy, zostaje wycięty z wynikowego kodu źródłowego, co sprawia że nie jest uwzględniony w pliku binarnym.
 * `#else` spowoduje skompilowanie kodu jeżeli warunek odpowiadający `#if` jest fałszywy, aż do napotkania `#elif`, `#endif` lub `#if`.
 * `#elif` to połączenie `#if` i `#else`.
 * `#endif` zamyka blok otworzony przez `#if`, `#else` lub `#elif`.

Przykład:

```
#define OS 0 /* 0-Windows, 1-Linux, 2-MacOS */

#if OS == WINDOWS
    printf ("You are using Windows");
#elif OS == LINUX
    printf ("You are using Linux");
#else
    printf ("Are you using MacOS?");
#endif
```

### `#ifdef #ifndef`

Dyrektywy preprocesora `#ifdef #ifndef` działają podobnie, ale tylko sprawdzają czy dany symbol został wcześniej zdefiniowany lub nie, z jakąkolwiek wartością. Przejdźmy od razu do przykładu, przytaczając sobie plik nagłówkowy twojej pierwszej biblioteki:

```
#ifndef _NAZWA_AUTOR_H
#define _NAZWA_AUTOR_H

/* ... */

#endif
```

Widać tutaj dyrektywę `#ifndef` - kod między tą dyrektywą a `#endif` będzie kompilowany wyłącznie jeśli `_NAZWA_AUTOR_H` nie zostało wcześniej zdefiniowane. Następnie definiujemy `#define _NAZWA_AUTOR_H`. Czemu?

Taki zabieg pozwala ominąć różne nieprzyjemne komplikacje wynikające z podwójnego dołączenia pliku. Możemy np dwukrotnie zadeklarować funkcję (wtedy nastąpi konflikt dwóch wcześniejszych zadeklarowań).

### `#error`

Dyrektywa `#error` wywołuje błąd kompilacji, jeśli tylko preprocesor ją napotka. Dziwne, nielogiczne? Spójrz na prosty przykład. Piszesz bibliotekę która może być kompilowana tylko pod systemami UNIX-like (bo np. korzysta z pliku nagłówkowego <unistd.h> zawierającego część funkcji z POSIXa, którego na próżno szukać na Windows-like). Teraz użytkownik chce to skompilować używając systemu Windows. Co się dzieje? Tona błędów które prawdopodobnie nic nie znaczą dla kompilującego. I co teraz? Kompilujący może ticketować issue z projektem (buuuu, nie buduje się). Aby tego uniknąć użyj `#error`.

```
#if defined (__unix__) || (defined (__APPLE__) && defined (__MACH__))
    /* Ok */
    #include <unistd.h>
    /* ... */
#else
    #error "This library can't be built and ran under non-POSIX operating system."
#endif
```

### `#warning`

Dyrektywa `#warning` ma podobne zastosowanie do `#error`, ale wywołuje ostrzeżenie a nie błąd krytyczny.

### `#line`

Dyrektywa ta powoduje wyzerowanie licznika linii. Osobiście nie widzę sensu w używaniu tej dyrektywy manualnie, w kodzie który nie był przetwarzany wstępnie przez inne programy.

```
/* aaa */
#line
/* bbb */
#line 1453
/* ccc */
```

### `#pragma`

Dyrektywa `#pragma` służy m.in. do tworzenia wątków w OpenMP. Z język aangielskiego "Pragmatic Information". Jest też wykorzystywana w kompilatorach Microsoftu m.in., dla zastępstwa dla include guards (konstrukcji wspomnianej przy `#ifndef`; `#pragma once`, jednak nie zalecam używania tej dyrektywy) 

### `## #`

Dość ciekawe możliwości ma w makrach "operator" #. Zamienia on stojący za nim identyfikator na napis. Przykład jest bardzo prosty:

```
#define tostring(identifier) printf("%s = %i\n", #x, x)

/* ... */

int x = 23;
tostring(x);
```

... i wyjście:

```
x = 23
```

"Operator" `##` jest lekko mniej użyteczny w bardziej casualowych programach. `##` łączy dwa identyfikatory w jeden. Przykład:

```

#include <stdio.h>

#define decl(x) int x##_variable
#define echo(y) printf("%s=%i", #y, y)

int main() {
    decl(abc) = 2;
    echo(abc_variable);
    return 0;
}

```

### Makra

Preprocesor C umożliwia tworzenie makr, czyli "funkcji", które są wstawiane do kodu.
Makra deklaruje się za pomocą dyrektywy `#define`, jak na poniższym schemacie:

```
#define macro(arg1, arg2, ...) (expr)
```

W momencie wystąpienia `macro` w kodzie źródłowym, preprocesor automatycznie zamieni makro na wyrażenie lub instrukcje. Ponieważ makro sprowadza się do prostego zastąpienia przez preprocesor wywołania makra przez jego tekst, jest bardzo podatne na trudne do zlokalizowania błędy. Makra są szybsze (nie następuje wywołanie funkcji, które zawsze zajmuje trochę czasu głównie na x86), ale też mniej elastyczne i miejscowo wydajne niż funkcje. 

A w praktyce? Przykład:

```
#define square(x) ((x)*(x))
```

Preprocesor w miejsce wyrażenia `square(2)` wstawi `((2)*(2))` => `4`. A co jakby napisać `square("2")`? Preprocesor po prostu wstawi napis do kodu, co da wyrażenie `(("2")*("2"))`, które jest nieprawidłowe (chyba że mówimy o języku JavaScript, gdzie wszystko można). Kompilator zgłosi błąd, ale jego szukanie może sprawić drobną trudność. Widać, że bezpieczniejsze jest użycie funkcji, które dają możliwość specyfikowania sztywnej i niezmienialnej typów argumentów. 

Mówiąc o makrach często mówimy o tzw. "Side effects". "Side effects" to rzeczy które mogą się wydarzyć nieoczekiwane podczas ewaluacji makra. Bierzemy na tapetę przykład:

```
int x = 2;
int y = square(++x);
```

Czy `y == 4`? Nie. Jeśli wiesz czemu, możesz spokojnie pominąć poniższe wyjaśnienie. Otóż preprocesor zamienia `square(++x)` na `((++x)*(++x))`, czyli po wyrzuceniu nawiasów `++x * ++x`. Jak widać, mamy do czynienia z podwójną inkrementacją, zamiast pojedynczej. Jak temu zaradzić? Użyć funkcji.

Poniższe makra również są błędne:

```
#define add(a, b) a + b
#define mul(a, b) a * b
```

Odnalezienie "słabych punktów" poniższych makr możesz potraktować jako ćwiczenie, w momencie pisania widzę tu conajmniej dwa możliwe sposoby wykorzystania braku nawiasów i inkrementacji.

### Predefiniowane makra

W języku wprowadzono pokaźną liczbe predefiniowanych makr, które mają ułatwić życie programiście. Oto lista najważniejszych:

 * `__FILE__` - nazwa aktualnie przetwarzanego pliku
 * `__LINE__` - numer linii
 * `__DATE__` - data kompilacji
 * `__TIME__` - godzina kompilacji
 * `__STDC__` - w kompilatorach zgodnych z ANSI przyjmuje wartość 1
 * `__STDC_VERSION__` - zależnie od poziomu zgodności kompilatora ze standardami, makro przyjmuje głównie następujące wartości:
   * standard z 1989: makro nie jest zdefiniowane,
   * standard z 1994: makro ma wartość 199409L,
   * standard z 1999: makro ma wartość 199901L.

## Biblioteka standardowa

W którymś z początkowych rozdziałów tej książki napisane jest, że czysty rasowo język C bez biblioteki standardowej nie może zbyt wiele. Tak naprawdę, to język C sam w sobie praktycznie nie ma mechanizmów do obsługi np. i/o (co powinieneś już pamiętać, drogi Czytelniku). Dlatego też większość systemów operacyjnych posiada tzw. bibliotekę standardową zwaną też biblioteką języka C (libc). To właśnie w niej zawarte są podstawowe funkcjonalności, dzięki którym program może, na przykład, wypisać coś na ekran.

Jak biblioteka standardowa implementuje te funkcje, skoro sam język C tego nie potrafi? Odpowiedź jest prosta - biblioteka standardowa nie jest napisana wyłącznie w C. Ponieważ C jest kompilowany do kodu maszynowego, nie ma przeszkód, żeby połączyć go z językiem niskiego poziomu, jakim jest Assembly, ale o nim później. Dlatego biblioteka C udostępnia gotowe funkcje w języku C, a z drugiej za pomocą niskopoziomowych mechanizmów komunikuje się z systemem operacyjnym, który wykonuje odpowiednie czynności.

Najpopularniejsze wersje libc to m.in. `glibc` - GNU libc, `uClibc` - implementacja dla systemów embedded, `Diet libc` - odchudzona wersja biblioteki standardowej (która i tak nie potrzebuje odchudzania, tylko dopasania)

Jak sprawdzić której wersji używa twój system (zakładając że jest uniksopodobny)?

```
ldd --version
# albo
getconf GNU_LIBC_VERSION
```

Gdzie są funkcje z biblioteki standardowej?

Pisząc program w języku C używa się różnego rodzaju funkcji, takich jak na przykład `printf`. Twój pierwszy program zaczynał się od takiej linijki:

```
#include <stdio.h>
```

Dyrektywa ta mówi preprocesorowi: "tu wstaw zawartość stdio.h". Nawiasy "<" i ">" oznaczają, że plik stdio.h znajduje się w standardowym katalogu z plikami nagłówkowymi. Wszystkie pliki z rozszerzeniem `.h` są właśnie plikami nagłówkowymi. Wracając teraz do tematu biblioteki standardowej...

Każdy system operacyjny ma za zadanie udostępniać pewne funkcje na rzecz programów. Wszystkie te funkcje zawarte są właśnie w bibliotece standardowej. W systemach z rodziny UNIX nazywa się ją, jak wcześniej wspominałem, `libc`. Tam właśnie są zaimplementowane funkcje typu `puts()` albo `scanf()`.

Oprócz podstawowych funkcji wejścia-wyjścia, biblioteka standardowa udostępnia też możliwość między innymi komunikację przez sieć, wykonywanie funkcji matematycznych, itd.

| Nazwa nagłówka | Standard | Co zawiera |
|----------------|----------|------------|
| <assert.h>     | C89   | makro assert |
| <complex.h> | C99 | funkcje do zarządzania liczbami zespolonymi |
| <ctype.h> | C89 | funkcje służące do zarządzania głównie znakami (`isspace`, `tolower`, ...) |
| <errno.h> | C89 | lista błędów jakie teoretycznie mogą wystąpić podczas wykonywania funkcji z biblioteki standardowej |
| <fenv.h> | C99 | funkcje służące do zarządzania środowiskiem liczb zmiennoprzecinkowych |
| <float.h> | C89 | implementation-specific code dotyczące biblioteki liczb zmiennoprzecinkowych |
| <inttypes.h> | C99 | rozmiar typów pochodnych od `int` (np. `double`) |
| <iso646.h> | C99 | m.in. aliasy dla operatorów |
| <limits.h> |	C89 | stałe dotyczące zmiennych platform-specific typu int |
| <locale.h> |	C89 | funkcje dotyczące internacjonalizacji |
| <math.h> | C89 | funkcje matematyczne |
| <setjmp.h> | C89 | setjmp i longjmp, używane do nielokalnych skoków |
| <signal.h> | C89 | obsługa sygnałów |
| <stdalign.h> | C11 | ustawianie i sprawdzanie wyrównania obiektów |
| <stdarg.h> | C89 | obsługa zmiennej liczby argumentów do funkcji |
| <stdatomic.h> | C11 | obsługa operacji atomowych między wątkami |
| <stdbool.h> |	C99	| typ danych boolean |
| <stddef.h> |	C89 | użyteczne typy i makra |
| <stdint.h> |	C99 | standaryzuje rozmiary liczb całkowitych (np. `int8_t` => `char`) |
| <stdio.h> | C89 | standardowe funkcje wejścia i wyjścia |
| <stdlib.h> |	C89 | funkcje zezwalające np. na konwersję typów danych, generację liczb pseudolosowych, alokację pamięci, zarządzanie procesem |
| <stdnoreturn.h> | C11 | konstrukcje zezwalające na tworzenie funkcji które nie zwracają kontroli wywoływaczowi |
| <string.h> | C89 | funkcje do manipulacji ciągami |
| <tgmath.h> | C99 | type-generic funkcje matematyczne |
| <threads.h> |	C11	| zestaw narzędzi do pracy z wielowątkowością |
| <time.h> 	|	C89 | zestaw narzędzi do pracy z czasem |
| <uchar.h> | 	C11 | funkcje do zarządzania tzw. `wide-characters` |
| <wchar.h> |	C99 | funkcje do zarządzania `wide` ciągami znaków |
| <wctype.h> |	C99 | funkcje do klasyfikowania `wide-characters` zależnie od ich typów oraz konwertowania ich na wielkie lub małe litery |

Pełny opis biblioteki standardowej C znajdziesz w rozdziale 6.

Czasami korzystanie z funkcji bibliotecznych oraz standardowych plików nagłówkowych jest niepożądane. Aby wyłączyć używanie biblioteki C w opcjach kompilatora GCC należy dodać następujące argumenty:

```
-nostdinc -fno-builtin -nostdlib
```

Jeśli pliki startowe również są niepożądane, można użyć:

```
-nostartfiles
```

W systemach uniksowych możesz uzyskać pomoc dzięki narzędziu man, np.:

```
man puts
man gets  # Nie używaj gets()!
man libc
```

## Jak działa kompilator, Makefile

### Preprocesor

W pierwszej części pozwolę sobie zahaczyć głównie o preprocesor. Aby zobaczyć jak wygląda kod po tym procesie, spójrz na przykład poniżej:

```

#include <stdio.h>

int main(void) {
    printf("Hi, hello!");
}

```

Po kompilacji za pomocą polecenia

```
gcc -E example.c -o example.i
```

Naszym oczom w example.i okaże się coś takiego ([link](https://pastebin.com/yznK5DjV), ponieważ wyjście jest bardzo długie)

### GNU Make

Program prawie zawsze składa się z kilku plików źródłowych. Jeśli jest ich dużo, trzeba musimy stworzyć szybki i ładny sposób kompilacji naszego programu. Właśnie po to, aby zautomatyzować proces kompilacji powstał program make. Program make analizuje pliki Makefile i na ich podstawie wykonuje określone czynności. Najważniejszym elementem pliku Makefile są zależności oraz reguły przetwarzania. Zależności polegają na tym, że np., jeśli program ma być zbudowany z 4 plików, to na początku należy skompilować każdy z tych 4 plików, a dopiero później połączyć je w jeden cały program. Zatem zależności określają kolejność wykonywanych czynności. Natomiast reguły określają jak skompilować dany plik. Zależności tworzy się tak:

```
target: deps
    rules
```

Dzięki temu `make` zna już kolejność wykonywanych działań oraz czynności, jakie ma wykonać. Aby zbudować `target` należy wykonać polecenie `make target`. Pierwsza reguła w pliku Makefile jest regułą domyślną. Uruchamiając make bez parametrów, zbudujesz regułę domyślną. Tak więc dobrze jest jako pierwszą regułę wstawić regułę budującą końcowy plik wykonywalny, zwyczajowo taki target nazywa się all. Należy pamiętać, by sekcji `target` nie wcinać, natomiast `rules` wcinać tabulatorem. Część `deps` może być pusta.

Plik Makefile umożliwia też definiowanie pewnych zmiennych. Nie trzeba tutaj się już troszczyć o typ zmiennej, wystarczy napisać:

```
variable=value
```

W ten sposób można zadeklarować dowolnie dużo zmiennych. Zmienne mogą być różne - nazwa kompilatora, jego parametry, itd. Zmiennej używa się w następujący sposób `$(variable)`. Komentarze w pliku Makefile tworzymy zaczynając linię od znaku hash (#). Przykład:

Program nazywa się test oraz składa się z czterech plików:

    lexer.c
    parser.c
    codegen.c
    main.c

oraz plików nagłówkowych

    lexer.h
    parser.h
    codegen.h

Odpowiedni plik Makefile powinien wyglądać mniej więcej tak:

```
CC=gcc  
CFLAGS=-std=c89 -O3

all: lexer.o parser.o codegen.o main.o
    $(CC) $(CFLAGS) lexer.o parser.o codegen.o main.o -o app

lexer.o: lexer.c lexer.h
    $(CC) $(CFLAGS) lexer.c -c -o lexer.o

parser.o: lexer.c lexer.h parser.c parser.h
    $(CC) $(CFLAGS) parser.c -c -o parser.o

codegen.o: codegen.c codegen.h
    $(CC) $(CFLAGS) codegen.c -c -o codegen.o

main.o: main.c
    $(CC) main.c -c -o main.o

love:
    @echo "not war"
```

Program zależy od 4 plików z rozszerzeniem .o, potem każdy z tych plików zależy od plików .c, które program make skompiluje w pierwszej kolejności, a następnie połączy w jeden program. Nazwę kompilatora zapisano jako zmienną, ponieważ powtarza się i zmienna jest sposobem, by zmienić ją wszędzie za jednym zamachem, podobnie jak CFLAGS - flagi kompilatora. dodanie jako zależności plików z rozszerzeniem .h zapewnia rekompilację plików w których są używane zdefiniowane w nich wartości. Brak tych wpisów jest najczęstszą przyczyną braku zmian działania programu po zmianie ustawień w plikach nagłówkowych.

Używanie make jest bardzo proste. Warto na koniec naszego przykładu dodać regułę, która wyczyści katalog z plików .o:

```
clean:
    rm -f *.o app
```

Ta reguła spowoduje usunięcie wszystkich plików .o oraz programu w wersji binarnej po wykonaniu `make clean`. Można też ukryć wykonywane komendy albo dopisać własny opis czynności:

```
clean:
    @echo Cleaning up ...
    @rm -f *.o test
```

Uwaga: Jeśli target nie jest plikiem, dodaj przed nim polecenie .PHONY:

```
.PHONY: clean
clean:
    @echo Cleaning up ...
    @rm -f *.o test
```

Ten sam Makefile mógłby wyglądać prościej:


```
CFLAGS=-std=c89 -O3
LIBS=-lm
OBJ=lexer.o   \
    parser.o  \
    codegen.o \
    main.o

.PHONY: all
all: main

.PHONY: clean
clean:
    rm -f *.o test

.c.o:
    $(CC) -c $(INCLUDES) $(CFLAGS) $<

.PHONY: main
main: $(OBJ)
    $(CC) $(OBJ) $(LIBS) -o test

```

To jest bardzo podstawowe wprowadzenie do używania programu make, jednak jest ono wystarczające aby początkujący użytkownik tego udogodnienia zaczął z niego korzystać. Wyczerpujące omówienie całego programu niestety przekracza zakres materiału tej książki.

### Optymalizacje

Kompilator GCC umożliwia generację kodu zoptymalizowanego dla konkretnej architektury. Służą do tego opcje `-march=` i `-mtune=`. Stopień optymalizacji ustalamy za pomocą opcji `-On`, gdzie `n` jest numerem stopnia optymalizacji (od 1 do 3). Możliwe jest też użycie opcji `-Os`, która powoduje generowanie kodu o jak najmniejszym rozmiarze. Aby skompilować dany plik z najwyższym poziomem optymalizacji, trzeba użyć takiego polecenia:

```
gcc program.c -o program -march=natve -O3
```

#### Alignment

Alignment (pol. wyrównanie) jest pewnym zjawiskiem, które opisuje mała część źródeł dotyczących C. Ten podrozdział ma za zadanie wyjaśnienie Ci tego zjawiska oraz uprzedzenie o pewnych faktach, które w późniejszym programowaniu mogą zminimalizować czas stracony na szukaniu informacji o błędnie działającej aplikacji.

Kompilator w ramach optymalizacji wyrównuje elementy struktury tak, aby procesor mógł łatwiej używać zapisanych w niej danych.  Przykładowa struktura:

```
typedef struct {
    unsigned char age; /* 8 bit */
    unsigned char gender; /* 8 bit */
    unsigned int ssn; /* 32 bit */
    unsigned char something_other; /* 8 bit */
} person_t;
```

Aby procesor mógł łatwiej przetworzyć dane kompilator może dodać do tej struktury jedno ośmiobitowe pole. Wtedy struktura będzie wyglądać tak:

```
typedef struct {
    unsigned char fill[1]; /* 8 bit */
    unsigned int ssn; /* 32 bit */
    unsigned char gender; /* 8 bit */
    unsigned char age; /* 8 bit */
    unsigned char something_other; /* 8 bit */
} person_t;
```

Wtedy rozmiar zmiennych przechowujących dane będzie wynosił 64 bity - będzie zatem potęgą liczby dwa i procesorowi dużo łatwiej będzie tak ułożoną strukturę przechowywać w pamięci cache. Jednak taka sytuacja nie zawsze jest pożądana. Może się okazać, że nasza struktura musi odzwierciedlać np. pojedynczy pakiet danych, przesyłanych przez sieć. Nie może być w niej zatem żadnych innych pól, poza tymi, które są istotne do transmisji. Dodając takie coś, można ochronić strukturę:

```
__attribute__ ((packed))
```

Można jeszcze pakować struktury ręcznie, ale nie będę tego przedstawiał w tej książce.

Mając w domu dwa komputery, o odmiennych architekturach (x86 i Sparc / powerpc / mips) może zajść potrzeba stworzenia programu dla jednej maszyny, mając do dyspozycji tylko drugą. Możemy skorzystać z tzw. kompilacji krzyżowej. Polega ona na tym, że program nie jest kompilowany pod procesor, na którym działa kompilator, lecz na inną, zdefiniowaną wcześniej maszynę. Efekt będzie taki sam, a skompilowany program możemy bez problemu uruchomić na drugim komputerze.

Wśród przydatnych narzędzi, których można używać razem z kompilatorem warto wymienić:
 * `objdump`
 * `readelf`
 * GNU `autoconf`
 * GNU `automake`

`objdump` służy do analizy skompilowanych programów, a `readelf` służy do analizy pliku wykonywalnego w formacie ELF. Więcej informacji o tych programach możesz uzyskać, używając `man`:

```
man 1 objdump
man 1 readelf
```

## Przenośność programów

C umożliwia tworzenie programów, które mogą być uruchamiane na różnych platformach sprzętowych pod warunkiem ich powtórnej kompilacji. C należy formalnie do grupy języków wysokiego poziomu (tak na prawdę, to można go określić mianem języka middle-level), które tłumaczone są do asemblera. Z jednej strony jest to korzystne posunięcie, gdyż programy są szybsze i mniejsze niż programy napisane w językach interpretowanych (takich, w których kod źródłowy nie jest kompilowany do kodu maszynowego, tylko na bieżąco uruchamiany przez interpreter). Istnieje niestety druga strona medalu - pewne rzeczy specyficzne dla konkretnego sprzętu, które ograniczają przenośność programów. W tym podrozdziale wyjaśnię ci, jak bez problemu tworzyć poprawne i przenośne programy.

### Undefined Behavior, Implementation-specific behavior

Twórcy C wiedzieli, że wymuszanie jakiegoś konkretnego działania danego wyrażenia byłoby sporym obciążeniem dla osób piszących kompilatory, ponieważ dany wymóg może np. być trudny do zrealizowania na konkretnej architekturze. Dla przykładu, gdyby standard wymagał, że typ int ma rozmiar dokładnie 32 bity to napisanie kompilatora dla architektury, na której bajt ma 5 bitów byłoby cokolwiek kłopotliwe, a z pewnością wynikowy program działałby o wiele wolniej niżby to było możliwe. Z tego powodu, niektóre elementy języka nie są określone bezpośrednio w standardzie i są pozostawione do osoby piszącej konkretną implementację. W ten sposób, nie ma żadnych przeciwwskazań, aby na architekturze, gdzie bajty mają 5 bitów, typ int miał 20 bitów. Wybór powinien być jednak opisany w dokumentacji kompilatora, tak żeby osoba pisząca program w C mogła sprawdzić jak dana konstrukcja zadziała.

Należy pamiętać, że poleganie na jakimś konkretnym działaniu programu w przypadkach zachowania zależnego od implementacji zmniejsza przenośność kodu źródłowego. Undefined Behaviours są o znacznie groźniejsze, gdyż zaistnienie takowego może spowodować dowolny efekt, który często nie jest udokumentowany. Przykładem może tutaj być próba odwołania się do wartości wskazywanej przez wskaźnik o wartości NULL. Na systemie MS-DOS takie coś może zwrócić rzeczywisty wynik, natomiast na systemie Linux takie coś najprawdopodobniej zakończy program sygnałem SIGSEGV (`Segmentation Fault`). Jeżeli gdzieś w programie zaistnieje sytuacja niezdefiniowanego zachowania, to nie jest na 99% to kwestia przenośności, ale błędu w kodzie, chyba że świadomie korzysta się z rozszerzenia kompilatora. Według standardu operacja wyłuskania `NULL` (defererencji operatorem `*`) ma niezdefiniowany rezultat, to w szczególności program może wywołać jakąś z góry określoną funkcję, daną przez kompilator, może on coś takiego zrealizować sprawdzając wartość wskaźnika przed każdą dereferencją. W ten sposób niezdefiniowane zachowanie dla konkretnego kompilatora stanie się jak najbardziej zdefiniowane. Należy jednak na to uważać, bo znacznie zmniejsza to wydajność aplikacji. Przykład jest bardzo prosty - operatory przesunięć bitowych (bitshifts), gdy działają na liczbach ze znakiem. Konkretnie przesuwanie w lewo liczb jest dla niektórych danych wejściowych jest niezdefiniowane. Bardzo często w dokumentacji kompilatora działanie przesunięć bitowych jest dokładnie opisane. Jest to o tyle interesujący fakt, iż wielu programistów nie zdaje sobie z niego sprawy i nieświadomie korzysta z rozszerzeń używanego kompilatora.

Istnieje jeszcze trzecia klasa pochodna od niezdefiniownych zachowań. Zachowania nieokreślone (unspecified behaviour). Są to sytuacje, gdy standard określa kilka sposobów w jaki dane wyrażenie może działać i pozostawia kompilatorowi decyzję co z tym dalej zrobić. Coś takiego nie musi być nigdzie opisane w dokumentacji i znowu poleganie na konkretnym zachowaniu jest w większości przypadków błędem. Klasycznym przykładem może być kolejność obliczania argumentów wywołania funkcji. 

Rozmiar poszczególnych typów danych (np. `int`, `short`, `long`, `char`), różni się pomiędzy platformami, gdyż nie jest definiowany w sztywny sposób (poza typem char, który zawsze zajmuje 1 bajt), lecz w na zasadzie zależności typu `sizeof(long) >= sizeof(int)` lub `sizeof(int) >= sizeof(short)`. Pierwsza standaryzacja zakładała, że `int` będzie miał taki rozmiar, jak domyślna długość liczb całkowitych na danym komputerze, natomiast modyfikatory `short` oraz `long` zmieniały długość tego typu tylko wtedy, gdy dana maszyna obsługiwała typy o mniejszej lub większej długości.

Z tego powodu, nigdy nie zakłada się, że dany typ będzie miał określony rozmiar. Jeżeli potrzebny jest typ o pewnym konkretnym rozmiarze (konkretnej liczbie bitów) można skorzystać z pliku nagłówkowego `<stdint.h>` lub `<inttypes.h>`.

`<stdint.h>` definiuje typy `int8_t`, `int16_t`, `int32_t`, `int64_t`, `uint8_t`, `uint16_t`, `uint32_t` i `uint64_t` (oo ile dana architektura wspiera dane typy).

Podstawową jednostką danych jest bit, który może mieć wartość 0 lub 1. Kilka kolejnych bitów stanowi bajt. Często typ short ma wielkość dwóch bajtów. W jaki sposób te dwa bajty sa zapisane w pamięci, czy na początku jest ten bardziej znaczący (big endian), czy mniej znaczący (little endian)? Skąd takie śmieszne nazwy? Pochodzą one z książki "Podróże Guliwera", w której liliputy kłóciły się o stronę, od której należy rozbijać jajko na twardo. Jedni uważali, że trzeba je rozbijać od grubszego końca (big endian) a drudzy, że od cieńszego (little endian). Sprawa się jeszcze bardziej komplikuje w przypadku typów, które składają się np. z 4 bajtów. Wówczas są aż 24 (4 silnia) sposoby zapisania kolejnych fragmentów takiego typu. W praktyce zapewne spotkasz się jedynie z kolejnościami big endian lub little endian, co nie zmienia faktu, że inne możliwości także istnieją i przy pisaniu programów, które mają być przenośne należy to brać pod uwagę.

Poniższy przykład dobrze obrazuje oba sposoby przechowywania zawartości zmiennych w pamięci komputera (przyjmując `CHAR_BIT == 8` oraz `sizeof(long) == 4`, bez padding bits)

```
unsigned long zmienna = 0x01040503;

Byte #        | 0  | 1  | 2  | 3  |
big    endian |0x01|0x04|0x05|0x03|
little endian |0x03|0x05|0x04|0x01|
```

Czasami program A musi się komunikować z programem B, który działa na komputerze o innym porządku bajtów. Często najprościej jest przesyłać liczby jako tekst, gdyż jest on niezależny od innych czynników, jednak taki format zajmuje więcej miejsca, a nie zawsze możena pozwolić sobie na taką rozrzutność.

Przykładem może być komunikacja sieciowa, w której przyjęło się, że dane przesyłane są w porządku big-endian. Aby móc łatwo operować na takich danych, w standardzie POSIX zdefiniowano następujące funkcje (zazwyczaj są to makra):

```
// W <arpa/inet.h>

uint32_t htonl(uint32_t);
uint16_t htons(uint16_t);
uint32_t ntohl(uint32_t);
uint16_t ntohs(uint16_t);
```

Pierwsze dwie funkcje konwertują liczbę z reprezentacji lokalnej na reprezentację big endian (host -> network), natomiast kolejne dwie odwracają konwersję (network -> host). Można skorzystać z `<endian.h>`, gdzie definiowane są makra pozwalające określić porządek bajtów. Przykład:


```
#include <endian.h>
#include <stdio.h>
 
int main(void) {
    #if __BYTE_ORDER == __BIG_ENDIAN
        puts("Big endian");
    #elif __BYTE_ORDER == __LITTLE_ENDIAN
        puts("Little-endian");
    #elif defined __PDP_ENDIAN && __BYTE_ORDER == __PDP_ENDIAN
        puts("PDP");
    #endif
}
```

Ciągle jednak program polega na niestandardowym pliku nagłówkowym <endian.h>. Można go wyeliminować sprawdzając porządek bajtów w runtime (czasie wykonywania programu):

```
#include <stdio.h>
#include <stdint.h>

int main(void) {
    uint32_t val = 0x03050401;
    unsigned char * v = (unsigned char *)&val;
    int order = *v * 1000 + *(v + 1) * 100 + *(v + 2) * 10 + *(v + 3);
    if (order == 4321)
        printf("Big Endian");
    else if (order == 1234)
        printf("Little Endian");
    else if (order == 3412)
        printf("PDP");
    printf("Other, result: %d (from 1453)\n", order);
    return 0;
}
```

Powyższe przykłady opisują część problemów jakie mogą wynikać z próby przenoszenia danych pomiędzy wieloma platformami. Wszystkie zakładają, że bajt ma 8 bitów, co wcale nie musi być prawdą dla konkretnej docelowej architektury. Co więcej liczby mogą posiadać w swojej reprezentacji padding bits, które nie biorą udziały w przechowywaniu wartości liczby. Te wszystkie różnice mogą dodatkowo skomplikować kod, więc należy być świadomym, iż przenosząc dane musimy uważać na różne reprezentacje liczb.

Pisząc programy nieraz potrzebna będzie pomoc różnych bibliotek. Problem polega na tym, że nie zawsze będą one dostępne na tej platformie, na którym inny użytkownik będzie próbował go kompilować. Dlatego ważne jest, aby korzystać z przenośnych i łatwo dostępnych bibliotek, które dostępne są na wiele różnych systemów i platform sprzętowych.

Windows: funkcje atan2, floor i fabs są w tej samej bibliotece, co standardowe funkcje C.

Unix: osobna biblioteka matematyczna `libm` w wersji statycznej (`/usr/lib/libm.a`) i pliku nagłówkowym math.h, oraz dynamicznej (`/usr/lib/libm.so`).
Aby korzystać z tych funkcji należy należy załączyć plik <math.h>, a przy kompilacji dołączyć bibliotekę libm przełącznikiem `-lm`
Opcja -lm używa libm.so albo libm.a w zależności od tego, która wersja została znaleziona, i w zależności od obecności przełącznika `-static`

### Kompilacja Warunkowa

Przy zwiększaniu przenośności kodu może pomóc preprocessor. Jeśli program korzysta ze słowa kluczowego wprowadzonego w standardzie C99, ale równocześnie powinien się kompilować przez kompilatory C89, można skorzystać z takiego kodu:

```
#ifndef restrict
    #if __STDC_VERSION__ >= 199901L
        #define __restrict__ restrict
    #else
        #define __restrict
    #endif
#endif
```

a w kodzie programu zamiast `restrict` stosować `__restrict__`. GCC rozumie słowa kluczowe tak tworzone i w jego przypadku warto nie redefiniować wartości:

```
#ifndef __GNUC__
    #ifndef __inline__
        #if __STDC_VERSION__ >= 199901L
            #define __inline__ inline
        #else
            #define __inline__
        #endif
    #endif
#endif
```

Korzystając z kompilacji warunkowej można także korzystać z różnego kodu zależnie od m.in. systemu operacyjnego. Przykładowo, przed kompilacją na konkretnej platformie tworzy się odpowiedni plik `config.h`, który następnie dołącza się do wszystkich plików źródłowych, w których podejmuje się decyzje na podstawie zdefiniowanych makr. Przykład:

```
#ifndef CONFIG_H
#define CONFIG_H

/* #define USE_WINDOWS */
/* #define USE_LINUX */

#error Please edit config.h file and remove error lines.
#error Uncomment macros depending on platform used.

#endif
```

Plik źródłowy:

```
#include "config.h"

#ifdef USE_WINDOWS
    /* ... */
#else
    /* ... */
#endif
```

Istnieją różne narzędzia, które pozwalają na automatyczne tworzenie plików pokroju `config.h`, dzięki czemu użytkownik przed skompilowaniem programu nie musi ręcznie edytować pliku konfiguracji, a jedynie uruchomić odpowiednie polecenie. Przykładem jest zestaw autoconf i automake. 

## Łączenie C i innych języków

W przypadku większości języków pokroju Pythona albo Lua, informacje o łączeniu z C możesz znaleźć w dokumentacji. W tym rozdziale opiszę możliwości łączenia C i Assemblera oraz C i C++. Jeśli nie znasz ani C++, ani podstaw Assemblera, możesz spokojnie pominąć ten rozdział i wrócić do niego, jak już go poznasz.

Język C może być z łatwością łączony z innymi językami programowania, które podlegają kompilacji bezpośrednio do kodu maszynowego (Asembler, Fortran, C++). Ponadto dzięki specjalnym bibliotekom można go łączyć z językami bardzo wysokiego poziomu (Python, Ruby).

Łączenie języka C i języka asemblera jest częsty zjawiskiem. Dzięki możliwości połączenia obu tych języków programowania można było utworzyć bibliotekę dla języka C, która niskopoziomowo komunikuje się z jądrem systemu operacyjnego komputera. Ponieważ zarówno asembler jak i C są językami tłumaczonymi do poziomu kodu maszynowego, za ich łączenie odpowiada program zwany linkerem. 

W przykładzie założę, że w pliku test.S zawarty będzie kod, napisany w asemblerze, a test.c to kod z programem w języku C. Program w języku C będzie wykorzystywał jedną funkcję, napisaną w języku asemblera, która wyświetli na ekran klasyczny już napis "Hello, world!". Z powodu ograniczeń technicznych zakładm, że program będzie uruchamiany na systemie operacyjnym bazowanym na Linuksie i platformie i386, skompilowany kompilatorem gcc. Używana składnia asemblera to AT&T. Przykład:

```
    .text
    .globl hello
hello:
    pushl %ebp /* setting up registers */
    movl %esp, %ebp
    movl $4, %eax /* 4 - syscall write */
    movl $1, %ebx /* 1 - stdout */
    movl $msg, %ecx /* address of variable */
    movl $len, %edx /* string length */
    int $0x80 /* make a syscall */
    popl %ebp
    ret

    .data
msg:
    .string "Hello, world!\n"
    len = . - msg
```

I plik .c:

```
extern void hello(void);
int main() {
    hello();
}
```

Kompilacja:

```
as test.S -o testasm.o
gcc test.c -c -o testc.o
gcc testasm.o testc.o -o test
./test
```

=>

```
Hello world
```

Na razie istnieje tylko bardzo prosta funkcja, która w zasadzie nie komunikuje się z C. Nie zwraca żadnej wartości ani nie pobiera argumentów. Jednak, aby zacząć pisać obsługę funkcji, która będzie pobierała argumenty i zwracała wyniki trzeba poznać działanie C od jeszcze niższego poziomu.

C korzysta ze stosu do przekazywania argumentów do funkcji. Argumenty odkładane są w kolejności od ostatniego do pierwszego. Ponadto na końcu odkładany jest tzw. adres powrotu, dzięki któremu po wykonaniu funkcji procesor wie do którego miejsca ma powrócić. Prolog funkcji wygląda tak:

```
pushl %ebp
movl %esp, %ebp
```

Na stosie znajduje się zawartość rejestru EBP, adres powrotu i argumenty od pierwszego do ostatniego.

Na architekturze x86 do zwracania wyników pracy programu używa się rejestru EAX, lub jego cząstek, AX i AH/AL. Zatem aby funkcja napisana w asemblerze zwróciła 0 przed rozkazem ret należy przenieść do rejestru EAX 0:

```
movl $0, %eax
```

Czas na połączenie twojej wiedzy w całość. Poniższy kod zadziała praktycznie pod każdym systemem operacyjnym na x86. Uwaga! Systemy uniksopodobne nie potrzebują underscore! Przykład:

```
 .text
 .globl _myadd
 _myadd:
   pushl %ebp
   movl %esp, %ebp
   movl 8(%esp), %eax /* eax = arg0 */
   addl 12(%esp), %eax /* eax += arg1 */
   popl %ebp
   ret /* ret eax */
```

```
#include <stdio.h>
extern int myadd(int a, int b);
int main(void) {
    printf ("977 + 476 = %d\n", myadd(997, 476));
}
```

Po skompilowaniu i uruchomieniu programu na ekranie powinno pojawić się działanie `977 + 476 = 1453`.

Oprócz możliwości wstępnie skompilowanych modułów możesz posłużyć się także tzw. wstawkami asemblerowymi. Ich użycie powoduje wstawienie w miejsce wystąpienia wstawki odpowiedniego kodu maszynowego, który powstanie po przetłumaczeniu kodu asemblerowego. Ponieważ wstawki asemblerowe nie są standardowym elementem języka C, każdy kompilator ma całkowicie odmienną filozofię ich stosowania. Ponieważ w tym podręczniku skupiam się kompilatorach GNU, omówię filozofię stosowania wstawek asemblera według programistów GNU. Przykład:

```
int main (void) {
    asm ("nop\n");
}
```

W tym wypadku wstawiona zostanie instrukcja "nop" (no operation).

Język C++ z racji swojego podobieństwa do C będzie łatwy do łączenia. Pewnym utrudnieniem może być obiektowość języka C++, przestrzenie nazw oraz możliwość przeciążania funkcji. Oczywiście nadal zakładam, że główny program jest napisany w C. C nie wspiera wszystkich możliwości C++, więc trzeba zmusić go do wyłączenia swoich "zalet" nad C, aby można było połączyć ze sobą elementy programu napisane w C i C++. Używa się takiej konstrukcji:

```
extern "C" {
    /* ... */
}
```

Plik .C:

```
#include <stdio.h>

extern void echo(int a);
 
int main(void) {
    echo(2);
}
```

Plik .CPP:

```
#include <iostream>

extern "C" {
    void echo(int a) {
        std::cout << a << std::endl;
    }
}
```

Kompilacja:

```
gcc test.c -c -o ctest.o
g++ test.cpp -c -o cpptest.o
```

Linkowanie:

```
gcc ctest.o cpptest.o -o test -lstdc++ # Uwaga na libstdc++!
```

Bardzo istotne jest to, żeby nie zapomnieć o `extern "C"`.

## Ćwiczenia

1. Napisz program, obliczający cosinus kąta `π/2` i `4π` (wartości w radianach)
2. Napisz program wczytujący z stdin trzy liczby rzeczywiste, liczący ich średnią i pierwiastek kwadratowy, wypisując najmniejszą z tych liczb.
3. Wczytaj od użytkownika ciąg znaków o **dowolnej** długości i wypisze jego długość.
4. Napisz prosty kalkulator konsolowy.
5. Przygotuj funkcje dodawania, odejmowania, mnożenia i dzielenia w Asemblerze i połącz je z C.
6. Napisz program, który przyjmuje dwa ciągi. Pierwszy to losowy ciąg, a drugi to losowy ciąg z dodanym ciągiem trzech liter w dowolnym miejscu. Program powinien wypisać: dodany znak i indeks w ciągu jego wystąpienia. Przykład:

```
test("hello", "aaahello");
/* stdout:
 * Dodano znak: a
 * Indeks: 0 */
```

7. Napisz program, który przyjmie wyrażenie matematyczne w postaci `2+3*4/5`, mając dany brak spacji między operatorami i wyłącznie jednocyfrowe liczby, oraz podstawowe operatory: `+-*/`, oraz wykona wyrażenie matematyczne i wypisze jego wynik. Kilka test cases: `2+2*2`, `1*2+3-4/5*6+7-8/9`, `2/0` - to powinno wypisać błąd.

**[Powrót do spisu treści](..)**
