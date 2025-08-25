# Service Request Ticket — Solution
## 1. Клонування репозиторію застосунку
```bash
git clone --depth=1 https://github.com/den-vasyliev/go-demo-app.git
cd go-demo-app
```
## 2. Встановлення застосунку через Helm
```bash
helm install current-version ./helm
```
Це підняло базові мікросервіси: API, базу, брокер, gateway (Ambassador).
## 3. Перевірка роботи API

Прокидуємо порт ambassador на локальну машину:
```bash
kubectl port-forward svc/ambassador 8080:80
```
Тестуємо:
```bash
curl http://localhost:8080/api/
# k8sdiy-api:599e1af
```
## 4. Розгортання нової версії API

Для підготовки робочого маніфесту використовуємо команду:
```bash
helm template new-version ./helm \
  -s templates/api-deploy.yaml \
  -s templates/api-svc.yaml \
  -s templates/app-configmap.yaml \
  -s templates/secret.yaml \
  --set image.tag=build-802e329 \
  --set api.canary=true \
  --set api.header=X-QA
```
Ця команда рендерить повний набір ресурсів (Deployment, Service, ConfigMap, Secret) для нової версії API з тегом build-802e329 у канарейковому режимі, доступному лише по заголовку X-QA.
## 5. Створення Mapping у Emissary (Ambassador)

Після оновлення до Emissary Ingress 3.9.0 та встановлення CRD ми додали два маршрути:

Публічний — на стару версію:
```yaml
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: api-public
spec:
  prefix: /api/
  service: current-version-api:8080
```
QA-only — на нову версію (із пріоритетом та заголовком):
```yaml
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: api-qa
spec:
  prefix: /api/
  service: new-version-api:8080
  headers:
    X-QA: "1"
  precedence: 20
```
### публічний трафік → стара версія
curl http://localhost:8080/api/
### k8sdiy-api:599e1af

### QA трафік → нова версія
curl -H "X-QA: 1" http://localhost:8080/api/
### k8sdiy-api:802e329

## Результат

Перша проблема ("ContainerCreating") не відтворилась — усі поди працювали.

Нова версія API (802e329) розгорнута і доступна тільки QA-команді (через заголовок).

Публічний трафік лишився на старій версії (599e1af).
