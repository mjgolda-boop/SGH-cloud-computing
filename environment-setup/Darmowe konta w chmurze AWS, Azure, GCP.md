# Darmowe konta w chmurze: AWS, Azure, GCP

## W skrócie

Na AWS zacznij od AWS Academy Learner Lab. SGH jest w tym programie, nie trzeba podawać karty i nie da się dostać rachunku. Na Azure najbezpieczniejszy jest Azure for Students. Każde zadanie wysyła się osobno i można je zrobić na innej platformie, więc opcje da się łączyć, np. zadanie 4 na Azure, a resztę na AWS.

| Opcja | Darmowe środki | Ważność | Karta | Co się dzieje na końcu | Czy da się zrobić wszystkie 5 zadań? |
| --- | --- | --- | --- | --- | --- |
| [AWS Academy Learner Lab](https://its.ucsc.edu/services/teaching-and-learning/learning-management/aws-academy) | budżet przydzielony przez program, zwykle 50 USD | do 6 miesięcy (czas trwania kursu) | nie | środowisko się wyłącza, nic nie jest naliczane | Tak. Zadania 3 i 5 (wariant SNS) korzystają z tymczasowego tokenu sesji laboratorium. Przy zadaniu 4 (Oracle w RDS) i 5 (EKS) pilnuj budżetu, a przed zadaniem 4 sprawdź, czy laboratorium pozwala na silnik Oracle. |
| [AWS Free Plan](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html) | 100 USD na start + do 100 USD za zadania w konsoli | 6 miesięcy lub do wyczerpania kredytu | tak | konto zamyka się samo, bez opłat | Najpewniej tak. Część usług jest na Free Plan zablokowana, więc przed startem sprawdź RDS z Oracle (zad. 4) i EKS (zad. 5). Zadanie 5 można też zrobić w wariancie SNS. |
| [Azure for Students](https://learn.microsoft.com/en-us/answers/questions/5837170/question-about-azure-for-students) | 100 USD | 12 miesięcy, odnawialne co rok | nie | kredyt się kończy, nic nie jest naliczane | Tak. Subskrypcja ma limity vCPU i regionów, więc klaster AKS (zad. 5) zbuduj na jednym małym węźle. |
| [Azure free account](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/avoid-charges-free-account) | 200 USD + wybrane usługi darmowe przez 12 miesięcy | kredyt: 30 dni | tak (nie przedpłacona) | subskrypcja wyłączona, dopóki nie przejdziesz na pay-as-you-go | Tylko jeśli zrobisz wszystko w 30 dni. Potem trzeba przejść na pay-as-you-go, czyli płacić za wszystko ponad darmowe limity. |
| [Google Cloud Free Trial](https://docs.cloud.google.com/free/docs/free-cloud-features) | 300 USD + usługi Free Tier | 90 dni | tak | konto rozliczeniowe zamyka się samo, bez opłat | Tak, w ciągu 90 dni. Najwięcej kosztują GKE (zad. 5) i Cloud SQL (zad. 4), więc usuń je zaraz po zaliczeniu. |

Portal sprawdza zasób na żywo, w chwili wysłania zadania. Maszyna, strona, baza albo klaster muszą wtedy działać, a w Learner Lab dodatkowo musi trwać sesja laboratorium. Gdy zobaczysz **OK - ACCEPTED**, usuń to, czego już nie potrzebujesz.

## AWS Academy Learner Lab

SGH należy do programu AWS Academy, więc to podstawowe środowisko do części kursu poświęconej AWS. Nie trzeba podawać karty, a gdy budżet się skończy, środowisko po prostu się wyłącza. Prowadzący widzi zużycie kredytu każdej osoby i może pomóc bezpośrednio w jej konsoli.

1. Poczekaj na zaproszenie z AWS Academy na adres uczelniany. Przyjdzie z platformy Canvas ([awsacademy.instructure.com](https://awsacademy.instructure.com)).
2. Kliknij link w mailu, ustaw hasło do konta AWS Academy i zaakceptuj regulamin.
3. Otwórz kurs **AWS Academy Learner Lab**, przejdź do **Modules → Launch AWS Academy Learner Lab** i zaakceptuj warunki środowiska Vocareum.
4. Kliknij **Start Lab**. Gdy kropka przy napisie **AWS** zmieni kolor na zielony, kliknij go. Otworzy się zwykła konsola AWS.
5. Po skończonej pracy kliknij **End Lab** i usuń zasoby, których już nie potrzebujesz. One też zużywają budżet.

O czym trzeba pamiętać (szczegóły są w *AWS Academy Learner Lab Student Guide* w tym samym kursie):

- Budżetu nie da się doładować. Kto go przepali, traci dostęp do środowiska do końca kursu.
- Działają tylko wybrane regiony w USA, zwykle **us-east-1** i **us-west-2**.
- Sesja trwa kilka godzin (licznik widać w panelu). Po jej zakończeniu maszyny się zatrzymują, ale nie znikają.
- Nie można tworzyć własnych użytkowników ani ról IAM, jest tylko gotowa rola laboratoryjna. W zadaniach, które wymagają kluczy AWS, użyj access key, secret key i session token z panelu **AWS Details**. Wygasają razem z sesją.
- Część przykładów z dokumentacji AWS nie zadziała, bo niektóre usługi i typy instancji są zablokowane.

## AWS: konto z Free Plan

To konto warto założyć tylko wtedy, gdy chcesz pracować na AWS po kursie albo potrzebujesz usługi zablokowanej w Learner Lab. Potrzebujesz adresu e-mail, telefonu i karty płatniczej. Rejestracja trwa około 10 minut.

1. Wejdź na [aws.amazon.com/free](https://aws.amazon.com/free) i kliknij **Create a free account**.
2. Podaj adres e-mail (to będzie login użytkownika *root*) i nazwę konta, np. `sgh-cloud-jan-kowalski`. Wpisz kod, który przyjdzie mailem.
3. Ustaw silne hasło dla użytkownika root i zapisz je w menedżerze haseł.
4. Wybierz typ konta **Personal** i uzupełnij dane kontaktowe bez polskich znaków (np. *Warszawa*, *ul. Niepodleglosci*).
5. Wybierz **Free plan**. To ważne: na tym planie AWS nie obciąży karty ponad kredyt, a po 6 miesiącach konto po prostu się zamyka. Plan płatny można włączyć później jednym kliknięciem.
6. Podaj dane karty. AWS może zrobić tymczasową autoryzację na niewielką kwotę, która zniknie po kilku dniach.
7. Potwierdź tożsamość kodem z SMS-a lub połączenia głosowego.
8. Przy wyborze planu wsparcia zostaw **Basic support – Free**.
9. Zaloguj się do konsoli jako root na [console.aws.amazon.com](https://console.aws.amazon.com). Region ustaw na **Europe (Frankfurt) eu-central-1**.

Od razu po pierwszym logowaniu zrób trzy rzeczy:

- [ ] Włącz MFA dla konta root (menu z nazwą konta → **Security credentials** → **Assign MFA device**).
- [ ] Utwórz budżet w **AWS Budgets**, np. 5 USD z alertem mailowym. Przy okazji to jedno z zadań w widżecie *Explore AWS*, za które AWS dopisuje 20 USD kredytu.
- [ ] Do codziennej pracy utwórz osobnego użytkownika (IAM Identity Center lub IAM), a z konta root korzystaj tylko awaryjnie. Do zadań 3 i 5 utwórz osobne klucze, tylko z prawem odczytu albo publikowania.

Uwaga: na Free Plan część usług jest niedostępna, głównie tych, które od razu zjadłyby cały kredyt. Jeśli konsola blokuje usługę potrzebną do zadania, napisz do prowadzącego, zanim przełączysz konto na plan płatny.

## Azure: dwie drogi

**Wariant A: Azure for Students (bez karty, zalecany)**

1. Wejdź na [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students) i kliknij **Start free**.
2. Zaloguj się kontem Microsoft. Najprościej użyć uczelnianego adresu e-mail; prywatne konto też zadziała, jeśli w kolejnym kroku potwierdzisz status studenta.
3. Zweryfikuj status studenta adresem uczelnianym SGH. Microsoft wyśle na niego link lub kod.
4. Potwierdź numer telefonu kodem z SMS-a i zaakceptuj warunki.
5. Po kilku minutach w [portal.azure.com](https://portal.azure.com) pojawi się subskrypcja **Azure for Students** ze 100 USD kredytu ważnego 12 miesięcy.

Jeśli weryfikacja domeny uczelnianej się nie powiedzie, wybierz wariant B.

**Wariant B: Azure free account (z kartą)**

1. Wejdź na [azure.microsoft.com/free](https://azure.microsoft.com/free) i kliknij **Try Azure for free**.
2. Zaloguj się kontem Microsoft albo GitHub (lub załóż nowe).
3. Uzupełnij profil: kraj **Poland**, imię, nazwisko, telefon. Potwierdź telefon kodem.
4. Podaj kartę kredytową lub debetową. Karty przedpłacone nie są akceptowane. Microsoft nie obciąży karty, dopóki sam nie przejdziesz na pay-as-you-go.
5. Zaakceptuj umowę i kliknij **Sign up**. Trafisz do portalu z subskrypcją **Azure subscription 1** i kredytem 200 USD.

Pamiętaj, że te 200 USD przepada po 30 dniach od rejestracji. Konto zakładaj dopiero wtedy, gdy jesteś gotów zrobić zadania na Azure jedno po drugim, a nie na początku semestru.

Po pierwszym logowaniu:

- [ ] Włącz MFA w ustawieniach konta Microsoft (Microsoft Entra ID i tak wymaga go przy logowaniu do portalu).
- [ ] W **Cost Management + Billing → Budgets** ustaw budżet z alertem mailowym.
- [ ] Na każde zadanie utwórz osobną *resource group*, np. `rg-sgh-zad3`. Sprzątanie to wtedy usunięcie jednej grupy.

## Google Cloud: Free Trial

Potrzebujesz konta Google (np. Gmail) i karty płatniczej. Darmowy okres dostaje tylko ktoś, kto nigdy nie płacił za Google Cloud, Maps Platform ani Firebase i wcześniej nie korzystał z Free Trial.

1. Wejdź na [cloud.google.com/free](https://cloud.google.com/free) i kliknij **Get started for free**.
2. Zaloguj się kontem Google.
3. Wybierz kraj **Poland**, zaznacz cel (np. *Personal project* lub *Education*) i zaakceptuj warunki.
4. Wybierz typ konta **Individual**, uzupełnij adres i dane karty. Google może zrobić tymczasową autoryzację, która później znika.
5. Kliknij **Start my free trial**. Otworzy się konsola [console.cloud.google.com](https://console.cloud.google.com) z kontem rozliczeniowym *Free Trial* i kredytem 300 USD na 90 dni.
6. Utwórz projekt dla zajęć (**Select a project → New project**), np. `sgh-cloud-2026`, i upewnij się, że jest podpięty pod konto rozliczeniowe Free Trial.

Konsola może proponować przycisk **Activate / Upgrade**. Nie klikaj go na potrzeby zajęć. Dopóki konto jest w trybie Free Trial, Google nic nie nalicza, a po 90 dniach lub wyczerpaniu kredytu konto zamyka się samo (z 30-dniowym okresem na odzyskanie danych). Bezterminowe limity Free Tier zostają tylko po przejściu na płatne konto.

Po pierwszym logowaniu:

- [ ] Włącz weryfikację dwuetapową na koncie Google.
- [ ] W **Billing → Budgets & alerts** ustaw budżet z alertami na 50%, 90% i 100%.
- [ ] Do zadań 3 i 5 utwórz osobne konto serwisowe z prawem tylko do odczytu (Firestore) albo tylko do publikowania (Pub/Sub) i użyj jego klucza JSON. Usuń klucz, gdy zadanie zostanie zaliczone.

## Jak nie dostać rachunku

Najczęstsza przyczyna niespodziewanych kosztów jest banalna: maszyna wirtualna, baza danych albo klaster Kubernetes zostawione na noc lub na weekend. Kilka nawyków wystarcza, żeby tego uniknąć.

- Po każdym zaliczonym zadaniu usuń to, co utworzyłeś: instancje, dyski, adresy IP, bazy, klastry, LoadBalancery. Zatrzymana maszyna nadal płaci za dysk.
- Nie przełączaj konta na plan płatny (AWS Paid plan, Azure pay-as-you-go, GCP Upgrade), jeśli prowadzący wyraźnie o to nie prosi.
- Nigdy nie wklejaj do portalu danych konta root i nie wrzucaj kluczy dostępowych do publicznego repozytorium na GitHubie. Boty znajdują je w kilka minut i uruchamiają na cudzym koncie koparki kryptowalut.
- Raz w tygodniu zajrzyj do panelu kosztów: AWS Billing, Azure Cost Management, GCP Billing.
- Jedna osoba może skorzystać z darmowej oferty tylko raz na każdej platformie. Nie zakładaj „testowego” konta wcześniej, bo potem zabraknie kredytu na zadania.

## Źródła

- [AWS Free Tier – dokumentacja](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html)
- [Ogłoszenie AWS: 200 USD kredytu i 6-miesięczny Free Plan (lipiec 2025)](https://aws.amazon.com/about-aws/whats-new/2025/07/aws-free-tier-credits-month-free-plan/)
- [AWS Academy Learner Lab – opis programu (UC Santa Cruz)](https://its.ucsc.edu/services/teaching-and-learning/learning-management/aws-academy)
- [Azure: jak uniknąć opłat na darmowym koncie](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/avoid-charges-free-account)
- [Microsoft Q&A: Azure for Students](https://learn.microsoft.com/en-us/answers/questions/5837170/question-about-azure-for-students)
- [Google Cloud Free Trial i Free Tier](https://docs.cloud.google.com/free/docs/free-cloud-features)
- [Google Cloud Free Trial – FAQ](https://cloud.google.com/signup-faqs)
