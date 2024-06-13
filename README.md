# EK Eigenes Projekt: Webserver / Database im Eigenheim

### Projektbeschreibung: Xubuntu Server als Datenbank- und Webserver im Heimnetzwerk

### Einleitung

Für ein Schulprojekt habe ich einen Xubuntu Server eingerichtet, der sowohl als Datenbank- als auch als Webserver dient. Der Server ist in meinem Heimnetzwerk integriert und bietet von außen über eine Domain Zugriff auf die gehosteten Anwendungen. Um die Sicherheit zu gewährleisten, wurde ein SSL-Zertifikat eingerichtet. Im Folgenden dokumentiere ich die einzelnen Schritte zur Installation, Konfiguration und Absicherung des Servers.

### 1. Installation und Einrichtung des Xubuntu Servers

1. **Installation des Betriebssystems:**
    - Das Xubuntu Betriebssystem wurde von der offiziellen Webseite heruntergeladen und auf einen USB-Stick übertragen.
    - Der Zielrechner wurde von diesem USB-Stick gebootet und die Installation von Xubuntu wurde durchgeführt.
2. **Grundlegende Konfiguration:**
    - Nach der Installation wurden die aktuellen Updates eingespielt:
        
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
        
    - Ein statisches Netzwerk-Setup wurde konfiguriert, um sicherzustellen, dass der Server immer die gleiche IP-Adresse im Heimnetzwerk hat.

### 2. Installation und Konfiguration des Webservers (Apache)

1. **Apache Webserver Installation:**
    - Der Apache Webserver wurde mit folgendem Befehl installiert:
        
        ```bash
        sudo apt install apache2 -y
        ```
        
2. **Überprüfung der Installation:**
    - Der Webserver wurde gestartet und überprüft, ob er läuft:
        
        ```bash
        sudo systemctl start apache2
        sudo systemctl enable apache2
        ```
        
    - Die Standard-Apache-Seite wurde durch den Zugriff auf `http://tgm-edu.d` überprüft.

### 3. Installation und Konfiguration der Datenbank (MySQL/MariaDB)

1. **Datenbank Installation:**
    - MariaDB wurde installiert, da es eine beliebte Alternative zu MySQL ist:
        
        ```bash
        sudo apt install mariadb-server mariadb-client -y
        ```
        
2. **Sicherheitskonfiguration:**
    - Das Sicherheitssetup-Skript wurde ausgeführt:
        
        ```bash
        sudo mysql_secure_installation
        ```
        
3. **Erstellung einer Datenbank und eines Benutzers:**
    - Eine neue Datenbank und ein Benutzer wurden erstellt:
        
        ```bash
        sudo mysql -u root -p
        CREATE DATABASE schule;
        CREATE USER 'schuluser'@'localhost' IDENTIFIED BY 'password';
        GRANT ALL PRIVILEGES ON schule.* TO 'schuluser'@'localhost';
        FLUSH PRIVILEGES;
        EXIT;
        ```
        

### 5. Download und Installation von Adminer

1. **Erstellen des Adminer-Verzeichnisses:**
    - Im Webserver-Verzeichnis wird ein neues Verzeichnis für Adminer erstellt:
        
        ```bash
        sudo mkdir -p /var/www/html/adminer
        ```
        
2. **Download von Adminer:**
    - Die neueste Version von Adminer wird heruntergeladen und in das erstellte Verzeichnis verschoben:
        
        ```bash
        sudo wget -O /var/www/html/adminer/index.php https://www.adminer.org/latest.php
        ```
        

### 2. Konfiguration des Apache Webservers für Adminer

1. **Erstellen einer Apache-Konfigurationsdatei für Adminer:**
    - Eine neue Konfigurationsdatei für Adminer wird erstellt, um sicherzustellen, dass Adminer ordnungsgemäß geladen wird:
        
        ```bash
        sudo nano /etc/apache2/sites-available/adminer.conf
        ```
        
