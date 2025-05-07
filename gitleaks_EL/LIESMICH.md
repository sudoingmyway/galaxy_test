# Ansible Role: Gitleaks für RHEL 9 und Derivate

[[_TOC_]]

## Beschreibung

Diese Ansible-Rolle installiert Gitleaks, einen schnellen und sicheren Git-Secrets-Scanner, entweder im Offline- oder Online-Modus, basierend auf der Konfiguration. Die Rolle stellt sicher, dass die angegebene Version von Gitleaks installiert ist und überprüft optional die Installation mittels Checksum-Validierung im Offline-Modus.

Sie unterstützt die Installation auf Red Hat Enterprise Linux 9 und seinen Derivaten, einschließlich AlmaLinux 9, Rocky Linux 9 und Oracle Linux 9.

## Voraussetzungen

- Ansible 2.12 oder höher
- Internetzugang (für Online-Installation) oder eine vorher heruntergeladene Gitleaks-Binärdatei (für Offline-Installation)
- Root-Rechte oder `become: true` Zugriff ist erforderlich, da die Rolle Gitleaks in `/usr/local/bin` installiert und systemweite Operationen durchführt
- Eines der folgenden Betriebssysteme:
  - Red Hat Enterprise Linux 9
  - AlmaLinux 9
  - Rocky Linux 9
  - Oracle Linux 9

## Aufgabenübersicht

1. **Überprüfung der Gitleaks-Installation:** Prüft, ob die gewünschte Version von Gitleaks bereits auf dem System installiert ist.

2. **Installation von Gitleaks (Offline-Modus):** Installiert die Gitleaks-Binärdatei aus einer vorhandenen Datei (vom Benutzer bereitgestellt) im Offline-Modus.

3. **Installation von Gitleaks (Online-Modus):** Lädt Gitleaks von den offiziellen GitHub-Releases im Online-Modus herunter und installiert es.

4. **Überprüfung der Gitleaks-Installation:** Verifiziert, dass die installierte Gitleaks-Binärdatei der gewünschten Version entspricht.

5. **Bereinigung temporärer Dateien (Online-Modus):** Bereinigt alle temporären Dateien, die während des Installationsprozesses im Online-Modus erstellt wurden.

## Rollen-Variablen

Die Rolle hat die folgenden [Standard](./defaults/main.yml)-Variablen, die vor der Verwendung der Rolle gemäß den Anforderungen überprüft werden müssen:

Variable                  | Standardwert | Beschreibung
------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------
`configure_offline`       | `false`      | Wenn `true`, wird eine vorgefertigte Gitleaks-Binärdatei [gitleaks](./files/gitleaks) für Version 8.25.0 verwendet. Sie können auch Ihre eigene Binärdatei bereitstellen und den erwarteten Checksum mit `offline_gitleaks_sha256` festlegen. Wenn `false`, wird Gitleaks von GitHub heruntergeladen.
`gitleaks_version`        | `"8.25.0"`  | Zu installierende Gitleaks-Version, wenn `configure_offline` auf `false` gesetzt ist.
`gitleaks_dest`           | `"/usr/local/bin/gitleaks"` | Endgültiger Installationspfad für die Gitleaks-Binärdatei.
`gitleaks_download_url`   | `"https://github.com/gitleaks/gitleaks/releases/download/v{{ gitleaks_version }}/gitleaks_{{ gitleaks_version }}_linux_x64.tar.gz"` | URL zum Herunterladen von Gitleaks von GitHub (im Online-Modus verwendet).
`gitleaks_tmp_dir`        | `"{{ ansible_env.HOME }}/.gitleaks_tmp_install"` | Temporäres Verzeichnis für die Online-Installation.
`offline_gitleaks_sha256` | `"a4b6ec120e0d4da50370e7ae64d0ef17f2d120d0f6143931a3061e3992f00565"` | SHA256-Checksum zur Überprüfung der Gitleaks-Binärdatei bei der Offline-Installation.

## Abhängigkeiten

Keine.

## Rollenverwendung

