DevOps Monitoring Stack (Project X)

## Описание проекта

В рамках проекта была развернута базовая DevOps-инфраструктура, включающая мониторинг, логирование, алертинг, автоматизацию и ChatOps.

Цель — продемонстрировать полный цикл работы с инфраструктурой:
сбор метрик, централизованное логирование, визуализация, алертинг и управление через чат-бота.

---

## Инфраструктура

На личном ноутбуке в VirtualBox развернуты 3 виртуальные машины:

- VM1 — RHEL 9.6 — Monitoring (Prometheus, Grafana, Loki)
- VM2 — RHEL 9.7 — Managment (Kubernetes (k3s control-plane))
- VM3 — RHEL 9.4 — target node (логи и тестовая нагрузка)

Сетевые адреса:

- VM1: 192.168.56.20
- VM2: 192.168.56.21
- VM3: 192.168.56.22

Используемые технологии:

- Kubernetes (k3s)
- Terraform
- Prometheus
- Grafana
- Loki + Promtail / Alloy
- Node Exporter
- Telegram Bot API (ChatOps)
- Ansible (AWX роли и playbooks)

---

## Дисковая разметка

На всех ВМ были выделены отдельные разделы:

- /var/log — для хранения логов
- /home — пользовательские данные

Это позволяет изолировать логи и избежать переполнения root-раздела.

---

## Мониторинг

### Prometheus + Node Exporter

На всех ВМ установлен Node Exporter для сбора метрик:

- CPU
- RAM
- Disk
- Network

Prometheus развернут на VM1 и собирает метрики со всех узлов.

---

### Grafana

Grafana развернута на VM1.

Используется дашборд:

- Node Exporter Full

Реализована визуализация:

- загрузки CPU
- использования RAM
- дисков
- сетевого трафика

---

## Логирование

### Loki

Loki развернут на VM1.

С VM2 и VM3 в Loki отправляются системные логи (/var/log/messages).

Пример запроса:

```logql
{job="system"} |~ "(?i)error"
Алертинг

В Grafana настроены алерты на основе Loki:

поиск ошибок (ERROR / FAILED / FATAL)
отсутствие данных (No Data)

При срабатывании алерта отправляется уведомление в Telegram.

ChatOps

Реализован Telegram-бот для работы с логами.

Функциональность:

получение последних ошибок
просмотр логов
фильтрация по хостам

Доступные команды:

/start — проверка работоспособности бота
/errors — последние ERROR логи
/logs — последние system логи
/logs VM2 — логи VM2
/logs VM3 — логи VM3

Бот реализован на Python и работает как systemd сервис.

## Kubernetes (k3s)

### На VM2 развернут кластер k3s (control-plane).

Проверка:

kubectl get nodes

Результат:

localhost.localdomain   Ready   control-plane
Terraform + Kubernetes

С помощью Terraform выполнено управление Kubernetes ресурсами:

Создано:

Namespace
Deployment (nginx)
Service (NodePort)

Проверка:

kubectl get pods -n test-terraform
kubectl get svc -n test-terraform

Доступ к приложению:

http://192.168.56.21:30007
AWX + Ansible

AWX развернут на одной из ВМ.

Реализовано:

подключение Git-репозитория
выполнение playbooks

Создан playbook для:

создания локальных пользователей
Git интеграция

Используются репозитории:

https://github.com/Aleksrootik/awx-test
https://github.com/Aleksrootik/awx-backups

В них размещены:

playbooks
роли
конфигурации мониторинга
backup сценарии
Бэкапы

Через AWX реализованы бэкапы:

конфигураций
данных

Бэкапы сохраняются в отдельный Git-репозиторий.

## Loki + Telegram

### Grafana Alerting интегрирован с Telegram.

Реализовано:

отправка алертов при появлении ошибок в логах
уведомления о состоянии (firing/resolved)

---------

## Основные проблемы и решения

1. kubectl работает только под root

Ошибка:

permission denied /etc/rancher/k3s/k3s.yaml

Решение:

mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
2. Ошибка Loki API (parse error)

Причина:

неправильный HTTP-запрос к Loki API

Решение:

использование корректных параметров запроса (query_range + параметры времени)

3. Terraform конфликт namespace

Ошибка:

namespace already exists

Причина:

ресурс уже существует вне Terraform state

Решение:

удалить ресурс вручную
или
импортировать в Terraform state
4. Telegram бот не работал после выхода из сессии

Причина:

бот запускался вручную

Решение:

создан systemd сервис:

/etc/systemd/system/chatops-bot.service

Бот теперь автоматически запускается и работает после перезагрузки.

Итог

В рамках проекта реализовано:

развертывание инфраструктуры на виртуальных машинах
настройка мониторинга (Prometheus + Grafana)
централизованное логирование (Loki)
алертинг в Telegram
ChatOps бот для работы с логами
управление Kubernetes через Terraform
автоматизация через AWX (Ansible)
организация бэкапов через Git

Проект демонстрирует базовые навыки работы DevOps-инженера с реальным стеком технологий.

Возможные улучшения
настройка Ingress и домена
использование Helm
CI/CD (GitHub Actions / GitLab CI)
настройка Alertmanager
RBAC и безопасность Kubernetes
