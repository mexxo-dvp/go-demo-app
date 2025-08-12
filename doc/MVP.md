## MVP

## Demo app працює так (Argo CD + Ambassador)

### Що ми змінили
1) Helm only (ArgoCD читає path: helm)

Що: прибрали розкидані YAML-маніфести поза чартом (Ingress/mappings у k8s/...) і залишили єдине джерело правди — Helm-чарт у форку go-demo-app.
Навіщо: ArgoCD синхронізує тільки те, що під spec.source.path: helm. Будь-які файли поза цим шляхом він ігнорує → виникає дублі/дрейф.
Як перевірити: у UI ArgoCD у вкладці Manifest бачиш лише те, що рендерить Helm; будь-які «голі» k8s-yaml з репо більше не застосовуються.
2) Ambassador (gateway) → оновлення образу

Що: замінили образ з дуже старого quay.io/datawire/ambassador:0.51.2 на робочий 1.14.x (Docker Hub).
Навіщо: 0.51.2 використовує Docker manifest v1 (prettyjws), який сучасний containerd не тягне → not implemented: media type .... Версії 1.14.x з маніфестом v2/OCI тягнуться без проблем.
Де змінили: helm/values.yaml →
```yaml
api-gateway:
  image:
    tag: 1.14.4
```
Як перевірити: kubectl -n demo get deploy ambassador -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}' → бачиш тег 1.14.4. Pod у Running без ImagePullBackOff.
3) Порти gateway: 8080 в контейнері ↔ 80 у сервісі

Що: вирівняли сервіс так, щоб він мапився на реальний порт процесу в контейнері Ambassador.

    Контейнер Ambassador слухає 8080.

    Сервіс ambassador віддає port: 80 → targetPort: 8080.
    Де змінили: helm/values.yaml →
```yaml
api-gateway:
  service:
    type: NodePort
    port: 80
    targetPort: 8080
```
Навіщо: інакше kubectl port-forward svc/ambassador 8088:80 падає з connection refused.
Як користуватись: для стабільності портфорвардимо деплоймент (бо точно 8080):
kubectl -n demo port-forward deploy/ambassador 8088:8080.
4) Маршрути (роутинг) через анотації Helm — app.version: v3

Що: переключили app.version з v4 на v3.
Навіщо: у шаблонах чарта є умова {{ if eq .Values.app.version "v3" }} — тільки при v3 чарт додає до сервісів анотації getambassador.io/config з Mapping-ами для /api/, /img/, /ascii/. При v4 ці анотації не рендерилися, тож Ambassador не знав, куди роутити.
Де змінили: helm/values.yaml →
```yaml
app:
  version: v3
```
### Як перевірити локально
> Вважаємо, що кластер уже має namespace `demo`, а ArgoCD Application вказує на цей форк (`path: helm`, `main`).

```bash
# 1) Port-forward до Ambassador (він слухає 8080)
kubectl -n demo port-forward deploy/ambassador 8088:8080
```
# 2) Ping API
```bash
curl -s http://localhost:8088/api/ | head -n1
```
# 3) Тестове зображення
```bash
wget -O /tmp/g.png https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png
```
# 4) Зображення -> ASCII (endpoint /img/)
```bash
curl -s -F "image=@/tmp/g.png" http://localhost:8088/img/ -o /tmp/ascii.txt
file /tmp/ascii.txt && head -n 20 /tmp/ascii.txt
```

### Демо роботи застосунку
   посилання та Демо авто-синхронізації: 
[![Дивитися відео](https://www.loom.com/share/e1ff9d94580842e8b900ce42f6ab35d5?sid=cbfedf14-0a19-4625-afd1-e941449998e2/0.jpg)](https://www.loom.com/share/e1ff9d94580842e8b900ce42f6ab35d5?sid=cbfedf14-0a19-4625-afd1-e941449998e2)