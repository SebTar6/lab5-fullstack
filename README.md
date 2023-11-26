# Nazwa Projektu

Repozytorium zawiera rozwiązanie do laboratorium numer 5 z przedmiotu Programowanie Full-Stack w Chmurze Obliczeniowej.

## Spis

- [namespace.yaml](#namespace.yaml)
- [res_quota.yaml](#res_quota.yaml)
- [worker_pod.yaml](#worker_pod.yaml)
- [php_apache_deployment.yaml](#php_apache_deployment.yaml)
- [autoscaler.yaml](#autoscaler.yaml)
- [instalacja](#instalacja)
- [testowanie](#testowanie)

## namespace.yaml

Plik definiuje namespace, w której będą uruchamiane komponenty rozwiązania.

## res_quota.yaml

Plik definiuje limity zasobów dla przestrzeni nazw "zad5", takie jak maksymalna liczba Pod-ów, maksymalne żądania CPU i maksymalne żądania pamięci RAM.
Te ograniczenia będą obowiązywać w obrębie tej przestrzeni nazw, a system Kubernetes będzie monitorować i egzekwować te limity.

## worker_pod.yaml

Plik definiuje Pod o nazwie "worker" w przestrzeni nazw "zad5" z jednym kontenerem opartym na obrazie Nginx.
Ograniczenia na zasoby (pamięć RAM i procesor) są również zdefiniowane dla tego kontenera.

## php_apache_deployment.yaml


Plik definiuje Deployment oraz Service w przestrzeni nazw zad5 dla aplikacji "php-apache".
Deployment określa, że każda instancja ma być oparta na obrazie "k8s.gcr.io/hpa-example" z zdefiniowanymi ograniczeniami na zasoby.
Dodatkowo, Service "php-apache" jest utworzony jako interfejs dla komunikacji z replikami, przekierowując ruch z portu 80 do odpowiednich Pod-ów opartych na etykiecie "run: php-apache".
Ten plik jest kluczowy do zarządzania skalowaniem i dostępnością aplikacji w środowisku Kubernetes.

## autoscaler.yaml

Plik definiuje obiekt HorizontalPodAutoscaler (HPA) w przestrzeni nazw zad5.
HPA umożliwia automatyczne skalowanie aplikacji w oparciu o aktualne obciążenie. 
W tym przypadku, HPA skonfigurowano dla Deploymentu "php-apache". Określa się minimalną (1) i maksymalną (5) liczbę replik (instancji) wdrożenia oraz docelową utylizację CPU na poziomie 50%.
Dzięki temu, gdy obciążenie aplikacji wzrasta, HPA automatycznie zwiększa liczbę replik, a gdy spada, zmniejsza, dostosowując się do zapotrzebowania na zasoby.
Konfiguracja HPA jest kluczowa dla elastycznego dostosowywania infrastruktury do zmieniających się warunków działania aplikacji w środowisku Kubernetes.

## Instalacja

Utwórz przestrzeń nazw:
```bash
kubectl apply -f zad5-namespace.yaml

Utwórz Quotę:
```bash
kubectl apply -f zad5-quota.yaml

Utwórz Pod-a w przestrzeni zad5:
```bash
kubectl apply -f zad5-worker-pod.yaml

Utwórz Deployment i Service:
```bash
kubectl apply -f zad5-php-apache.yaml

Utwórz HorizontalPodAutoscaler:
```bash
kubectl apply -f zad5-hpa.yaml

Potwierdź działanie i sprawdź zasoby:
```bash
kubectl get all -n zad5
```bash
kubectl describe quota -n zad5 resource-limit
```bash
kubectl describe hpa -n zad5 php-apache-hpa

## Testowanie

Uzyskaj adres IP i port usługi "php-apache" w przestrzeni nazw "zad5":
```bash
kubectl get service -n zad5 php-apache

Do testowania obciążenia można użyć różnych narzędzi.
Ja na potrzeby testu użyję narzędzia "wrk", które testuje wydajność HTTP.

Instalacja dla Red Hat/CentOS:
```bash
sudo yum install wrk

Instalacja dla Debian/Ubuntu:
```bash
sudo apt-get install wrk
