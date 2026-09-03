# Remote-Desktop: iPad -> eigener Rechner

Planungsdokument. Ausgangslage: Zugriff nur ueber ein iPad, der Zielrechner
steht anderswo. Ziel: Bildschirm, Maus und Tastatur des Rechners vom iPad aus
nutzen (u.a. fuer EVE Online und dieses Tool).

## 1. Realitaets-Check

Was aus einer Claude-Code-Session heraus **nicht** geht:

- Die Session laeuft in einem kurzlebigen Cloud-Container ohne Netzwerkweg in
  dein Heim-/Buero-LAN. Ich kann auf dem Zielrechner nichts installieren.
- Es gibt keinen Weg, einen Rechner fernzusteuern, auf dem noch nie ein
  Fernwartungs-Dienst eingerichtet wurde. Der erste Schritt muss immer **auf
  dem Rechner selbst** passieren.

Daraus folgt die einzige echte Weiche:

**Gibt es auf dem Rechner schon irgendeinen Fuss in der Tuer?**
(SSH-Server, Tailscale, TeamViewer/AnyDesk mit unbeaufsichtigtem Zugriff,
aktiviertes RDP + VPN, Chrome Remote Desktop, macOS-Bildschirmfreigabe)

- **Ja** -> alles Weitere laesst sich vom iPad aus nachruesten (Abschnitt 5).
- **Nein** -> es braucht genau einmal 3 Minuten am Geraet: du selbst, wenn du
  zurueck bist, oder jemand vor Ort, den du telefonisch durch einen Installer
  lotst. Danach nie wieder.

## 2. Entscheidungsbaum

```
Laeuft schon SSH / Tailscale / TeamViewer auf dem Rechner?
├─ ja  -> Variante B (Tailscale + natives Remote-Desktop) headless nachruesten
└─ nein
   ├─ jemand kann kurz an den Rechner -> Variante A (schnellster Weg, 5 Min)
   └─ niemand kann an den Rechner     -> aktuell nicht loesbar; siehe Abschnitt 8
```

## 3. Die drei sinnvollen Varianten

| | A: Chrome Remote Desktop | B: Tailscale + RDP/VNC | C: Sunshine + Moonlight |
|---|---|---|---|
| Aufwand am Rechner | 1 Installer, 5 Min | 2 Installer, 10 Min | 1 Installer + Pairing |
| Netzwerk-Konfiguration | keine | keine (Mesh-VPN) | keine (ueber Tailscale) |
| Kosten | 0 EUR | 0 EUR (Client ggf. bezahlt) | 0 EUR |
| Latenz / Bildqualitaet | ok fuer Buerokram | gut | sehr gut, 60 fps |
| Fuer EVE spielbar | nein | eingeschraenkt | ja |
| Windows Home moeglich | ja | nur mit VNC statt RDP | ja |

**Empfehlung:** B als Dauerloesung (stabil, sicher, keine Fremd-Relays),
A als Notnagel/erste Bruecke, C zusaetzlich, wenn du EVE wirklich *spielen*
und nicht nur den Markt-Scanner bedienen willst.

## 4. Variante A - Chrome Remote Desktop (schnellster Weg)

Am Rechner (einmalig, GUI noetig):

1. In Chrome bei dem Google-Konto anmelden, das du auch auf dem iPad hast.
2. `remotedesktop.google.com/access` -> "Fernzugriff einrichten" -> Host-Paket
   installieren.
3. Rechnernamen vergeben und eine **6-stellige PIN** setzen.
4. Energieoptionen: Standby aus (siehe Abschnitt 7), sonst ist der Rechner
   nach 30 Minuten weg.

Am iPad: `remotedesktop.google.com/access` in Safari, oder die
"Chrome Remote Desktop"-App aus dem App Store (die App ist von Google
laenger nicht gepflegt worden - falls sie zickt, den Browserweg nehmen).

