# Linkare dinamică

Linkarea dinamică înseamnă că în executabil nu sunt incluse componentele folosite din bibliotecă.
Acestea vor fi incluse mai târziu, la încărcare (*load time*) sau chiar la rulare (*runtime).
În urma linkării dinamice, executabilul reține referințe la bibliotecile folosite și la simbolurile folosite din cadrul acestora.
Aceste referințe sunt similare unor simboluri nedefinite.
Rezolvarea acestor simboluri are loc mai târziu, prin folosirea unui loader / linker dinamic.

Așadar, în cazul linkării dinamice, aspecte precum rezolvarea simbolurilor sau stabilirea adreselor nu sunt efectuate pentru simbolurile bibliotecilor.

În directorul `06-dynamic/` avem un conținut similar directorului `05-static/`.
Diferența este că acum, folosim linkare dinamică în loc de linkare statică pentru biblioteca standard C.
Pentru aceasta, am renunțat la argumentul `-static` folosit la linkare.

Pentru acest exemplu, obținem un singur executabil `main`, din legarea statică cu biblioteca `libinc.a` și legarea dinamică cu biblioteca standard C.
Similar exemplului din directorul `05-static/, folosim comanda `make` pentru a obține executabilul `main`:



Fișierul executabil `main` obținut prin linkare dinamică are un comportament identic fișierului executabil `main` obținut prin linkare statică.
Observăm că dimensiunea sa este mult mai redusă: ocupă `7 KB` comparativ cu `600 KB` cât avea varianta sa statică.
De asemenea, folosind utilitarul `file`, aflăm că este executabil obținut prin linkare dinamică (*dynamically linked*), în vreme cel obținut în exemplul anterior este executabil obținut prin linkare statică (*statically linked).

Investigăm simbolurile executabilului:


Simbolurile obținute din modulul obiect `main.o` și din biblioteca statică `libinc.o` sunt rezolvate și au adrese stabilite.
Observăm că folosirea bibliotecii standard C a dus la existența simboblului `_start`, care este entry pointul programului.
Dar, simbolurile din biblioteca standard C, (`printf`, __libc_start_main`) sunt marcate ca nedefinite (`U`).
Aceste simboluri nu sunt prezente în executabil: rezolvarea, stabilirea adreselor și relocarea lor se va realiza mai târziu, la încărcare (load time).

La încărcare, o altă componentă software a sistemului, loaderul / linkerul dinamic, se va ocupa de:

- localizarea în sistemul de fișiere a fișierelor bibliotecă dinamice care sunt folosite de fișierul executabil încărcat
- încărcarea în memorie a acelor biblioteci dinamice, lucru care duce și la stabilirea adreselor simbolurilor din bibliotecă
- parcurgerea simbolurilor nedefinite din cadrul fișierului executabil, localizarea lor în biblioteca înacarcată dinamic și relocarea lor în executabilul încărcat în memorie

Putem investiga bibliotecile dinamice folosite de un executabil prin intermediul utilitarului `ldd`:



În rezultatul de mai sus, observăm că executabilul folosește biblioteca standard C, localizată la calea `/lib/i386-linux-gnu/libc.so.6`.
`/lib/ld-linux.so.2` este loaderul / linkerul dinamic.
`linux-gate.so.1` e o componentă specifică Linux pe care nu vom insista.

Pe lângă dimensiunea redusă a executabilelor, marele avantaj al folosirii linkării dinamice, este că se pot partaja secțiunile de cod (nu de date) ale bibliotecilor dinamice.
Când un executabil dinamic este încărcat, se identifică bibliotecile dinamice de care acesta depinde.
Dacă o bibliotecă dinamică deja există în memorie, se face referire direct la zona existentă, partajând astfel biblioteca dinamică.
Acest lucru conduce la o reducere semnificativă a memoriei ocupate de aplicațiile sistemului.
10 aplicații care folosesc, probabil toate, biblioteca standard C, vor partaja codul bibliotecii.

Din acest motiv, bibliotecile dinamice mai sunt numite și obiecte partajate (*shared objects*).
De aici este, în Linux, extensia `.so` a fișierelor de tip bibliotecă partajată.

## Biblioteci cu linkare dinamică

Numele corect al unei biblioteci dinamice este bibliotecă cu linkare dinamică (*dynamically linked library*) sau bibliotecă partajată.
În Windows, bibliotecile dinamice sunt numite *dynamic-link libraries* de unde și extensia `.dll`.

Din punctul de vedere al comenzii folosite, nu diferă linkarea unei biblioteci dinamice sau a unei biblioci statice.
Diferă executabilul obținut, care va avea nedefinite simbolurile folosite din bibliotecile dinamice.
De asemenea, loaderul / linkerul dinamic trebuie să fie informat de locul bibliotecii dinamice.

În directorul `07-dynlib/` avem un conținut similar directorului `06-dynamic/`.
Diferența este că acum, folosim linkare dinamică în loc de linkare statică și pentru a include funcționalitatea `inc.c`, nu doar pentru biblioteca standard C.
Pentru aceasta, construim fișierul bibliotecă partajată `libinc.so`, în locul fișierului bibliotecă statică `libibc.a`.

Similar exemplului din directorul `06-dynamic/`, folosim comanda `make` pentru a obține executabilul `main`:



Executabilul obținut are dimensiunea în jur de `7 KB` puțin mai mică decât a executabilului din exemplul anterior.
Diferența cea mai mare este că, acum, simbolurile din biblioteca `libinc.so` (`increment`, `init`, `print`, `read`) sunt nerezolvate.

Dacă încercăm lansarea în execuție a executabilului, observăm că primim o eroare:


Variabila de mediu `LD_LIBRARY_PATH` pentru loader este echivalentul opțiunii `-L` în comanda de linkare: precizează directoarele în care să fie căutate biblioteci pentru a fi încărcate, respectiv linkate.
Folosirea variabilei de mediu `LD_LIBRARY_PATH` este recomandată pentru teste.
Pentru o folosire robustă, există alte mijloace de precizare a căilor de căutare a bibliotecilor partajate, documentate în (pagina de manual a loaderului / linkerului dinamic)(https://man7.org/linux/man-pages/man8/ld.so.8.html#DESCRIPTION).
