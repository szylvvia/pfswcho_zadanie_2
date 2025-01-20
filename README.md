# ZADANIE 2 - Programowanie FullStack w Chmurze Obliczeniowej

<p>W ramach zadania opracowano wdrożenie stack-a LAMP w środowisku Kubernetes (minikube), zdefiniowano zasady dostępu do niego z zewnątrz oraz przetestowano uruchomienie, w celu potwierdzenia poprawności wykonanych działań.</p><br>

## Założenia wstępne
<ol>
  <li>Jako stack do wdrożenia wybrano <b>LAMP</b> (Linux, Apache, MySQL, PHP).</li>
  <li>Wszytskie zasoby są wdrażane w przestrzeni nazw zadanie-2.</li>
  <li>Zasoby są skonfigurowane do używania zmiennych środowiskowych z sekretów.</li>
  <li>Zamontowano wolumin ConfigMap, który pozwala na wyświetlenie przykładowej zawartości aplikacji.</li>
  <li>Dostęp z zewnątrz jest możliwy przy wykorzystaniu kontrolera i obiektów Ingress, natomiast dostęp do bazy danych MySQL jest ograniczony do komunikacji wewnętrznej w klastrze.</li>
  <li>Skonfigurowano również domyślny backend.</li>
  <li>Zaimplementowano NetworkPolicy, która pozwala na ruch przychodzący do wszystkich podów w przestrzeni nazw zadanie-2.</li>
  <li>Skonfigurowano PersistentVolume i PersistentVolumeClaim do przechowywania danych MySQL.</li>
</ol>

## Niezbędne zasoby
<ol>
  <li>Przestrzeń nazw w celu zapewnienia izolacji.</li>
  <li>Deployment oraz service dla serwera Apache-PHP oraz domyślnego backendu.</li>
  <li>StatefulSet oraz service (headless) dla bazy danych MySQL wraz z obiektami PV(PersistentVolume) oraz PVC (PersistentVolumeClaim).</li>
  <li>Obiekt Ingress dla umożliwienia dostepu z zewnątrz.</li>
  <li>NetworkPolicy w celu ograniczenia ruchu tylko do przestrzeni nazw.</li>
  <li>Pliki sekretów z danymi konfiguracyjnymi.</li>
  <li>Obiekt ConfigMap pozwalajacy na zamontowanie przykładowej zawartosci strony.</li>
</ol>

## Manifesty stanowiące definicję wdrożenia 
<p>Wszytskie niezbędne pliki Znajdują się w folderze resources.</p>

## Sposób wdrożenia
<p>Proces wdrożenia stack-a polega na uruchomieniu poszczególnych plików manifestów. Ponieważ znajdują się w jednym folderze, można uruchomić je zbiorczo, przy pomocy poniższego polecenia:

```
kubectl apply -f./resources
```

![image](https://github.com/user-attachments/assets/84b4966b-6463-4fc7-b8f3-6878c6f23364)

Po uruchomieniu plików manifestów warto sprawdzić poprawność utworzenia i wystartowania poszczególnych obiektów, można wykorzystać do tego poniższe polecenia:

```
kubectl get all -n zadanie-2
```

![image](https://github.com/user-attachments/assets/21603d22-b0ca-4088-bf97-e023bce9da96)

```
kubectl get ingress -n zadanie-2
```

![image](https://github.com/user-attachments/assets/21e740c8-fd8b-46c3-b629-17662fb00e7f)

```
kubectl get secret -n zadanie-2 
```

![image](https://github.com/user-attachments/assets/880b6942-79dd-454d-a8b2-28081f713651)

```
kubectl describe  secret -n zadanie-2
```

![image](https://github.com/user-attachments/assets/bc82e74c-3153-4fac-96cb-3f12dac1a692)

```
kubectl get pv,pvc -n zadanie-2
```

![image](https://github.com/user-attachments/assets/2ca14d53-0161-4917-9b1f-328061ce82f3)

```
kubectl describe  networkpolicy -n zadanie-2
```

![image](https://github.com/user-attachments/assets/eeeec049-f2d7-4cba-8375-e6ea06002f92)


## Sposób dostępu z zewnątrz i weryfikacja działania

Dzięki zastosowaniu kontrolera Ingress i jego obiektów możliwy jest dostęp do serwera Apache-PHP z zewnątrz. Można to zrobić wykorzystując od tego przeglądarkę internetowa i wpisujac adres:

```
http://myapp.local/app
```

![image](https://github.com/user-attachments/assets/93647bbf-a9f2-4bf7-89e3-83a8be27e695)


Ponieważ został zaimplementowany domyślny backend, próba przejścia pod nieistniejacy adres zakończy sie wyswietleniem hello-app.

```
http://myapp.local/nonexisting
```

![image](https://github.com/user-attachments/assets/2a17a538-3e4b-4980-a821-70e100bc296d)


<p>Aby możliwe było skorzystanie z tego adresbu bespośrednio konieczny był wpis do pliku C:\Windows\System32\drivers\etc\hosts </p>

```
127.0.0.1 myapp.local
```

<p>W celu sprawdzenia poprawności działania bazy danych, można przetestować możliość połaczenia się z nią za pomocą poda: </p>

```
kubectl exec -it mysql-0 -n zadanie-2 -- /bin/bash
```

<p>Hasło dla roota podajemy to samo (tylko w postaci niezakdowanej), które zostało skonfigurowane w sekretach. </p>

```
mysql -u root -p
```

![image](https://github.com/user-attachments/assets/b787f5f7-d1da-42ba-b531-251f4fbe4e54)




