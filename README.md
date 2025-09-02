# FortiGate SNMP Lab — Prometheus + Grafana + snmp_exporter + Ansible

Лабораторний стенд для моніторингу **FortiGate** через **SNMP** із використанням **Prometheus**, **Grafana** та **snmp_exporter**.
Плейбук **Ansible** вмикає SNMP (v2c або v3), створює community та (опційно) додає `snmp` в `allowaccess` на керуючому інтерфейсі.

> Репозиторій зібрано під таку структуру:
>
> ```
> .
> ├─ ansible/
> │  ├─ playbook.yml
> │  ├─ inventory.ini
> │  ├─ group_vars/all.yml
> │  └─ .env                 # локальні секрети (НЕ комітити)
> └─ monitoring/
>    ├─ docker-compose.yml
>    ├─ prometheus/prometheus.yml
>    ├─ snmp_exporter/snmp.yml
>    ├─ snmp_exporter/snmp_auth.yml
>    └─ grafana/
>       ├─ provisioning/datasources/datasource.yml
>       ├─ provisioning/dashboards/dashboards.yml
>       └─ dashboards/fortigate-interfaces.json
> ```

---

## Передумови

- Docker + Docker Compose v2
```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Ansible 2.16+
```
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
```  
- Доступ до FortiGate (адмін/REST API admin)

---

## 1) Створити **API-токен** FortiGate (для Ansible)

**GUI:** *System → Administrators → Create New → REST API Admin* → виставити права/Trusted Hosts(Вказуємо наш Ansible) → зберегти й скопіювати **API Token**.  
**CLI (альтернатива):**
```shell
config system api-user
    edit "ansible"
        set accprofile super_admin
        config trusthost
            set ipv4-trusthost1 10.0.0.0 255.255.255.0
        end
    next
end
execute api-user generate-key ansible   # надрукує токен
```
## 2) Завантажуємо репозиторій на сервер і розпаковуємо його
## 3) Створити локальний ansible/.env
```
FGT_API_TOKEN=ВАШ ТОКЕН з п.1
```
## 4) В файлі inventory.ini
Змінюємо в рядку IP на IP адресу свого Fortigate
```
[fortigates]
fgt01 ansible_host=<IP> vdom=root
```
## 5) Запускаємо наш плейбук Ansible 
```
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```
## 6) Після того як все відпрацювало, можемо запускати наш докер композ для моніторингу
```
cd monitoring
docker compose up -d
```
Доступи за замовчуванням:

Prometheus — http://<host>:9090
snmp_exporter — http://<host>:9116/metrics
Ручна перевірка — http://<host>:9116/snmp?module=if_mib&auth=${SNMP_AUTH_NAME}&target=${FGT_HOST}
Grafana — http://<host>:3000

(c) Dr. Helm 2025
