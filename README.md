Projekat iz Autoelektronike
Tema: Kontrola sigurnosnih pojaseva u automobilu.

Zadatak projekta:

Napisati softver za simulaciju pracenja sigurnosnih pojaseva u automobilu. 
Za to koristiti minimum 4 taska
Prvi task za prijem podataka sa senzora
Drugi task za slanje i primanje podataka sa PC-a
Treći task za obradu podataka
Četvrti task za ispis na displeju
Trenutno stanje senzora simulirati pomoću UniCom
Informacije o trenutnom stanju senzora za vozača i suvozača dobijaju preko UniCom softvera svakih 100 ms na kanalu 0
Komunikaciju sa PC-ijem ostvariti preko simulatora serijske veze, ali na kanalu 1
Za simulaciju displeja koristiti Seg7Mux, a za simulaciju logičih ulaza i izlaza koristiti LED_bar. 
Za vozačevo i suvozačevo mesto postoji digitalni senzor koji daje 1 ako su vezani i 0 ako su odvezani.
Za suvozačevo mesto postoji i analogni senzor koji detektuje pritisak u rasponu od 0-1023 odabiraka ako je pritisak veći od 400 računa da postoji suvozač.
Preko simularne seriske komunikacije treba slati naredbe start+ stop+ i prag+ koje pokreću,zaustavljaju ili ipisuju vrednost sezora ovog sistema.
Podesiti dva stubca ledovki jedan kao ulazni a dugi kao izlazni
Podesiti sistem da može da se upali pritiskom na najnižu diodu levok stuba.
Kada je sistem uključen potrebno je jednu izlaznu diodu na LED baru koristiti kao LED_sistem_aktivan znak.(u našem slučaju donja desna)
Ako je sistem uključen, a nije vezan pojas, potrebno je da preostale izlazne LED diode blink-aju svakih 2 sekunde. 
A ako pojas nije vezan duže od 20 sekundi, sve diode treba da blinkaju svakih 200ms.
Na LCD displeju prikazati trenutno stanje senzora pojasa  (Left/Right ili Both)
Tu informaciju slati i preko serijske komunikacije na 100ms, dok se ne veže pojas. 

Realizacija i simulacija.


Prvi način za pokretanje sistema.
Sistem pokrećemo slanjem naredbe start+ na kanal1 seriske komunikacije.
Pri pokretanju sistema pali se dioda u donjem desnom uglu i oba putnika su odvezana što se i vidi na displeju.
Vrednost praga pri pokretanju sistema je 400 a to možemo menjati slanjem naredbe (npr. prag 500+)na kanali 1 seriske komunikacije
Na kanalu 0 serijske komunikacije, koriscenjem opcije auto i slanjem trigger signala 'T', na svakih 100ms primamo informaciju o vrednosti senzora koje dobijamo iz
stringa u obliku npr 1 1 700+
Komandom (1 1 700+) sistem prepoznaje da vozač vezan , da je suvozač vezan dok treći broj označava vrednost analognog senzora.
Ako je vrednos treće cifre veća od praga suvozač je prisutan a ako nije onda je vozač sam u autu.
Ukoliko je bar jedan od putnika odvezan, izlazni LED stubac treba da treperi na svake dve sekunde u trajanju od 20s. 
Ukoliko se nakon 20 sekundi ne vežu oba putnika(ili samo vozač, ukoliko je samo on prisutan), frekvencija će se povecati, tj treperenje će biti na 200ms.
Sve vreme na display-u i kanalu 1 serijske komunikacije dobijamo poruke:
Both - oba putnika odvezana
Left - odvezan vozac
Right - odvezan suvozac
Ili je prazan display, i bez slanja na serijsku ukoliko su oba zavezana, ili samo vozac(ukoliko nema suvozaca u kolima)
Sistem zaustavljamo slanjem poruke stop+ ili gasenjem diode u donjem levom uglu.

Drugi način za pokretanje sistema:
Takođe sistem možemo pokrenuti pritiskom na dolju levu diodu.
Ako želimo da sistem zaustavi jednostavno isključimo donju levu diodu.


U prilogu se nalaze slike koje pokazuju rad sistema
Slika1 Pokrećemo sistem pomoću komande (start+)(niko nije vezan)
Slika2 Ovde simuliramo prisustvo vozača koji je vezan i suvozača koji je vezan kao i vrednost analognog senzora koji je iznad praga(1 1 600+)
Slika3 Ovde simuliramo prisustvo vozača koji je vezan  kao i vrednost analognog senzora koji je ispod praga(1 1 300+)(nema suvozača)
Slika4 Ovde simuliramo prisustvo vozača koji je vezan i suvozača koji nije vezan
Slika5 Ovde simuliramo prisustvo vozača koji nije vezan i suvozača koji jeste vezan
Slika6 Slanjem komande prag+ na kanalu 1 seriske komunikacije može se videti da li su putnici vezani i koliki je prag prisustva putnika u vozilu

*** Napomena (diode su podešene da blinkanja kao što je traženo u zadatku)***

SerialReceive0_Task

Ovaj task ima za zadatak da obradi podatke koji stižu sa kanala 0 serijske komunikacije. 
Podatak koji stiže je u vidu poruke npr. 1 1 600+. To je podatak o koji govori da li su putnici vezani i trenurno stanje analognog senzora
Karakteri se smeštaju u niz koji se smješta u red, kako bi ostali taskovi taj podatak imali na raspolaganju.
 Ovaj task "čeka" semafor koji će da pošalje interrupt svaki put kada neki karakter stigne na kanal 0.

OnLED_ChangeInterrupt

Ovaj task nam sluzi iskljucivo za proveru da li je pritisnut skroz donji taster prvog stubca.
 Ukoliko jeste palimo skroz donju diodu drugog stubca kao indikaciju da je sistem upaljen, 
postavljano promeljivu start na 1 i saljemo to u queue u koji se skladiste start/stop komande. 
Ukoliko nije pritisnut, ili ga je korisnik iskljucio(prebacio u drugo stanje),
 gasimo prethodno upaljenu diodu, start postavljamo na 0 i to saljemo u isti queue.

start(void* pvParameters)

Ispituje pristigli string da li sadrži kod za aktivaviju ili gašenje sistema.

SerialReceive1_Task

Ovaj task ima za zadatak da obradi podatke koji stižu sa kanala 1 serijske komunikacije. 
Naredbe koje stižu su formata start+, stop+, prag+. Task takođe kao i prethodni čeka odgovarajući semafor da bi se odblokirao i izvršio. 
Taj semafor daje interrupt svaki put kada pristigne karakter na kanal 1.

obrada_senzora(void* pvParameters)

U ovom tasku se prvo detektuje o kom prijemu se radi. 
Nakon toga obradjuje podatke koji se dobijaju sa SerialReceive0_Taska, SerialReceive1_Taska ili LED_bar_Taska.

SerialSend_Task(void* pvParameters)

Ovaj task ima za zadatak da šalje trenutnu vrednost stanja pojaseva u autu na kanal 1 serijske komunikacije svakih 1s.
Task šalje samo pod uslovom da postoji putnik koji nije vezan , 
Uveden je brojač, kako bi se osiguralo da svaki naredni put šalje naredni karakter iz stringa str.




