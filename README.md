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

![image](https://github.com/user-attachments/assets/f67fe5a4-b5fa-4443-a1f2-a44a12e3bf41)


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

![image](https://github.com/user-attachments/assets/39aad443-b255-45e1-a8cb-9db876c2e16d)

<p>Dostęp z innej przestrzeni nazw nie jest możliwy, ze względu na zastosowaną NetworkPolicy.</p>

![image](https://github.com/user-attachments/assets/d22dcfde-6daf-42b8-9e84-5e06892d3ec7)

<p>Natomiast w tej samej przestrzeni nazw ruch jest możliwy.</p>

![image](https://github.com/user-attachments/assets/2c67ae74-3f1f-407f-b630-3dfc333d97bd)

<p>W celu sprawdzenia czy skalowanie działa obiektu StatefullSet działa poprawnie wraz ze zdefiniowanymi PV oraz PVC przeskalowano bazę danych do 3 replik za pomocą polecenia: </p>

```
kubectl scale statefulset mysql --replicas=3 -n zadanie-2
```

![image](https://github.com/user-attachments/assets/dfe6711f-1208-4610-b8a0-b1fd56011f4c)

<p>Nowe PVC są tworzone automatycznie dla każdego nowego poda w StatefulSet.Każdy PVC jest przypisany do unikalnego PV. </p>

![image](https://github.com/user-attachments/assets/febc18cd-4d6a-46c4-8443-ef18c3a04430)

Podczas skalowania w dół pody sa usuwane w kolejności od największego indeksu.

![image](https://github.com/user-attachments/assets/0ac886c0-7905-49ed-987a-5e1b7f05c122)

## Opis wybranych manifestów
<ul>
  <li><b>1-namespace-zadanie-2</b> tworzy przestrzeń nazw zadanie-2</li>
  <li><b>apache-content.yaml</b> tworzy obiekt typu ConfigMap i przechowuje plik index.php z prostym skryptem PHP wyświetlającym komunikaty testowe.</li>
  <li><b>apache-service</b> tworzy usługę typu ClusterIP, która kieruje ruch na port 80 do podów z selektorem app: php-apache, analogicznie sytuacja wygląda dla pliku default-backend.yaml w sekcji service</li>
  <li><b>ingress-for-lamp.yaml</b> tworzy zasób Ingress z regułą, która kieruje ruch HTTP na ścieżkę /app w domenie myapp.local do usługi apache-service na porcie 80, a domyślny backend to default-backend-service na porcie 80</li>
  <li><b>mysql-pv.yaml</b> tworzy zasów PersistentVolume, przydzielający 1Gi pamięci i używający hostPath na ścieżce /mnt/data</li>
  <li><b>mysql-pvc.yaml</b> żąda 1Gi przestrzeni dyskowej i ma dostęp do PV w trybie ReadWriteOnce</li>
  <li><b>mysql-secret.yaml oraz php-apache-secret.yaml</b> zawierają zakodowane w base64 dane dostępowe do bazy MySQL</li>
  <li><b>mysql-statefulset.yaml</b> uruchamia jeden replikę kontenera MySQL, dodaje niezbędne dane konfigurcyjne jak hasła do bazy danych, działa na porcie 3306, implementuje asób PersistentVolumeClaim dla przechowywania danych MySQL na dysku z mysql-pvc</li>
  <li><b>mysql-pvc.yaml</b> żąda 1Gi przestrzeni dyskowej i ma dostęp do PV w trybie ReadWriteOnce</li>
  <li><b>network-policy.yaml</b> zezwala na przychodzący ruch (Ingress) do wszystkich podów w tym namespace zadanie-2</li>
  <li><b>php-apache.yaml</b> uruchamia dwie repliki kontenera PHP-Apache na porcie 80. Montuje wolumen apache-content (pochodzący z ConfigMap), korzystają z danych z sekretów.</li>
</ul>

## Podsumowanie 
Pomyślnie udało się wdrożyć stack LAMP w środowisku Kubernetes przy użyciu Minikube. Poszczególne komponenty działają zgodnie z założeniami. Dzięki wykonanym testom, udało się zweryfikować poprawność działania aplikacji, bazy danych oraz wszystkich powiązanych zasobów. Wykonanie tego zadania pozwoliło na zweryfikowanie zdobytych umiejętnosciw praktyczym zatosowanaiu, które są podstawą w rozwiązaniach chmurowych.

<hr>
<div style="text-align: justify;">
  <i>Opracowanie zadania powstało w ramach laboratorium.</i>
</div>
