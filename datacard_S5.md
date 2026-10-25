# Data Card: MIT-BIH Arrhythmia Database (Mini-Twin S5)

## 1. Descriere Generală și Sursă

* **Nume dataset:** MIT-BIH Arrhythmia Database (v1.0.0)
* **Sursă / Repository:** PhysioNet ([MIT-BIH Arrhythmia Database](https://physionet.org/content/mitdb/1.0.0/))
* **DOI:** [10.13026/C2F305](https://doi.org/10.13026/C2F305)
* **Autori originali:** George B. Moody și Roger G. Mark, în cadrul colaborării dintre MIT și Beth Israel Hospital.
* **Perioadă de colectare:** 1975–1979. Baza de date a fost distribuită inițial în 1980, iar versiunea completă a devenit disponibilă pe PhysioNet în februarie 2005.
* **Obiectul twinuit:** Pacient monitorizat prin Holter, reprezentat prin activitatea electrică a inimii înregistrată prin semnal ECG.
* **Scopul utilizării în S5:** Pregătirea datelor ECG pentru dezvoltarea ulterioară a unui sistem de clasificare a bătăilor cardiace în cinci clase AAMI.

## 2. Specificații Tehnice și Rezoluție

* **Număr de înregistrări:** 48 de înregistrări ECG ambulatorii provenite de la 47 de subiecți.
* **Caracteristici demografice:** 25 de bărbați cu vârste între 32 și 89 de ani și 22 de femei cu vârste între 23 și 89 de ani.
* **Durată per înregistrare:** Aproximativ 30 de minute. Numărul exact de eșantioane depinde de durata fiecărei înregistrări.
* **Canale:** Două derivații ECG per înregistrare:
  * MLII (Modified Lead II) este utilizată în majoritatea înregistrărilor.
  * Al doilea canal poate fi V1, V2, V4 sau V5, în funcție de înregistrare.
* **Rezoluție temporală:** Frecvență de eșantionare de 360 Hz per canal, corespunzătoare unui interval temporal de aproximativ 2,78 ms între eșantioane.
* **Rezoluție de amplitudine:** Digitizare la 11 biți, pe un interval de 10 mV.
* **Unități fizice:** După decodarea semnalului cu biblioteca `wfdb`, amplitudinea ECG este exprimată în milivolți (mV).
* **Adnotări:** Baza originală conține aproximativ 110.000 de adnotări ale bătăilor cardiace. Adnotările au fost revizuite de cardiologi, iar dezacordurile au fost soluționate prin consens.
* **Clase AAMI EC57:** Pentru proiect, etichetele sunt grupate în cinci clase:
  * **N:** Bătăi normale și anumite bătăi de conducere sau de escape.
  * **S:** Bătăi ectopice supraventriculare.
  * **V:** Bătăi ectopice ventriculare.
  * **F:** Bătăi de fuziune între o bătaie normală și una ventriculară.
  * **Q:** Bătăi neclasificabile și anumite bătăi stimulate sau de fuziune cu bătăi stimulate.

## 3. Calitatea Datelor, Goluri și Zgomot

* **Goluri și valori lipsă:** Pipeline-ul verifică valorile nefinite (`NaN` și `Inf`) pentru semnalul MLII din cele 44 de înregistrări selectate. La execuția validată, verificările au trecut. Nu a fost efectuată o analiză exhaustivă a tuturor posibilelor întreruperi temporale sau a artefactelor.
* **Artefacte și zgomot:** Înregistrările ECG ambulatorii pot conține zgomot real de achiziție, inclusiv:
  * Deriva liniei de bază (*baseline wander*), asociată respirației și mișcării.
  * Interferențe de la rețeaua electrică.
  * Artefacte electromiografice (EMG).
  * Artefacte produse de mișcare sau de contactul electrozilor.
* **Limitări ale verificării calității:** Pipeline-ul verifică existența fișierelor, frecvența de eșantionare, canalul selectat și valorile nefinite. Nu efectuează detecția automată a artefactelor și nici cuantificarea nivelului de zgomot.
* **Înregistrări excluse:** Înregistrările 102, 104, 107 și 217 conțin bătăi stimulate de pacemaker și sunt excluse din protocolul de clasificare utilizat. Rămân 44 de înregistrări selectate.

## 4. Împărțire Train / Test și Prevenirea Data Leakage-ului

* **Risc de leakage:** Împărțirea aleatorie a bătăilor la nivel individual poate produce *data leakage* atunci când bătăi de la același pacient ajung atât în train, cât și în test. Caracteristicile morfologice specifice pacientului pot conduce astfel la o evaluare prea optimistă a generalizării pe pacienți noi.
* **Protocol ales:** Split inter-pacient modificat, pornind de la partiționarea De Chazal et al. (2004). Înregistrările 201 și 202 provin de la același subiect și sunt păstrate împreună în train.
* **DS1 — Train:** 23 de înregistrări:

  `101, 106, 108, 109, 112, 114, 115, 116, 118, 119, 122, 124, 201, 202, 203, 205, 207, 208, 209, 215, 220, 223, 230`

* **DS2 — Test:** 21 de înregistrări:

  `100, 103, 105, 111, 113, 117, 121, 123, 200, 210, 212, 213, 214, 219, 221, 222, 228, 231, 232, 233, 234`

* **Procesare realizată de pipeline:**
  * Încărcarea semnalelor ECG și a adnotărilor din fișierele PhysioNet.
  * Selectarea canalului MLII după nume.
  * Extragerea ferestrelor ECG de 200 de eșantioane în jurul adnotărilor eligibile: 90 de eșantioane înaintea poziției R și 110 începând cu poziția R.
  * Maparea simbolurilor adnotărilor în clasele AAMI N, S, V, F și Q.
  * Calcularea intervalelor RR precedente și următoare în secunde.
  * Organizarea datelor în matricele de train și test și verificarea dimensiunilor, etichetelor și valorilor numerice.

* **Rezultatele efective ale pipeline-ului:**

| Indicator | Train | Test |
|---|---:|---:|
| Număr de înregistrări | 23 | 21 |
| Bătăi extrase | 53.111 | 47.534 |
| Clasa N | 47.883 | 42.159 |
| Clasa S | 998 | 1.781 |
| Clasa V | 3.807 | 3.200 |
| Clasa F | 415 | 387 |
| Clasa Q | 8 | 7 |

* **Notă metodologică:** Split-ul implementat este o modificare a partiționării De Chazal originale, care folosește 22 de înregistrări pentru fiecare set. În proiectul S5, înregistrarea 202 a fost mutată din test în train pentru a rămâne împreună cu înregistrarea 201. Prin urmare, split-ul utilizat este de 23/21, nu split-ul canonic 22/22.
* **Limitarea intervalelor RR:** În implementarea curentă, intervalele RR sunt calculate între adnotările ale căror simboluri sunt incluse în maparea AAMI. Adnotările cu simboluri nemapate sunt omise; prin urmare, aceste intervale nu sunt neapărat intervalele dintre toate bătăile consecutive din semnal.
* **Interpretarea numărului de bătăi:** Cele 100.645 de bătăi raportate reprezintă ferestrele extrase de pipeline din cele 44 de înregistrări selectate, nu totalul adnotărilor bazei originale.

## 5. Licență și Limitări

* **Licență:** Open Data Commons Attribution License v1.0 (ODC-By v1.0). Condițiile de reutilizare și atribuire sunt disponibile pe pagina oficială [PhysioNet MIT-BIH](https://physionet.org/content/mitdb/1.0.0/).
* **Citare recomandată:** Moody, G. B., & Mark, R. G. (2005). MIT-BIH Arrhythmia Database (version 1.0.0). PhysioNet. https://doi.org/10.13026/C2F305
* **Limitări pentru Mini-Twin:**
  1. **Dezechilibru de clasă:** Clasa N predomină, în timp ce clasele S și F au mult mai puține exemple. Acest dezechilibru trebuie luat în considerare la evaluarea ulterioară a unui clasificator.
  2. **Număr redus de subiecți:** Baza originală conține 47 de subiecți, ceea ce limitează diversitatea populației reprezentate.
  3. **Vechimea datelor:** Înregistrările au fost colectate între 1975 și 1979; tehnologia de achiziție și caracteristicile populației nu reflectă neapărat practicile clinice actuale.
  4. **Generalizare limitată:** Rezultatele obținute pe MIT-BIH nu garantează aceeași performanță pe alte echipamente, populații sau configurații de derivații ECG.
  5. **Protocol de split modificat:** Împărțirea 23/21 diferă de protocolul original De Chazal. Comparațiile cu studiile publicate trebuie să țină cont de această diferență.

---

## 6. Tabel State of the Art (SOA) — 10 Articole Relevante

| # | Articol (Autor, An) | Metodă / Arhitectură | Date & Protocol Split | Metrică Principală | Rezultat Raportat |
|---|---|---|---|---|---|
| 1 | [**De Chazal et al. (2004)**](https://doi.org/10.1109/TBME.2004.827359) | Caracteristici de morfologie ECG și intervale RR; clasificator statistic supervizat | MIT-BIH, 44 de înregistrări non-pacemaker; inter-pacient, DS1/DS2, 22/22 | Sensibilitate (Se), predictivitate pozitivă (+P) și rata fals pozitive | SVEB: Se 75,9%, +P 38,5%; VEB: Se 77,7%, +P 81,9% |
| 2 | [**Kachuee et al. (2018)**](https://doi.org/10.1109/ICHI.2018.00092) *(Ancoră 1)* | CNN profund pentru clasificarea bătăilor și reprezentare transferabilă | MIT-BIH pentru aritmii și PTB Diagnostic Database pentru infarct miocardic; protocolul autorilor | Acuratețe (Acc) | Acc 93,4% pentru clasificarea aritmiilor; 95,9% pentru clasificarea infarctului miocardic pe PTB |
| 3 | [**Hannun et al. (2019)**](https://doi.org/10.1038/s41591-018-0268-3) *(Ancoră 2)* | Rețea neuronală profundă end-to-end pentru clasificarea ritmurilor ECG | 91.232 de ECG-uri ambulatorii cu o singură derivație, de la 53.549 de pacienți; test independent cu adnotări realizate prin consens de cardiologi | ROC AUC și F1-score | AUC mediu 0,97; F1 0,837 pentru model față de 0,780 pentru cardiologii evaluați |
| 4 | [**Rajpurkar et al. (2017)**](https://arxiv.org/abs/1707.01836) | CNN cu 34 de straturi pentru clasificarea ECG | ECG-uri ambulatorii cu o singură derivație, 30 s, 200 Hz; 64.121 de înregistrări de antrenare de la 29.163 de pacienți și set de test separat | F1-score | F1 0,809 pe setul de evaluare raportat de autori |
| 5 | [**Kiranyaz et al. (2016)**](https://doi.org/10.1109/TBME.2015.2468589) | CNN 1D adaptiv, specific pacientului (*patient-specific*) | MIT-BIH, clasificare specifică pacientului | Sensibilitate (Se) și specificitate (Sp) pentru SVEB/VEB | VEB: Se 93,9%, Sp 98,9%; SVEB: Se 60,3%, Sp 99,2%. Rezultatele sunt specifice protocolului patient-specific |
| 6 | [**Acharya et al. (2017)**](https://doi.org/10.1016/j.compbiomed.2017.08.022) | CNN cu 9 straturi și augmentare pentru echilibrarea claselor | MIT-BIH, cinci clase AAMI; seturi augmentate, cu și fără atenuarea zgomotului | Acuratețe (Acc) | După augmentare: Acc 94,03% pe semnalul cu zgomot și 93,47% pe setul fără zgomot; pe datele originale dezechilibrate: 89,07% și, respectiv, 89,30% |
| 7 | [**Mousavi & Afghah (2019)**](https://doi.org/10.1109/ICASSP.2019.8683140) | CNN și model de învățare secvență-la-secvență (*sequence-to-sequence*) | MIT-BIH, cinci clase AAMI; evaluate atât paradigme intra-pacient, cât și inter-pacient | Se și +P pentru SVEB/VEB | Inter-pacient: SVEB Se 88,94%, +P 92,57%; VEB Se 99,94%, +P 99,50% |
| 8 | [**Niu et al. (2020)**](https://doi.org/10.1109/JBHI.2019.2942938) | Reprezentare simbolică a morfologiei și ritmului, combinată cu un CNN multi-perspectivă | MIT-BIH; clasificare inter-pacient a SVEB și VEB | Acuratețe globală și F1 pe SVEB/VEB | Acc 96,4%; F1 SVEB 76,6%; F1 VEB 89,7% |
| 9 | [**Ribeiro et al. (2020)**](https://doi.org/10.1038/s41467-020-15432-4) | Rețea neuronală profundă pentru clasificarea ECG-urilor cu 12 derivații | Baza clinică CODE, 2.322.513 ECG-uri de la 1.676.384 de pacienți; evaluare pe un set de test separat de 827 ECG-uri | F1-score și specificitate pentru șase anomalii | F1 peste 80% și specificitate peste 99% pentru anomaliile evaluate |
| 10 | [**Sellami & Hwang (2019)**](https://doi.org/10.1016/j.eswa.2018.12.037) | CNN profund cu *batch-weighted loss* pentru dezechilibrul claselor, folosind ECG brut cu o singură derivație | MIT-BIH; evaluări intra-pacient și inter-pacient | Acc, Se, Sp și predictivitate pozitivă (+P) | În rezultatele raportate pentru evaluarea inter-pacient: Acc 88,34%, Se 90,90%, Sp 88,51% și +P 48,25% |

**Notă privind comparabilitatea:** Rezultatele sunt preluate din protocoalele originale ale autorilor. Studiile utilizează seturi de date, clase, împărțiri și metrici diferite; valorile nu reprezintă o comparație directă între toate metodele. Split-ul implementat în S5 este o modificare față de împărțirea originală De Chazal și trebuie descris ca atare.