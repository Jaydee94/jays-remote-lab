# Remote VPS Lab

Ansible-Projekt zum Verwalten des VPS `85.215.129.251`.  
Jeder Playbook-Run hält Ubuntu und alle installierten Tools automatisch aktuell.

---

## Was wird eingerichtet?

| Tool | Beschreibung | Quelle |
|---|---|---|
| **Ubuntu** | System-Updates (apt dist-upgrade) | apt |
| **podman** | Container-Runtime (Docker-Alternative) | apt |
| **starship** | Schneller, hübscher Shell-Prompt | GitHub Releases |
| **llama.cpp** | Lokale KI-Inferenz (LLMs lokal laufen lassen) | GitHub Releases |
| **opencode** | KI-gestützter Coding-Assistent im Terminal | GitHub Releases |
| **paperclip** | Clipboard-/Snippet-Tool | GitHub Releases |

---

## Voraussetzungen

Auf deinem lokalen Rechner (nicht auf dem VPS):

1. **Ansible** installiert (`pip install ansible` oder Paketmanager)
2. **SSH-Zugang** zum VPS funktioniert – teste es kurz:
   ```bash
   ssh jaydee@85.215.129.251
   ```

---

## Loslegen

### 1. Repo klonen
```bash
git clone <repo-url>
cd jays-remote-lab
```

### 2. Verbindung testen
```bash
ansible all -m ping
```
Erwartete Ausgabe: `vps | SUCCESS`

### 3. Playbook ausführen
```bash
ansible-playbook site.yml
```

Das war's. Ansible kümmert sich um den Rest:
- Ubuntu wird auf den neuesten Stand gebracht
- Alle Tools werden installiert oder auf die aktuellste Version aktualisiert

---

## Struktur

```
.
├── ansible.cfg            # Ansible-Konfiguration (Inventory, SSH-User …)
├── inventory/
│   └── hosts.yml          # VPS-IP und Verbindungsparameter
├── group_vars/
│   └── all.yml            # Gemeinsame Variablen (User, Pfade …)
├── site.yml               # Haupt-Playbook – startet alle Rollen
└── roles/
    ├── common/            # apt update + upgrade + Basis-Pakete
    ├── podman/            # Container-Runtime
    ├── starship/          # Shell-Prompt
    ├── llama_cpp/         # Lokale LLM-Inferenz
    ├── opencode/          # AI Coding Assistant
    └── paperclip/         # Clipboard-Tool
```

---

## Einzelne Rollen ausführen

Möchtest du nur ein bestimmtes Tool aktualisieren, nutze Tags über `--tags` oder führe nur eine Rolle aus:

```bash
# Nur System-Updates
ansible-playbook site.yml --tags common

# Nur starship aktualisieren
ansible-playbook site.yml --limit vps -e "ansible_run_tags=starship"
```

Oder starte den Playbook einfach komplett – er ist idempotent und ändert nur, was sich geändert hat.

---

## Anpassungen

### Anderer SSH-User oder SSH-Key

Passe `inventory/hosts.yml` an:
```yaml
all:
  hosts:
    vps:
      ansible_host: 85.215.129.251
      ansible_user: meinuser
      ansible_ssh_private_key_file: ~/.ssh/mein_key
```

### Tool-Versionen / GitHub-Repos ändern

Jede Rolle hat eine `defaults/main.yml` mit konfigurierbaren Variablen.  
Beispiel für llama.cpp (`roles/llama_cpp/defaults/main.yml`):

```yaml
llama_cpp_github_owner: ggml-org
llama_cpp_github_repo: llama.cpp
llama_cpp_install_dir: /usr/local/bin
```

### Paperclip – GitHub-Repo anpassen

Sollte das Standard-Repo für paperclip nicht passen, einfach in  
`roles/paperclip/defaults/main.yml` das richtige Repo eintragen:

```yaml
paperclip_github_owner: <org-oder-user>
paperclip_github_repo: <repo-name>
paperclip_asset_pattern: "linux.*amd64"
```

### Neues Tool hinzufügen

1. Neue Rolle anlegen: `mkdir -p roles/mein_tool/{defaults,tasks}`
2. `roles/mein_tool/tasks/main.yml` nach dem Muster der anderen Rollen schreiben
3. Rolle in `site.yml` einbinden:
   ```yaml
   roles:
     - ...
     - mein_tool
   ```

---

## Wie funktioniert die automatische Aktualisierung?

Für Tools aus GitHub Releases:
1. Ansible fragt beim jeweiligen GitHub-Release-Endpunkt die aktuelle Version ab.
2. In `/usr/local/share/ansible-versions/` wird die zuletzt installierte Version gespeichert.
3. Weichen die Versionen voneinander ab, wird die neue Version heruntergeladen und installiert.

Für apt-Pakete (Ubuntu, podman) wird bei jedem Lauf `apt dist-upgrade` ausgeführt.

---

## Häufige Fragen

**Der Playbook-Run schlägt mit „Permission denied" fehl.**  
→ Prüfe, ob `jaydee` passwordlos sudo ausführen kann:  
```bash
ssh jaydee@85.215.129.251 "sudo id"
```
Falls nicht, füge den User zur sudoers hinzu:  
```bash
echo "jaydee ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jaydee
```

**Ich sehe „No Ubuntu x64 asset found for llama.cpp".**  
→ llama.cpp hat den Asset-Namen geändert. Öffne  
`roles/llama_cpp/tasks/main.yml` und passe das `search`-Muster beim  
Task „Find Ubuntu x64 binary asset" an.

**Starship wird installiert, aber ich sehe es nicht im Terminal.**  
→ Starte eine neue SSH-Session oder führe `source ~/.bashrc` aus.