Grenzen: Maus-/Tastaturunterstuetzung im Browser ist mittelmaessig, kein
Sound, Bildrate niedrig. Als Bruecke zum Nachruesten von B aber perfekt.

## 5. Variante B - Tailscale + natives Remote-Desktop (Dauerloesung)

Tailscale ist ein WireGuard-Mesh-VPN: beide Geraete melden sich beim selben
Konto an und sehen sich danach direkt, ohne Portfreigabe, ohne DynDNS, ohne
offene Ports im Internet.

### 5.1 Am Rechner

Windows:
```powershell
winget install --id tailscale.tailscale -e
# danach einmal anmelden (GUI oder: tailscale up)
```

macOS:
```bash
brew install --cask tailscale
```

Linux:
```bash
curl -fsSL https://tailscale.com/install.sh | sh && sudo tailscale up
```

### 5.2 Remote-Desktop-Dienst aktivieren

**Windows 11/10 Pro** (RDP ist eingebaut, Home hat *keinen* RDP-Server):
```powershell
# als Administrator
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name fDenyTSConnections -Value 0
Enable-NetFirewallRule -DisplayGroup "Remotedesktop"
```
**Windows Home:** stattdessen RustDesk oder AnyDesk (unbeaufsichtigter
Zugriff + festes Passwort) oder Variante C.

**macOS:** Systemeinstellungen -> Allgemein -> Freigabe -> Bildschirmfreigabe
aktivieren, Benutzer freigeben. Client auf dem iPad: Jump Desktop oder
RealVNC Viewer.

**Linux:** `sudo apt install xrdp && sudo systemctl enable --now xrdp`
(bei GNOME 42+ alternativ die eingebaute RDP-Freigabe in den
Einstellungen -> Freigabe -> Remote-Desktop).

### 5.3 Am iPad

