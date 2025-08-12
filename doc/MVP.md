### MVP
## Робота демонстраційного застосунку (Argo CD + Ambassador)
# Внесені зміни

Оновлення образу Ambassador (gateway)
# Суть змін:
замінено застарілий образ quay.io/datawire/ambassador:0.51.2 на актуальний 1.14.x з Docker Hub.
# Причина:
версія 0.51.2 використовує Docker manifest v1 (prettyjws), який сучасний containerd не підтримує → помилка not implemented: media type .... Версії 1.14.x використовують manifest v2/OCI та працюють коректно.
# Місце змін:
helm/values.yaml
```yaml       
api-gateway:
  image:
    tag: 1.14.4
```
# Перевірка:
```yaml
kubectl -n demo get deploy ambassador \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```
Має відображатися тег 1.14.4, Pod у стані Running без ImagePullBackOff.

### Вирівнювання портів Gateway (8080 у контейнері ↔ 80 у сервісі)
# Суть змін:
сервіс налаштовано так, щоб він відповідав реальному порту процесу Ambassador у контейнері.

Контейнер слухає порт 8080.

Сервіс ambassador віддає port: 80 → targetPort: 8080.
Місце змін: helm/values.yaml
```yaml
api-gateway:
  service:
    type: NodePort
    port: 80
    targetPort: 8080
```
# Причина: 
уникнення помилок connection refused під час kubectl port-forward svc/ambassador 8088:80.
Рекомендація: для стабільної роботи виконувати портфорвардинг деплойменту:
```bash
kubectl -n demo port-forward deploy/ambassador 8088:8080
```
### Маршрутизація (routing) через анотації Helm — app.version: v3
# Суть змін: 
параметр app.version змінено з v4 на v3.
# Причина:
у шаблонах чарта умова
```go
{{ if eq .Values.app.version "v3" }}
```
забезпечує додавання анотацій getambassador.io/config з налаштуваннями Mapping для маршрутів /api/, /img/, /ascii/. Для v4 ці анотації не рендеряться, і Ambassador не має правил маршрутизації.
# Місце змін:
helm/values.yaml
```yaml
    app:
      version: v3
```
### Перевірка роботи локально

Передбачається, що кластер містить простір імен demo, а ArgoCD Application налаштований на відповідний форк (path: helm, гілка main).

### Port-forward до Ambassador (8080 у контейнері):
```bash
kubectl -n demo port-forward deploy/ambassador 8088:8080
```
### Перевірка API:
```bash
curl -s http://localhost:8088/api/ | head -n1
```
### Тестове зображення:
```bash
wget -O /tmp/g.png https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png
```
### Конвертація зображення в ASCII (endpoint /img/):
```bash
curl -s -F "image=@/tmp/g.png" http://localhost:8088/img/ -o /tmp/ascii.txt
file /tmp/ascii.txt && head -n 20 /tmp/ascii.txt
```
### Демо роботи застосунку
   посилання та Демо авто-синхронізації та роботи застосунку: 
[![Дивитися відео](https://www.loom.com/share/e1ff9d94580842e8b900ce42f6ab35d5?sid=cbfedf14-0a19-4625-afd1-e941449998e2/0.jpg)](https://www.loom.com/share/e1ff9d94580842e8b900ce42f6ab35d5?sid=cbfedf14-0a19-4625-afd1-e941449998e2)