# PowerTech Performance Blog - Server-Konfiguration

## Problem: 404-Fehler bei Clean URLs

Fast alle Artikel zeigen 404-Fehler, wenn man die URLs ohne `.shtml` Erweiterung aufruft.

**Lösung:** Der Server muss `.shtml` Dateien automatisch laden, wenn die URL ohne Erweiterung aufgerufen wird.

---

## 1. Apache Server (mit mod_rewrite)

**Anforderungen:**
- Apache mit `mod_rewrite` aktiviert
- `.htaccess` Dateien erlaubt

**Datei:** `.htaccess` (bereits im Repo vorhanden)

**Überprüfung:**
```bash
# Prüfe ob mod_rewrite aktiv ist:
a2enmod rewrite
systemctl restart apache2
```

**Test:**
- Öffne: `https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit.shtml` (sollte funktionieren)
- Öffne: `https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit` (sollte jetzt auch funktionieren)

---

## 2. Nginx Server

**Anforderungen:**
- Nginx 1.16+
- Zugriff auf `/etc/nginx/sites-available/` oder `/etc/nginx/conf.d/`

**Datei:** `nginx.conf`

**Installation:**
```bash
# 1. nginx.conf in Nginx-Verzeichnis kopieren
sudo cp nginx.conf /etc/nginx/sites-available/blog.powertech-performance.com.conf

# 2. Aktivieren
sudo ln -s /etc/nginx/sites-available/blog.powertech-performance.com.conf /etc/nginx/sites-enabled/

# 3. Testen
sudo nginx -t

# 4. Neustarten
sudo systemctl restart nginx
```

---

## 3. Vercel Deployment

**Anforderungen:**
- Vercel Account
- Git-Repo ist mit Vercel verbunden

**Datei:** `vercel.json` (bereits im Repo)

**Installation:**
1. Push zur GitHub
2. Vercel erkennt `vercel.json` automatisch
3. Deployment sollte funktionieren

**Test:**
```bash
vercel --prod
```

---

## 4. Netlify Deployment

**Anforderungen:**
- Netlify Account
- Git-Repo ist mit Netlify verbunden

**Datei:** `netlify.toml` (bereits im Repo)

**Installation:**
1. Push zur GitHub
2. Netlify erkennt `netlify.toml` automatisch
3. Deployment sollte funktionieren

**Test:**
```bash
netlify deploy --prod
```

---

## Diagnose: Welcher Server wird verwendet?

### Test 1: Mit .shtml Erweiterung
```
https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit.shtml
```
- ✅ Funktioniert? → Server hat die Datei, aber URL-Rewrite funktioniert nicht
- ❌ 404? → Dateien liegen nicht auf dem Server

### Test 2: Ohne .shtml Erweiterung
```
https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit
```
- ✅ Funktioniert? → URL-Rewrite ist aktiv ✓
- ❌ 404? → URL-Rewrite nicht konfiguriert

### Test 3: Server-Typ Erkennung
```bash
# Mit curl testen und Header anschauen
curl -I https://blog.powertech-performance.com/

# Suche nach Server-Header:
# Server: Apache/2.4.x
# Server: nginx/1.x
# Server: Vercel
```

---

## Häufige Fehler

### Fehler 1: "404 Not Found" bei allen URLs ohne .shtml
**Ursache:** `.htaccess` nicht aktiv oder mod_rewrite nicht installiert

**Lösung:**
```bash
# Überprüfen
apache2ctl -M | grep rewrite

# Wenn nicht aktiv:
a2enmod rewrite
a2enmod include
systemctl restart apache2
```

### Fehler 2: Infinite Redirect Loop
**Ursache:** `.htaccess` Regeln konfligieren

**Lösung:** `.htaccess` vereinfachen
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ $1.shtml [L]
```

### Fehler 3: "500 Internal Server Error"
**Ursache:** `.htaccess` Syntax falsch

**Lösung:**
1. Backup machen: `cp .htaccess .htaccess.bak`
2. Syntax testen: `apache2ctl -S`
3. `.htaccess` vereinfachen

---

## Aktuelle Link-Struktur

Alle Links sind jetzt **absolute URLs**:
```html
<!-- VORHER (kaputt): -->
<a href="adblue-deaktivierung-ford-transit">Link</a>

<!-- NACHHER (funktioniert): -->
<a href="https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit">Link</a>
```

Damit funktionieren Links auch wenn URL-Rewrite nicht aktiv ist.

---

## Checkliste für Go-Live

- [ ] Server-Typ identifiziert (Apache/Nginx/Vercel/Netlify)
- [ ] Entsprechende Konfigurationsdatei installiert
- [ ] mod_rewrite aktiv (bei Apache)
- [ ] Test mit `.shtml` URL durchgeführt
- [ ] Test ohne `.shtml` URL durchgeführt
- [ ] Alle Artikel geladen ohne 404-Fehler
- [ ] Links funktionieren

---

## Support

Wenn 404-Fehler noch immer erscheinen:

1. **Stelle sicher, dass Dateien auf Server sind:**
   ```bash
   curl -I https://blog.powertech-performance.com/adblue-deaktivierung-ford-transit.shtml
   ```

2. **Überprüfe Server-Logs:**
   - Apache: `/var/log/apache2/error.log`
   - Nginx: `/var/log/nginx/error.log`

3. **Kontaktiere Hosting-Provider** und frage nach:
   - Ist `mod_rewrite` aktiviert?
   - Werden `.htaccess` Dateien beachtet?
   - Welche PHP/Server-Version wird verwendet?

EOF
cat SERVER_SETUP.md
