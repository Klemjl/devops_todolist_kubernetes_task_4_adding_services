# INSTRUCTION.md

## Apply Manifests

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/clusterIP.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/busybox.yml
```

Перевірка стану:
```bash
kubectl get pods -n todoapp
kubectl get services -n todoapp
```

---

## 1. Тестування через ClusterIP Service (DNS з busybox контейнера)

ClusterIP-сервіс доступний лише всередині кластера. Зайдіть у under `busybox`, який знаходиться в тому ж namespace `todoapp`:

```bash
kubectl exec -it busybox -n todoapp -- sh
```

Усередині контейнера виконайте запит за DNS-іменем сервісу:
```sh
curl http://todoapp-service.todoapp.svc.cluster.local/api/health
```

Або коротшою формою (працює в межах того ж namespace):
```sh
curl http://todoapp-service/api/health
```

Kubernetes автоматично розподілятиме запити між обома pod'ами (`todoapp` та `todoapp-1`).

---

## 2. Тестування через `port-forward` сервісу

Прокиньте локальний порт на ClusterIP-сервіс:
```bash
kubectl port-forward service/todoapp-service 8080:80 -n todoapp
```

У **другому терміналі** виконайте:
```bash
curl http://localhost:8080/api/health
```

Або відкрийте `http://localhost:8080` у браузері.

Щоб зупинити тунель — натисніть `Ctrl+C`.

---

## 3. Доступ до застосунку через NodePort Service

NodePort-сервіс відкриває застосунок на порту **30007** кожної ноди кластера.

На Docker Desktop NodePort доступний напряму через `localhost`:
```bash
curl http://localhost:30007/api/health
```

Або відкрийте `http://localhost:30007` у браузері.

На реальному/хмарному кластері спершу дізнайтесь IP ноди:
```bash
kubectl get nodes -o wide
```

Потім використайте `INTERNAL-IP` або `EXTERNAL-IP`:
```bash
curl http://<NODE_IP>:30007/api/health
```

---

## Cleanup

```bash
kubectl delete -f .infrastructure/
```