1. Tailscale aus dem App Store, mit demselben Konto anmelden.
2. Client installieren:
   - Windows-Ziel: **"Windows App"** von Microsoft (frueher "Microsoft Remote
     Desktop"), kostenlos.
   - macOS/Linux-VNC-Ziel: **Jump Desktop** (ca. 35 EUR, mit Abstand der beste
     iPad-Client, echte Trackpad- und Tastaturunterstuetzung) oder
     RealVNC Viewer (kostenlos).
3. Neue Verbindung anlegen mit dem Tailscale-Namen des Rechners
   (MagicDNS, z.B. `desktop-abc`) oder der 100.x.y.z-Adresse aus der
   Tailscale-App.

### 5.4 Headless nachruesten, wenn nur SSH da ist

Wenn auf dem Rechner bereits ein SSH-Server laeuft, brauchst du niemanden vor
Ort. Vom iPad aus mit **Blink Shell** oder **Termius** einloggen und die
Befehle aus 5.1 und 5.2 ausfuehren. Fuer Tailscale ohne Browser am Rechner:
im Tailscale-Admin-Panel einen Auth-Key erzeugen und

```
tailscale up --authkey tskey-auth-XXXX
```

benutzen. Damit ist der komplette Weg vom Terminal aus machbar.

## 6. Variante C - Sunshine + Moonlight (fuer EVE selbst)

Fuer echtes Spielen ist ein Desktop-Protokoll zu langsam. Game-Streaming
loest das:

- Am Rechner: **Sunshine** installieren (Open Source, Nachfolger von
  NVIDIA GameStream, laeuft mit AMD/Intel/NVIDIA). Web-UI auf
  `https://localhost:47990`, dort Benutzer und PIN setzen.
- Am iPad: **Moonlight** (kostenlos im App Store), Rechner ueber die
  Tailscale-Adresse hinzufuegen, PIN eingeben.
- Einstellungen fuer iPad: 1920x1200 (bzw. Displayaufloesung), 60 fps,
  20-30 Mbit/s. Ueber ein gutes WLAN ist das spielbar; ueber Mobilfunk
  spuerbar, aber fuer Marktfenster und Scanner voellig ausreichend.

Sunshine und RDP schliessen sich nicht aus - beides parallel installieren und
je nach Zweck waehlen. Achtung: eine aktive RDP-Sitzung sperrt die lokale
Konsole und stoert Sunshine; entweder/oder pro Sitzung.

## 7. Betrieb - die Details, an denen es real scheitert

- **Standby.** Der Rechner darf nicht schlafen, sonst ist er ab dem ersten Mal
  unerreichbar.
  Windows: `powercfg /change standby-timeout-ac 0` und
  `powercfg /change hibernate-timeout-ac 0`, Bildschirm darf ausgehen.
  macOS: Systemeinstellungen -> Batterie/Energie -> "Automatischen
  Ruhezustand ... verhindern".
  Linux: `sudo systemctl mask sleep.target suspend.target`.
- **Automatische Anmeldung / Sperrbildschirm.** Nach einem Neustart durch
  Windows Update steht der Rechner im Anmeldebildschirm. RDP kommt damit klar,
  VNC und Sunshine oft nicht. Entweder Auto-Login aktivieren (Sicherheits-
  abwaegung) oder RDP als Fallback bereithalten.
- **Aufwecken, wenn er doch aus ist.** Wake-on-LAN braucht ein zweites Geraet
  im selben LAN (Router mit Tailscale, Raspberry Pi, Fritzbox-WoL ueber
  MyFRITZ). Alternativ im BIOS "Restore on AC power loss" aktivieren und eine
  smarte Steckdose davorschalten - das ist der einzige Weg, einen komplett
  toten Rechner aus der Ferne hochzubekommen.
- **Feste Erreichbarkeit.** Mit Tailscale brauchst du weder feste IP noch
  DynDNS; MagicDNS-Name bleibt stabil.

## 8. Wenn niemand an den Rechner kann

Dann ist die ehrliche Antwort: es geht nicht. Es gibt keinen legitimen Weg,
auf einem Rechner ohne vorhandenen Dienst aus der Ferne Software zu starten.
Realistische Auswege:

1. Jemanden vor Ort telefonisch durch Abschnitt 4 lotsen (5 Minuten,
   erfordert keine Vorkenntnisse).
2. Bis zur Rueckkehr mit diesem Tool im Browser des iPads arbeiten - es ist
   reines HTML/JS und laeuft ohne den Rechner (Abschnitt 9).
3. Fuer die Zukunft: Tailscale + SSH gehoeren auf jeden Rechner, bevor man
   verreist. Diese Datei ist die Anleitung dafuer.

## 9. Zwischenloesung fuer dieses Repo

`index.html` und `reprocess.html` sind statische Seiten. Sie lassen sich ohne
den Heimrechner nutzen:

- ueber GitHub Pages aus diesem Repository veroeffentlichen, oder
- direkt aus der GitHub-Weboberflaeche als Datei ins iPad laden und in
  Safari oeffnen.

Zu beachten: das EVE-SSO-Login braucht eine Callback-URL, die in der
CCP-Developer-App hinterlegt ist. Fuer eine Pages-URL muss die dort
zusaetzlich eingetragen werden, sonst schlaegt der Login fehl.

## 10. Sicherheit

- **Niemals** Port 3389 (RDP) oder 5900 (VNC) im Router ins Internet
  weiterleiten. Das ist der Standardweg, ueber den Ransomware ins Heimnetz
  kommt. Tailscale oder der Relay des Anbieters loest das ohne offene Ports.
- Google-Konto (Variante A) mit 2FA absichern; die CRD-PIN ist sonst der
  einzige Schutz.
- Tailscale-ACLs so setzen, dass nur deine eigenen Geraete auf den Rechner
  duerfen; Key-Expiry fuer den Rechner deaktivieren, sonst faellt er nach
  180 Tagen aus dem Netz.
- TeamViewer/AnyDesk: unbeaufsichtigten Zugriff nur mit langem, eigenem
  Passwort, nicht mit dem Sitzungspasswort.

## 11. Dein Fall konkret: Windows Pro, nichts installiert, niemand vor Ort

Damit ist der direkte Weg zu. Bevor du aufgibst, drei Tueren, die schon offen
sein koennten, ohne dass jemals etwas eingerichtet wurde - alle vom iPad aus
in wenigen Minuten pruefbar, in dieser Reihenfolge:

### 11.1 Steam Link (beste Chance)

Wenn auf dem Rechner Steam laeuft (bei Windows-Spielern meist im Autostart),
ist Steam Remote Play **standardmaessig aktiv** und funktioniert auch ueber
das Internet, per Steam-Relay, ohne Portfreigabe.

1. App "Steam Link" auf dem iPad installieren (kostenlos).
2. Mit demselben Steam-Konto anmelden.
3. Findet die App den Rechner, bist du drin: Big Picture Mode, und darueber
   laesst sich jedes Spiel starten - EVE Online inklusive, wenn es ueber
   Steam installiert ist.

Das ist kein vollwertiger Desktop, aber es ist Bild, Ton, Maus und Tastatur.
Und: von einer laufenden Steam-Sitzung aus kommt man an einen Browser bzw.
ueber ein als "Nicht-Steam-Spiel" hinterlegtes Programm auch weiter - falls
so ein Eintrag schon existiert, ist das die Bruecke, um Abschnitt 5 nachtraeglich
zu installieren.

### 11.2 VPN des Routers

Viele Router bringen VPN mit, das ohne Zutun des PCs funktioniert:

- **FRITZ!Box**: MyFRITZ-Konto -> WireGuard/VPN-Zugang; Konfiguration geht
  komplett ueber `myfritz.net` im Safari, der PC ist daran unbeteiligt.
- **UniFi**: Teleport/WireGuard ueber die UniFi-App.
- **Speedport/Vodafone Station**: meist kein brauchbares VPN.

Bist du damit im LAN, teste mit der App "Windows App" (Microsoft) eine
RDP-Verbindung auf die lokale IP des PCs. Das klappt nur, wenn Remotedesktop
dort irgendwann einmal eingeschaltet wurde - probieren kostet zwei Minuten.

### 11.3 Ein anderes Geraet im selben Netz

Ein Synology/QNAP-NAS mit QuickConnect, ein Raspberry Pi mit offenem SSH, ein
Home-Assistant-Server: alles davon ist ein Brueckenkopf im LAN. Von dort aus
Wake-on-LAN an den PC schicken - und wenn auf dem PC RDP an ist, weiter per
RDP. Ohne aktiviertes RDP hilft aber auch das nicht.

### 11.4 Wenn alle drei zu sind

Dann bis zur Rueckkehr mit Abschnitt 9 arbeiten (dieses Tool laeuft im
iPad-Browser) und den PC bei naechster Gelegenheit einmal richtig aufsetzen.

### 11.5 Das 5-Minuten-Skript fuer den Moment, in dem du wieder am PC bist

PowerShell **als Administrator** oeffnen, einmal einfuegen, fertig:

```powershell
# 1) Tailscale (Mesh-VPN, keine Portfreigabe noetig)
winget install --id tailscale.tailscale -e --accept-source-agreements

# 2) Remotedesktop aktivieren (Windows Pro)
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name fDenyTSConnections -Value 0
Enable-NetFirewallRule -DisplayGroup "Remotedesktop"

# 3) OpenSSH-Server als zweites, unabhaengiges Standbein
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd

# 4) Standby aus, sonst ist der Rechner aus der Ferne tot
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0

# 5) Anmelden - oeffnet den Browser
tailscale up
```

Danach auf dem iPad: Tailscale-App + "Windows App", beide mit demselben Konto
bzw. dem Tailscale-Namen des Rechners. Ab da bist du von ueberall drin.

Zusaetzlich im Tailscale-Adminpanel unter *Machines* fuer den PC
**"Disable key expiry"** setzen - sonst faellt er nach 180 Tagen genau dann
aus dem Netz, wenn du wieder unterwegs bist.