2. **Konfigurationsinhalt für Adminer:**
    - Der folgende Inhalt wird in die Datei `adminer.conf` eingefügt:
        
        ```
        <VirtualHost *:80>
            ServerAdmin webmaster@localhost
            DocumentRoot /var/www/html/adminer
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
        
3. **Aktivieren der neuen Konfiguration:**
    - Die neue Site-Konfiguration wird aktiviert und der Apache-Webserver wird neu gestartet:
        
        ```bash
        sudo a2ensite adminer.conf
        sudo systemctl reload apache2
        ```
        

### 3. Absicherung von Adminer

1. **Verzeichnisschutz mit .htaccess:**
    - Eine `.htaccess` Datei wird erstellt, um den Zugang zu Adminer zu schützen:
        
        ```bash
        sudo nano /var/www/html/adminer/.htaccess
        ```
        
    - Der folgende Inhalt wird eingefügt, um grundlegende Authentifizierung zu verwenden:
        
        ```
        AuthType Basic
        AuthName "Restricted Access"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
        ```
        
2. **Erstellen eines Benutzeraccounts für die Authentifizierung:**
    - Ein Benutzer für die Authentifizierung wird erstellt:
        
        ```bash
        sudo htpasswd -c /etc/apache2/.htpasswd adminuser
        ```
        
    - Das System wird nach einem Passwort für `adminuser` fragen.
3. **Apache-Konfiguration für .htaccess anpassen:**
    - Die `adminer.conf` wird bearbeitet, um die Verwendung von `.htaccess` zu erlauben:
        
        ```
        <Directory /var/www/html/adminer>
            AllowOverride All
        </Directory>
        ```
        
4. **Neustart des Apache-Webservers:**
    - Der Apache-Webserver wird neu gestartet, um die Änderungen zu übernehmen:
        
        ```bash
        sudo systemctl restart apache2
        ```
        

### 4. SSL-Zertifikat für Adminer

1. **SSL-Zertifikat anwenden:**
    - Falls das SSL-Zertifikat bereits für den gesamten Server eingerichtet ist, ist keine weitere Konfiguration notwendig, da Adminer bereits über den Apache Webserver läuft und somit automatisch durch das bestehende Zertifikat geschützt wird.
2. **Überprüfung des Zugangs:**
    - Der Zugriff auf Adminer erfolgt nun über `https://<domain>/adminer`, wobei der Browser eine SSL-gesicherte Verbindung anzeigt.

### 6. Portweiterleitung im Heimnetzwerk

1. **Konfiguration des Routers:**
    - Zuerst wurde dem Server eine Reservierte IP addresse gegeben.
    - Auf dem Router wurde die Portweiterleitung für HTTP (Port 80) und HTTPS (Port 443) zur internen IP-Adresse des Xubuntu Servers (192.168.0.15) eingerichtet.
        - Zusätzliche Ports, wie der für die Datenbank (Port 3306), wurden bei Bedarf ebenfalls weitergeleitet, falls externe Verbindungen erforderlich sind.
            
            ![Screenshot 2024-06-13 074215.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6a3fa48a-9c6f-4129-b976-9eabc55a048f/e09e8a9c-ffdd-443f-b85a-224757195a4f/Screenshot_2024-06-13_074215.png)
            

### 5. Domain und DNS-Konfiguration

1. **Domain Registrierung:**
    - Eine Domain wurde bei einem Domain-Registrar registriert.
2. **DNS-Konfiguration:**
    - Ein A-Record wurde erstellt, der auf die externe IP-Adresse des Heimnetzwerks verweist.
    - Ein dynamischer DNS-Dienst (z.B. No-IP oder DynDNS) wurde konfiguriert, um sicherzustellen, dass die Domain auch bei einer sich ändernden externen IP-Adresse erreichbar bleibt.
    - Die Domain die dafür genutzt wurde ist `tgm-edu.de` und ist auf die Öffentliche IP des Heimnetzes gerichtet

### 6. Einrichtung des SSL-Zertifikats (Let's Encrypt)

1. **Installation von Certbot:**
    - Certbot wurde zur Verwaltung der SSL-Zertifikate installiert:
        
        ```bash
        sudo apt install certbot python3-certbot-apache -y
        ```
        
2. **Erstellung und Installation des SSL-Zertifikats:**
    - Ein SSL-Zertifikat wurde für die Domain beantragt und automatisch konfiguriert:
        
        ```bash
        sudo certbot --apache
        ```
        
    - Dieser Befehl führt durch den Prozess der Zertifikatsanfrage und Konfiguration von Apache zur Nutzung des Zertifikats.
3. **Automatische Erneuerung:**
    - Ein Cron-Job wurde erstellt, um das Zertifikat automatisch zu erneuern:
        
        ```bash
        echo "0 3 * * * /usr/bin/certbot renew --quiet" | sudo tee -a /etc/crontab > /dev/null
        ```
        

### 7. Test und Abschluss

1. **Test der Webanwendung:**
    - Der Zugriff auf die Webanwendung wurde über die Domain getestet, um sicherzustellen, dass sowohl HTTP als auch HTTPS korrekt funktionieren.
    - Die SSL-Konfiguration wurde mit Tools wie SSL Labs überprüft, um sicherzustellen, dass keine Sicherheitslücken bestehen.
2. **Test der Datenbank:**
    - Der Zugriff auf die Datenbank wurde über die Domain mittels DataGrip getestet.