Diese Ansible-Rolle unterstützt sowohl Offline- als auch Online-Installationsmodi, je nachdem, ob das Zielsystem Internetzugang hat. Um das Verhalten anzupassen, setzen Sie die entsprechenden Variablen wie unten gezeigt.

### 1. Offline-Modus

Im Offline-Modus müssen Sie die Gitleaks-Binärdatei und deren SHA256-Checksum zur Überprüfung bereitstellen. Verwenden Sie die Variable `configure_offline` und setzen Sie sie auf true.

Beispielverwendung in einem Playbook:

```yaml
- name: Gitleaks im Offline-Modus installieren
  hosts: all
  become: true
  roles:
    - role: gitleaks_EL
      vars:
        configure_offline: true
        gitleaks_dest: "/usr/local/bin/gitleaks"
        offline_gitleaks_sha256: "a4b6ec120e0d4da50370e7ae64d0ef17f2d120d0f6143931a3061e3992f00565" # Checksum ändert sich, wenn Sie die Binärdatei im files-Verzeichnis ändern
```
Stellen Sie sicher, dass die Gitleaks-Binärdatei im Verzeichnis files/ verfügbar ist.

### 2. Online-Modus

Beispielverwendung in einem Playbook:

```yaml
- name: Gitleaks im Online-Modus installieren
  hosts: all
  become: true
  roles:
    - role: gitleaks_EL
      vars:
        configure_offline: false
        gitleaks_version: "8.25.0"
```

## Tests

Diese Rolle enthält sowohl Molecule-Tests als auch Smoke-Tests.

### Molecule-Tests

Diese Rolle enthält Molecule-Tests für sowohl den Offline- als auch den Online-Modus. Die Testsuite umfasst:
- Syntax-Überprüfung
- Rollen-Anwendung
- Idempotenz-Test
- Überprüfung, dass Gitleaks installiert ist und funktioniert

#### Tests ausführen (Offline-Modus)

Um die Rolle im Offline-Modus mit Molecule zu testen, stellen Sie sicher, dass die Gitleaks-Binärdatei im Verzeichnis `files/` platziert ist.

Führen Sie den folgenden Befehl zum Testen aus:

```bash
molecule test -s offline
```

#### Tests ausführen (Online-Modus)

Um die Rolle im Online-Modus mit Molecule zu testen, führen Sie den folgenden Befehl aus:

```bash
molecule test -s online
```

### Smoke-Tests

Ein einfacher Smoke-Test ist im Verzeichnis `tests` verfügbar. Um ihn auf dem localhost (standardmäßig in der Inventory definiert) auszuführen, verwenden Sie die folgenden Befehle:

```bash
cd tests
ansible-playbook test.yml -K
```
Die `-K`-Flag weist Ansible an, nach dem sudo-Passwort zu fragen.

Wenn Sie bestimmte Hosts als Ziel haben möchten, sollten Sie ein Inventory mit der `-i`-Flag wie folgt übergeben:

```bash
cd tests
ansible-playbook -i inventory test.yml -K
```

Der Smoke-Test überprüft:
- Rollen-Anwendung
- Gitleaks-Installation
- Gitleaks-Versionsüberprüfung

⚠️ **Wichtiger Hinweis zu Smoke-Tests** ⚠️

Bei der Ausführung von Smoke-Tests auf Ihrem lokalen Computer, wie z.B. bei Verwendung von localhost als Zielhost, kann die Rolle Änderungen an Ihrem lokalen System vornehmen. Diese Änderungen umfassen:
- Installation oder Aktualisierung von Gitleaks
- Änderung der Berechtigungen für /usr/local/bin/gitleaks
- Erstellung von Verzeichnissen oder Kopieren von Dateien

Wenn Sie die Rolle auf Ihrem lokalen Computer testen (z.B. mit localhost), beachten Sie bitte, dass diese Änderungen direkt an Ihrem System vorgenommen werden.

## Lizenz

Apache-2.0

## Autor-Information

Diese Rolle wurde vom NDAAL-Team, Pierre Gronau, Ayesha Shafqat erstellt.
