# Nazwa Projektu

Repozytorium zawiera rozwiązanie do laboratorium numer 5 z przedmiotu Programowanie Full-Stack w Chmurze Obliczeniowej.

## Spis

- [namespace.yaml](#namespace)
- [res_quota.yaml](#resourcequota)
- [worker_pod.yaml](#pod)
- [php_apache_deployment.yaml](#deploymentservice)
- [autoscaler.yaml](#autoscaler)
- [instalacja](#instalacja)
- [testowanie](#testowanie)
- [wyliczanie max replik](#repliki)

## Namespace

Plik definiuje namespace, w której będą uruchamiane komponenty rozwiązania.

## ResourceQuota

Plik definiuje limity zasobów dla przestrzeni nazw "zad5", takie jak maksymalna liczba Pod-ów, maksymalne żądania CPU i maksymalne żądania pamięci RAM.
Te ograniczenia będą obowiązywać w obrębie tej przestrzeni nazw, a system Kubernetes będzie monitorować i egzekwować te limity.

## Pod

Plik definiuje Pod o nazwie "worker" w przestrzeni nazw "zad5" z jednym kontenerem opartym na obrazie Nginx.
Ograniczenia na zasoby (pamięć RAM i procesor) są również zdefiniowane dla tego kontenera.

## DeploymentService


Plik definiuje Deployment oraz Service w przestrzeni nazw zad5 dla aplikacji "php-apache".
Deployment określa, że każda instancja ma być oparta na obrazie "k8s.gcr.io/hpa-example" z zdefiniowanymi ograniczeniami na zasoby.
Dodatkowo, Service "php-apache" jest utworzony jako interfejs dla komunikacji z replikami, przekierowując ruch z portu 80 do odpowiednich Pod-ów opartych na etykiecie "run: php-apache".
Ten plik jest kluczowy do zarządzania skalowaniem i dostępnością aplikacji w środowisku Kubernetes.

## HorizontalPodAutoscaler

Plik definiuje obiekt HorizontalPodAutoscaler (HPA) w przestrzeni nazw zad5.
HPA umożliwia automatyczne skalowanie aplikacji w oparciu o aktualne obciążenie. 
W tym przypadku, HPA skonfigurowano dla Deploymentu "php-apache". Określa się minimalną (1) i maksymalną (5) liczbę replik (instancji) wdrożenia oraz docelową utylizację CPU na poziomie 50%.
Dzięki temu, gdy obciążenie aplikacji wzrasta, HPA automatycznie zwiększa liczbę replik, a gdy spada, zmniejsza, dostosowując się do zapotrzebowania na zasoby.
Konfiguracja HPA jest kluczowa dla elastycznego dostosowywania infrastruktury do zmieniających się warunków działania aplikacji w środowisku Kubernetes.

## Instalacja

Utwórz przestrzeń nazw:
```bash
kubectl apply -f namespace.yaml
```

Utwórz Quotę:
```bash
kubectl apply -f res_quota.yaml
```

Utwórz Pod-a w przestrzeni zad5:
```bash
kubectl apply -f worker_pod.yaml
```

Utwórz Deployment i Service:
```bash
kubectl apply -f php_apache_deployment.yaml
```

Utwórz HorizontalPodAutoscaler:
```bash
kubectl apply -f autoscaler.yaml
```

Potwierdź działanie i sprawdź zasoby:
```bash
kubectl get all -n zad5
```
```bash
kubectl describe quota -n zad5 resource-limit
```
```bash
kubectl describe hpa -n zad5 php-apache-hpa
```

## Testowanie

Uzyskaj adres IP i port usługi "php-apache" w przestrzeni nazw "zad5":
```bash
kubectl get service -n zad5 php-apache
```

Do testowania obciążenia można użyć różnych narzędzi.
Ja na potrzeby testu użyję narzędzia "wrk", które testuje wydajność HTTP.

Instalacja dla Red Hat/CentOS:
```bash
sudo yum install wrk
```

Instalacja dla Debian/Ubuntu:
```bash
sudo apt-get install wrk
```

Przykładowe użycie:
```bash
wrk -t10 -c10 -d1m http://<ip>:<port>/
```

## Repliki

Aby wyliczyć ilość replik należy uwzględnić dostępność zasobów dla danego namespace.
W namespace lab5 jest dostępnych: 2000m CPU oraz 1.5Gi RAM.

Mamy jeden działający pod (worker), który zabiera maksymalnie 200m CPU oraz 200Mi RAM.
Zostaje do dyspozycji dla depoymentu: 1800m CPU oraz 1.3Gi RAM.

Każdy pod w deploymencie zabiera maksymalnie 250m CPU oraz 250Mi RAM.
Należy wykonać dzielenie całkowite:
- 1800m // 250m = 7
- 1300Mi // 250Mi = 5

Widzimy, że jesteśmy ograniczeni ze względu na dostępny RAM, a maksymalna liczba replik, którą może uruchomić deployment w tym namespace to: **5**
