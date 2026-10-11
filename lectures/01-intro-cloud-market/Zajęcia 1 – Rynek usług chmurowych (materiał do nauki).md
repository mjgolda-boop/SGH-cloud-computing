# Zajęcia 1 – Rynek usług przetwarzania w chmurze

Materiał do samodzielnej nauki · Cloud Computing, SGH ·&#32;

## Co będziesz umieć po tej lekturze

Chmura to nie „cudzy komputer”. To sposób kupowania mocy obliczeniowej tak, jak kupujemy prąd: bierzesz tyle, ile potrzebujesz, i płacisz za to, co zużyłeś. To zdanie warto zapamiętać, bo będzie wracać przez cały semestr.

Po przeczytaniu tego materiału powinieneś umieć:

1. wyjaśnić koledze z innego kierunku, czym chmura różni się od zwykłej serwerowni, posługując się pięcioma cechami z definicji NIST;
2. rozpoznać, czy dana usługa to IaaS, PaaS czy SaaS, i powiedzieć, za co w każdym z tych modeli odpowiadasz Ty, a za co dostawca;
3. policzyć z grubsza, kiedy chmura się opłaca, a kiedy własny sprzęt wychodzi taniej;
4. opisać pozycję AWS, Microsoft Azure i Google Cloud na rynku, z aktualnymi liczbami, i dopasować do siebie ich podstawowe usługi.

Ten dokument powtarza treść zajęć 1 i uzupełnia ją o dane, do których warto wrócić przed kolokwium. Pytania kontrolne znajdziesz na końcu.

## Czym jest chmura obliczeniowa

