# AWX Example Inventory

Ten katalog zawiera przykładowe inventory dla AWX/Ansible Tower.

## Struktura

```
awx-example/
├── hosts.yml                      # Główny plik inventory z definicjami hostów i grup
├── group_vars/                    # Zmienne dla grup hostów
│   ├── all.yml                    # Zmienne globalne dla wszystkich hostów
│   ├── webservers.yml            # Zmienne dla serwerów WWW
│   ├── dbservers.yml             # Zmienne dla serwerów baz danych
│   ├── development.yml           # Zmienne dla środowiska deweloperskiego
│   └── production.yml            # Zmienne dla środowiska produkcyjnego
├── host_vars/                     # Zmienne specyficzne dla poszczególnych hostów
│   ├── web01.yml                 # Zmienne dla hosta web01
│   └── db01.yml                  # Zmienne dla hosta db01
└── README.md                     # Ten plik

```

## Grupy Hostów

### webservers
Serwery WWW obsługujące aplikacje webowe:
- **web01** (192.168.1.10) - Główny serwer WWW
- **web02** (192.168.1.11) - Dodatkowy serwer WWW

### dbservers
Serwery baz danych:
- **db01** (192.168.1.20) - Główna baza danych (master)
- **db02** (192.168.1.21) - Baza danych replika

### loadbalancers
Load balancery:
- **lb01** (192.168.1.30) - Load balancer

### development
Środowisko deweloperskie:
- **dev-web01** (192.168.2.10) - Deweloperski serwer WWW
- **dev-web02** (192.168.2.11) - Dodatkowy deweloperski serwer WWW
- **dev-db01** (192.168.2.20) - Deweloperska baza danych

### production
Środowisko produkcyjne (zawiera webservers, dbservers, loadbalancers)

## Użycie w AWX

### Import Inventory do AWX

1. **Przez GUI:**
   - Przejdź do Resources → Inventories
   - Kliknij "Add" → "Add inventory"
   - Wypełnij nazwę i opis
   - Po utworzeniu inventory, dodaj source typu "Sourced from a Project"
   - Wskaż projekt z tym repozytorium
   - W polu "Inventory file" wskaż: `inventory/awx-example/hosts.yml`

2. **Przez AWX CLI:**
   ```bash
   awx inventory create --name "AWX Example" --organization "Default"
   awx inventory_source create --name "Git Source" \
     --inventory "AWX Example" \
     --source scm \
     --source_project "Your Project Name" \
     --source_path "inventory/awx-example/hosts.yml"
   ```

### Testowanie Inventory

Testowanie połączenia z hostami:
```bash
ansible all -i inventory/awx-example/hosts.yml -m ping
```

Wyświetlenie zmiennych dla konkretnego hosta:
```bash
ansible-inventory -i inventory/awx-example/hosts.yml --host web01
```

Wyświetlenie wszystkich hostów w grupie:
```bash
ansible-inventory -i inventory/awx-example/hosts.yml --graph
```

## Konfiguracja Credentials w AWX

Dla tego inventory potrzebujesz skonfigurować credentials w AWX:

1. **Machine Credential** dla SSH:
   - Typ: Machine
   - Username: admin (lub developer dla dev)
   - SSH Private Key: [Twój klucz prywatny]
   - Privilege Escalation Method: sudo

2. **Vault Credential** (opcjonalnie, dla zaszyfrowanych zmiennych):
   - Typ: Vault
   - Vault Password: [Twoje hasło vault]

## Zmienne

### Zmienne Globalne (group_vars/all.yml)
- Konfiguracja SSH
- Ustawienia systemowe
- Bezpieczeństwo
- DNS i NTP

### Zmienne Specyficzne dla Grup
- **webservers**: Konfiguracja Nginx, SSL, monitoring
- **dbservers**: Konfiguracja PostgreSQL, backup, replikacja
- **development**: Ustawienia deweloperskie, debug mode
- **production**: Ustawienia produkcyjne, HA, monitoring

### Zmienne Specyficzne dla Hostów
Zawierają unikalne informacje o konkretnych serwerach:
- Lokalizacja w datacenter
- Specyfikacja sprzętowa
- Role w klastrze

## Dostosowanie

Aby dostosować to inventory do swoich potrzeb:

1. Zmień adresy IP w `hosts.yml` na właściwe dla Twojej infrastruktury
2. Dostosuj zmienne w `group_vars/` do swoich wymagań
3. Dodaj lub usuń hosty według potrzeb
4. Zaktualizuj credentials w AWX
5. Zsynchronizuj inventory source w AWX

## Bezpieczeństwo

⚠️ **Uwaga**: Ten przykład zawiera niezaszyfrowane zmienne. W środowisku produkcyjnym:

1. Użyj Ansible Vault do zaszyfrowania wrażliwych danych:
   ```bash
   ansible-vault encrypt group_vars/production.yml
   ```

2. Przechowuj hasła i klucze w AWX Credentials, nie w plikach inventory

3. Używaj AWX RBAC do kontroli dostępu do inventory

## Przykładowe Playbooki

Przykłady użycia tego inventory w playbook'ach znajdziesz w katalogu `playbooks/`.

---

**Uwaga**: To jest przykładowe inventory. Przed użyciem w produkcji, dostosuj adresy IP, credentials i inne parametry do swojego środowiska.
