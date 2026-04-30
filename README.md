# Ansible DHCP Server

Automatyzacja uruchomienia **serwera DHCP** na Linuxie w kilka minut — bez ręcznego klikania, bez rozjazdów konfiguracji między środowiskami i bez przepisywania tych samych kroków na każdym hoście.

Ten projekt zawiera gotowy playbook Ansible, który:
- konfiguruje interfejsy sieciowe,
- instaluje odpowiednie pakiety DHCP dla danej dystrybucji,
- generuje konfigurację serwera DHCP z szablonu,
- ustawia nasłuch na właściwym interfejsie,
- uruchamia usługę.

## Dlaczego warto wdrożyć ten projekt?

- **Szybki start** – uruchamiasz jeden playbook i masz działającą usługę DHCP.
- **Wielodystrybucyjność** – obsługa rodzin: **Debian/Ubuntu**, **RedHat** oraz **openSUSE Leap**.
- **Powtarzalność** – ta sama konfiguracja w labie, testach i produkcji.
- **Łatwa parametryzacja** – adresację i interfejsy zmieniasz w jednym pliku `vars/vars.yml`.
- **Infrastructure as Code** – pełna transparentność zmian i wygodna wersjonowalność w Git.

## Co dokładnie robi playbook?

1. Wczytuje zmienne z `vars/vars.yml`.
2. Konfiguruje sieć:
   - na RedHat/openSUSE przez `nmcli`,
   - na Debian/Ubuntu przez szablon Netplan + `netplan apply`.
3. Instaluje serwer DHCP odpowiednim menedżerem pakietów (`apt`, `yum`, `zypper`).
4. Renderuje plik `dhcpd.conf` z `templates/dhcp_conf.j2`.
5. Ustawia interfejs nasłuchu DHCP:
   - Debian: `/etc/default/isc-dhcp-server`,
   - RedHat: modyfikacja pliku usługi,
   - openSUSE: `/etc/sysconfig/dhcpd`.
6. Startuje usługę DHCP (`dhcpd` lub `isc-dhcp-server` zależnie od systemu).

## Wymagania

- Ansible na maszynie sterującej.
- Kolekcja `community.general` (używana przez moduł `nmcli`).
- Dostęp SSH do hosta docelowego.
- Uprawnienia `root` / `become: yes`.
- Co najmniej 2 interfejsy na hoście docelowym (domyślnie `eth0` i `eth1`).

## Szybkie uruchomienie

### 1) Sklonuj repozytorium

```bash
git clone <URL_REPO>
cd ansible-dhcp-server
```

### 2) Dostosuj zmienne

Edytuj `vars/vars.yml` i ustaw:
- interfejsy (`interfaces`),
- adresację (`ipv4`, `subnet`, `netmask`),
- ewentualnie nazwy usług i ścieżki pod swoją dystrybucję.

### 3) Przygotuj inventory

Przykład `inventory.ini`:

```ini
[dhcp]
192.168.56.10 ansible_user=root
```

### 4) Uruchom playbook

```bash
ansible-playbook -i inventory.ini main.yml
```

## Domyślna konfiguracja DHCP (z szablonu)

Szablon `templates/dhcp_conf.j2` tworzy pulę:
- sieć: `10.20.30.0/24`,
- zakres: `10.20.30.100 - 10.20.30.150`,
- gateway i DNS: `10.20.30.1`.

Jeśli chcesz inną pulę, zmień `subnet` i `netmask` w `vars/vars.yml`.

## Struktura projektu

```text
.
├── main.yml
├── vars/
│   └── vars.yml
├── templates/
│   ├── dhcp_conf.j2
│   ├── dhcp_listener.j2
│   └── ubuntu_netplan.j2
└── README.md
```

## Dla kogo jest ten projekt?

- dla administratorów, którzy chcą szybko postawić DHCP w labie,
- dla zespołów DevOps, które chcą ustandaryzować konfigurację sieci,
- dla osób uczących się Ansible i automatyzacji usług systemowych.

## Co można łatwo rozbudować dalej?

- walidację konfiguracji przed restartem usługi,
- obsługę wielu podsieci/VLAN,
- automatyczne testy po wdrożeniu (np. sprawdzenie lease’ów),
- wydzielenie projektu do roli Ansible.

---

Jeśli chcesz, mogę w kolejnym kroku przygotować też wersję „production-ready”: z handlerami, idempotentną obsługą usługi i rollbackiem konfiguracji.