Najczęściej cytowaną definicję podał amerykański NIST w 2011 roku ([SP 800-145](https://csrc.nist.gov/pubs/sp/800/145/final)). Chmura to model, w którym współdzieloną pulę zasobów (serwery, dyski, sieć, aplikacje) uruchamiasz na żądanie, szybko i przy minimalnym kontakcie z dostawcą. Brzmi abstrakcyjnie, więc pomaga analogia z prądem.

Na początku XX wieku fabryki miały własne generatory. Potem pojawiła się sieć energetyczna i nikt rozsądny nie budował już elektrowni przy hali produkcyjnej. Z serwerami dzieje się dziś to samo: firma nie musi ich kupować, żeby z nich korzystać.

NIST wymienia pięć cech, które usługa musi mieć, żeby nazwać ją chmurą:

| Cecha (NIST) | Co to znaczy w praktyce | Odpowiednik z sieci energetycznej |
| --- | --- | --- |
| Samoobsługa na żądanie | Klikasz w konsoli albo wpisujesz polecenie i po minucie masz serwer, bez maila do działu IT | Włączasz czajnik bez dzwonienia do elektrowni |
| Szeroki dostęp sieciowy | Zasoby są dostępne przez internet z laptopa, telefonu, skryptu | Gniazdko jest w każdym pokoju |
| Współdzielenie zasobów | Ten sam fizyczny serwer obsługuje wielu klientów, którzy o sobie nie wiedzą | Jedna elektrownia zasila całe miasto |
| Szybka elastyczność | Dziś 2 serwery, w Black Friday 200, jutro znów 2 | Zimą zużywasz więcej, latem mniej |
| Mierzalność usługi | Płacisz za sekundy, gigabajty i zapytania | Licznik prądu |

Test jest prosty: jeśli brakuje którejś z tych cech, to raczej hosting albo wynajem serwera niż chmura. Serwer dzierżawiony na trzy lata, zamawiany przez handlowca i rozliczany ryczałtem, nie spełnia warunków elastyczności ani mierzalności.

## IaaS, PaaS, SaaS: kto za co odpowiada

Trzy modele usług różnią się jednym: tym, gdzie przebiega granica między Twoją pracą a pracą dostawcy. Im wyżej w stosie, tym mniej masz do zrobienia i tym mniej możesz zmienić.

Najprościej pokazać to na pizzy. Możesz upiec ją w domu (własna serwerownia), kupić mrożoną i wrzucić do swojego piekarnika (IaaS), zamówić z dostawą i podać na własnym stole (PaaS) albo pójść do pizzerii (SaaS). Za każdym razem jesz pizzę, ale coraz mniej rzeczy zależy od Ciebie.

| Warstwa | Własna serwerownia | IaaS | PaaS | SaaS |
| --- | --- | --- | --- | --- |
| Dane i ich klasyfikacja | Ty | Ty | Ty | Ty |
| Aplikacja i jej kod | Ty | Ty | Ty | Dostawca |
| Środowisko uruchomieniowe, baza danych | Ty | Ty | Dostawca | Dostawca |
| System operacyjny, łatki | Ty | Ty | Dostawca | Dostawca |
| Wirtualizacja, serwery, dyski, sieć | Ty | Dostawca | Dostawca | Dostawca |
| Budynek, prąd, chłodzenie | Ty | Dostawca | Dostawca | Dostawca |
| Przykład | Szafa rack w biurze | Amazon EC2, Azure Virtual Machines, Google Compute Engine | Amazon RDS, Azure SQL Database, Google Cloud SQL | Microsoft 365, Gmail, Salesforce |

Zwróć uwagę na pierwszy wiersz. Za dane i za to, kto ma do nich dostęp, odpowiadasz zawsze, nawet w SaaS. To jest sedno **modelu współdzielonej odpowiedzialności** (shared responsibility model), który wszyscy trzej dostawcy opisują w swojej dokumentacji. Wycieki z chmury rzadko wynikają z włamania do centrum danych Amazona. Zwykle ktoś zostawił publicznie dostępny zasobnik S3 albo hasło w repozytorium.

Na tym przedmiocie zajmujemy się głównie IaaS (zajęcia 3–9) i PaaS (zajęcia 10–13). SaaS jest tłem: to model, który znasz jako użytkownik.

## Gdzie stoi chmura: modele wdrożenia

Model usługi mówi, *co* kupujesz. Model wdrożenia mówi, *czyje* to jest i kto jeszcze z tego korzysta.

- **Chmura publiczna.** Infrastruktura dostawcy, współdzielona przez tysiące klientów. AWS, Azure i GCP to chmury publiczne i na nich pracujemy w tym semestrze.
- **Chmura prywatna.** Te same mechanizmy (samoobsługa, elastyczność, rozliczanie), ale tylko dla jednej organizacji, często we własnym centrum danych, np. na OpenStacku albo VMware.
- **Chmura hybrydowa.** Część systemów w chmurze publicznej, część u siebie, połączone tak, żeby działały jak jedna całość. Typowy przykład: bank trzyma rdzeń systemu transakcyjnego lokalnie, a analitykę i aplikację mobilną w chmurze. Dostawcy mają na to osobne produkty: AWS Outposts, Azure Arc, Google Distributed Cloud.
- **Multi-cloud.** Korzystanie z więcej niż jednego dostawcy chmury publicznej. Bywa świadomą strategią (mniejsze uzależnienie, lepsza pozycja w negocjacjach), a bywa przypadkiem, bo jeden dział wybrał Azure, a drugi GCP.

NIST wymienia jeszcze chmurę wspólnotową (community cloud), współdzieloną przez kilka organizacji o podobnych wymaganiach, np. administrację publiczną. W praktyce rynkowej pojawia się rzadziej.

Warto pamiętać, że multi-cloud ma swoją cenę. Każda chmura ma inny model uprawnień, inne narzędzia i inne nazwy usług, a przesyłanie danych między dostawcami kosztuje. Właśnie dlatego na zajęciach 7 omówimy Terraform i Kubernetes, czyli narzędzia, które tę różnorodność częściowo wygładzają.

## Ekonomia chmury: dlaczego firmy w ogóle to kupują

Chmura zamienia wydatki inwestycyjne na koszty operacyjne. Zamiast kupić serwery za milion złotych i amortyzować je przez pięć lat (**CAPEX**), płacisz co miesiąc za faktyczne zużycie (**OPEX**). Dla dyrektora finansowego to zmiana sposobu myślenia, nie tylko pozycji w bilansie.

Za tym idą trzy konkretne korzyści.

**Nie musisz zgadywać przyszłości.** Kupując serwery, planujesz pod szczyt. Sklep internetowy, który w grudniu ma dziesięć razy większy ruch niż w lutym, przez jedenaście miesięcy utrzymuje sprzęt, który stoi bezczynnie. W chmurze dokładasz maszyny na grudzień i oddajesz je w styczniu.

**Porażka jest tania.** Eksperyment, który nie wypalił, kosztuje kilkadziesiąt dolarów za godziny pracy maszyn, a nie szafę sprzętu, którego nikt już nie potrzebuje. To dlatego startupy prawie zawsze zaczynają w chmurze.

**Efekt skali.** AWS, Microsoft i Google kupują serwery i energię w ilościach, do których żadna polska firma nie ma dostępu, i część tej przewagi oddają klientom w cenie.

### Kiedy chmura się nie opłaca

To jest ważne, bo łatwo wyjść z zajęć z przekonaniem, że chmura zawsze wygrywa. Tak nie jest.

Weź dwie firmy. Pierwsza to sklep internetowy z ruchem skaczącym między 5% a 100% mocy. Druga to firma, która przez całą dobę, przez cały rok, liczy ten sam model ryzyka na stałym obciążeniu 90%. Obie robią obliczenia, obie mogłyby iść do chmury. Ale druga nie korzysta z elastyczności, za którą w chmurze płaci w cenie każdej godziny. Przy stałym, przewidywalnym obciążeniu własny sprzęt (albo kolokacja) potrafi wyjść wyraźnie taniej w perspektywie kilku lat.

Do tego dochodzą koszty, o których się zapomina:

- **transfer danych na zewnątrz** (egress): wgranie danych do chmury jest zwykle darmowe, pobranie ich z powrotem już nie;
- **zasoby zapomniane**: maszyna uruchomiona na zajęciach i niewyłączona nalicza opłaty przez cały miesiąc;
- **kompetencje**: zespół, który zna jedną chmurę, często wykorzysta ją lepiej niż teoretycznie tańszą platformę, której nie zna.

Wniosek na kolokwium: chmura opłaca się tam, gdzie obciążenie jest zmienne albo trudne do przewidzenia. O tym, jak liczyć całkowity koszt posiadania (TCO) i jak obniżać rachunki, mówimy szczegółowo na zajęciach 14.

## Rynek: AWS, Microsoft Azure i Google Cloud

W II kwartale 2026 roku firmy wydały na usługi infrastruktury chmurowej 143,4 mld dolarów, o 43% więcej niż rok wcześniej. To najszybszy wzrost od ośmiu lat, a Synergy Research przypisuje go przede wszystkim generatywnej AI ([Synergy, 30.07.2026](https://www.srgresearch.com/articles/q2-cloud-market-passes-143-billion-highest-growth-rate-in-eight-years)). Trzej najwięksi dostawcy mają razem około 63% tego rynku.

&#91;embedded content: Synergy Research Group: Q2 2024 (via TechCrunch), Q2 2025 i Q2 2026 (komunikaty z 31.07.2025 i 30.07.2026)\]

Z wykresu łatwo wyciągnąć zły wniosek, że AWS słabnie. Tymczasem przychody AWS w tym samym kwartale wzrosły o 37%, do 42,2 mld dolarów ([Amazon, wyniki Q2 2026](https://www.sec.gov/Archives/edgar/data/0001018724/000101872426000024/amzn-20260630xex991.htm)). Udział spada, bo cały rynek rośnie szybciej (43%), a Google Cloud jeszcze szybciej (82% według [Alphabet](https://www.sec.gov/Archives/edgar/data/0001652044/000165204426000066/googexhibit991q22026.htm)). Udział w rynku i przychody to dwie różne rzeczy.

Dwie pułapki przy czytaniu takich danych. Spadek z 30% do 28% to spadek o 2 **punkty procentowe**, a nie o 2%. Poza tym różni analitycy mierzą różne rynki: Gartner, licząc wyłącznie publiczne IaaS za 2024 rok, przypisuje AWS 37,7%, Microsoftowi 23,9%, a Google 9,0% ([Gartner, 6.08.2025](https://www.gartner.com/en/newsroom/press-releases/2025-08-06-gartner-says-worldwide-iaas-public-cloud-services-market-grew-22-point-5-percent-in-2024)). Liczb z dwóch źródeł nie wolno porównywać ani uśredniać.

### Trzech graczy, trzy historie

**AWS** wystartował w 2006 roku usługami S3 i EC2 i przez kilka lat był praktycznie sam na rynku. Ta przewaga czasu przełożyła się na najszerszy katalog usług i największą liczbę inżynierów z doświadczeniem w AWS. Uczelnie, w tym SGH, mają też dostęp do programu AWS Academy.

**Microsoft Azure** ruszył komercyjnie w 2010 roku. Jego siłą jest to, co firmy już mają: Windows Server, Microsoft 365, Active Directory (dziś Microsoft Entra ID). Dla przedsiębiorstwa zbudowanego na produktach Microsoftu Azure jest naturalnym przedłużeniem, szczególnie w modelu hybrydowym (Azure Arc).

**Google Cloud** zaczął od platformy App Engine w 2008 roku, a maszyny wirtualne Compute Engine udostępnił w 2012. Google wnosi doświadczenie z własnej infrastruktury: z niej wyrosły Kubernetes i BigQuery. Mocne strony to analityka danych, kontenery i AI na własnych układach TPU.

### Te same klocki pod różnymi nazwami

Każdy z trzech dostawców ma odpowiednik prawie każdej usługi konkurencji. Różnią się głębią, integracją i ceną, a nie samym istnieniem usługi. Tabela pokazuje odpowiedniki, które spotkasz na kolejnych zajęciach.

| Kategoria | AWS | Microsoft Azure | Google Cloud | Zajęcia |
| --- | --- | --- | --- | --- |
| Maszyny wirtualne (IaaS) | Amazon EC2 | Azure Virtual Machines | Compute Engine | 3, 4 |
| Tożsamość i dostęp | AWS IAM | Microsoft Entra ID | Cloud IAM | 5 |
| Dyski blokowe | Amazon EBS | Azure Disk Storage | Persistent Disk | 6 |
| Infrastruktura jako kod | CloudFormation | ARM / Bicep | Deployment Manager (oraz Terraform dla wszystkich) | 7 |
| Kubernetes zarządzany | Amazon EKS | Azure AKS | Google GKE | 7 |
| Magazyn obiektów | Amazon S3 | Azure Blob Storage | Cloud Storage | 8 |
| Klastry HPC | AWS ParallelCluster | Azure CycleCloud / Batch | Cluster Toolkit | 9 |
| Relacyjne bazy danych (PaaS) | Amazon RDS | Azure SQL Database | Cloud SQL | 10 |
| Hurtownia danych | Amazon Redshift | Azure Synapse Analytics | BigQuery | 11 |
| NoSQL | Amazon DynamoDB | Azure Cosmos DB | Firestore / Bigtable | 12 |
| Hadoop i Spark | Amazon EMR | Azure HDInsight / Databricks | Dataproc | 13 |
| Kontrola kosztów | AWS Cost Explorer | Azure Cost Management | Cloud Billing | 14 |

Nazwy usług AI zmieniają się najszybciej. Vertex AI w kwietniu 2026 stał się częścią Gemini Enterprise Agent Platform, a platforma AI Microsoftu nazywa się obecnie Microsoft Foundry. Przed zajęciami 4 sprawdź aktualne nazwy w dokumentacji.

## Fizyczna strona chmury: regiony i strefy

Chmura ma konkretny adres. Każdy dostawca dzieli świat na **regiony** (np. Frankfurt, Warszawa, Wirginia), a region na **strefy dostępności** (availability zones). Strefa to jedno lub kilka centrów danych z niezależnym zasilaniem, chłodzeniem i siecią, oddalone od pozostałych na tyle, żeby pożar czy powódź nie objęły wszystkich naraz.

Dla Ciebie wynikają z tego dwie praktyczne zasady. Po pierwsze, system, który ma przetrwać awarię, uruchamiasz w co najmniej dwóch strefach. Po drugie, wybór regionu decyduje o tym, gdzie fizycznie leżą dane, a to ma znaczenie prawne (RODO), kosztowe (ceny różnią się między regionami) i praktyczne (opóźnienie do użytkowników).

| Dostawca | Regiony | Strefy | Źródło |
| --- | --- | --- | --- |
| AWS | 39 | 124 | [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) |
| Google Cloud | 43 | 130 | [Google Cloud locations](https://cloud.google.com/about/locations) |
| Microsoft Azure | ponad 70 | co najmniej 3 w regionach, które je obsługują | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/reliability/regions-overview) |

Tych liczb nie da się porównać jeden do jednego, bo każdy dostawca inaczej definiuje region. Microsoft i Google mają regiony w Warszawie (Poland Central i europe-central2), co ma znaczenie dla polskich firm z wymogami lokalizacji danych.

### Na czym będziemy ćwiczyć

Wszyscy trzej dostawcy dają nowym użytkownikom darmowe środki lub darmowy limit usług: [AWS Free Tier](https://aws.amazon.com/free/), [Azure Free Account](https://azure.microsoft.com/free/) i [Google Cloud Free Tier](https://cloud.google.com/free). Warunki się zmieniają, więc sprawdź je przed założeniem konta. Do części ćwiczeń z AWS użyjemy środowiska AWS Academy Learner Lab, które nie wymaga karty płatniczej.

Jedna rada, której warto posłuchać od pierwszego dnia: po każdych zajęciach wyłącz lub usuń uruchomione zasoby i ustaw alert budżetowy na kilka dolarów. To najtańsza lekcja z zarządzania kosztami w chmurze.

## Co zabierasz z zajęć 1

Wróćmy do zdania z początku: chmura to sposób kupowania mocy obliczeniowej jak prądu. Teraz możesz je rozwinąć.

1. **Chmurą jest tylko to, co ma pięć cech NIST.** Samoobsługa, dostęp przez sieć, współdzielenie, elastyczność i rozliczanie za zużycie. Brak którejś to sygnał, że masz do czynienia z hostingiem.
2. **Model usługi wyznacza granicę odpowiedzialności.** W IaaS dbasz o system operacyjny, w PaaS już nie, ale za dane odpowiadasz zawsze.
3. **Chmura opłaca się przy zmiennym obciążeniu.** Przy stałym, przewidywalnym obciążeniu policz TCO, zanim zdecydujesz.
4. **Rynek należy do trzech graczy, ale proporcje się przesuwają.** AWS prowadzi z 28%, Microsoft ma 20%, Google rośnie najszybciej i ma 15% (Synergy, II kw. 2026). Udział w rynku to nie to samo co przychody.

### Pytania kontrolne

1. Firma wynajmuje serwer w centrum danych na trzy lata za stałą miesięczną opłatę. Które cechy chmury z definicji NIST nie są spełnione?
2. Uruchamiasz bazę danych w Amazon RDS. Kto instaluje łatki bezpieczeństwa silnika bazy, a kto odpowiada za to, kto ma dostęp do tabeli z danymi klientów?
3. Podaj przykład firmy, dla której własna serwerownia będzie tańsza niż chmura, i uzasadnij dlaczego.
4. Udział AWS w rynku spadł w ciągu roku o 2 punkty procentowe, a przychody wzrosły o 37%. Jak to możliwe?
5. Dobierz odpowiedniki w Azure i Google Cloud dla Amazon S3, Amazon EC2 i Amazon DynamoDB.
6. Dlaczego nie należy porównywać udziałów rynkowych podawanych przez Gartnera i przez Synergy Research?

### Słowniczek

| Termin | Znaczenie |
| --- | --- |
| IaaS | Infrastructure as a Service: wynajem maszyn wirtualnych, dysków i sieci |
| PaaS | Platform as a Service: gotowa platforma (np. baza danych), bez zarządzania serwerem |
| SaaS | Software as a Service: gotowa aplikacja dostępna przez przeglądarkę |
| CAPEX / OPEX | Wydatki inwestycyjne (zakup sprzętu) / koszty operacyjne (opłaty bieżące) |
| TCO | Total Cost of Ownership: całkowity koszt posiadania, łącznie z ludźmi, energią i migracją |
| Region / strefa dostępności | Lokalizacja geograficzna dostawcy / odizolowane centrum danych w regionie |
| Egress | Transfer danych z chmury na zewnątrz, zwykle płatny |
| Hyperscaler | Dostawca o globalnej skali: AWS, Microsoft, Google |
| Multi-cloud | Korzystanie z więcej niż jednego dostawcy chmury publicznej |

### Źródła i dalsza lektura

- [NIST SP 800-145: The NIST Definition of Cloud Computing](https://csrc.nist.gov/pubs/sp/800/145/final), 2011
- [Synergy Research Group: Q2 Cloud Market Passes $143 Billion](https://www.srgresearch.com/articles/q2-cloud-market-passes-143-billion-highest-growth-rate-in-eight-years), 30.07.2026
- [Cloud Market Share 2026: AWS vs Azure vs Google Cloud](https://www.articsledge.com/post/cloud-market-share), Articsledge, 4.10.2026 (zestawienie danych Synergy i Gartnera)
- [Gartner: Worldwide IaaS Public Cloud Services Market Grew 22.5% in 2024](https://www.gartner.com/en/newsroom/press-releases/2025-08-06-gartner-says-worldwide-iaas-public-cloud-services-market-grew-22-point-5-percent-in-2024), 6.08.2025
- [Amazon: wyniki za II kwartał 2026](https://www.sec.gov/Archives/edgar/data/0001018724/000101872426000024/amzn-20260630xex991.htm) i [Alphabet: wyniki za II kwartał 2026](https://www.sec.gov/Archives/edgar/data/0001652044/000165204426000066/googexhibit991q22026.htm)
- Z sylabusa: Amazon, *The Economics of the AWS Cloud vs. Owned IT Infrastructure*; [Azure Fundamentals (AZ-900)](https://learn.microsoft.com/en-us/training/courses/az-900t00); [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework); J. Rosenberg, A. Mateos, *Chmura obliczeniowa. Rozwiązania dla biznesu*, Helion 2012
