2023er Weg:

# Installation WLAN

Diese Anleitung wurde auf einem Raspberry Pi 4B getestet mit Raspberrian Lite 64bit legacy als OS.

Schritt 1 bis 3 sind auf einem anderen Endgerät (z.B. Laptop) zu machen.
1. Folgende Link öffnen: https://www.easyroam.de/Auth/Wayf?entityID=https://www.easyroam.de/shibboleth&return=https://www.easyroam.de/Shibboleth.sso/Login

   Nach "Hochschule München" suchen und mit der eigenen Kennung anmelden.
3. Bei "Manuelle Optionen" als Dateityp "PKCS12" auswählen und einen selbstgewählten Namen für das Zertifikat vergeben. Dann auf "Zugang generieren" klicken.
   Die Datei wird automatisch runtergeladen.
4. Die runtergeladene Datei muss nun auf den Pi übertragen werden. Folgenden Command haben wir benutzt. (Eingabe in die CMD oder per Terminal)
   scp -r /Pfad der heruntergeladenen Datei/Die Datei hat die Endung .p12 NameDesPis@IP-AdresseDesPis:/Verzeichnis/Wo es gespeichert werden soll
   Zum Bespiel so:
   scp -r /Users/chau/Downloads/my_easyroam_cert.p12 hokopi-piper@192.168.217.157:/home/hokopi-piper

Ab hier geht es auf dem Pi weiter. Als Beispiel heißt die Datei (die von easyroam heruntergeladen wurde) hier "easyroam.p12", euro heißt natürlich anders und muss dementsprechend bei den Commands angepasst werden.
4. Am Anfang am besten ein Update des Pis durchführen:
   sudo apt-get update
   sudo apt-get upgrade
5. Navigation zum Verzeichnis, wo das Zertifikat gespeichert wurde. Anschließend folgenden Befehl ausführen:

   openssl pkcs12 -in easyroam.p12 -legacy -nokeys | openssl x509 > easyroam_client_cert.pem 
   
   openssl x509 -noout -subject -in easyroam_client_cert.pem -legacy | sed 's/.*CN = \(.*\), C.*/\1/' > CN 

   #Bitte beachten, da der wpa_supplicant in der Regel nur passwortgeschützte Private Keys akzeptieren, muss bei der Extrahierung ein Passwort gesetzt werden. Bei folgendem Komando   
   #erscheint zunächst Enter Import Password, also mit <Return> quittieren, dann erscheint Enter PEM pass phrase: Hier gibt man ein neues Password ein und merkt es sich!

   openssl pkcs12 -in easyroam.p12 -nodes -nocerts | openssl rsa -aes256 -out easyroam_client_key.pem

   openssl pkcs12 -in easyroam.p12 -legacy -cacerts -nokeys > easyroam_root_ca.pem

   openssl pkcs12 -info -in easyroam_23_10_2023_05_12_27.p12 -legacy -nodes

6. Dann einen neuen Ordner erstellen:
   sudo mkdir /etc/easyroam-certs
7. Dann Dateien verschieben:
   sudo mv easyroam_client_cert.pem CN easyroam_client_key.pem easyroam_root_ca.pem /etc/easyroam-certs/.
8. Anschließend muss die Identität geändert werden. Dafür muss die "CN" datei geöffnet werden mit:
   sudo nano CN
9. Die Datei durchsuchen und den Teil kopieren der in etwa so aussieht:
   8314582678853535859@easyroam-pca.hm.edu
   Die Zahlenfolge ist natürlich eine andere.
10. Anschließend muss die wpa_supplicant.conf geöffnet werden:
   sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
11. Es muss folgendes Netzwerk in der Datei ergänzt/angelegt werden:
    
    country=DE 

network={ 
   ssid="eduroam" 
   
   scan_ssid=1 
   
   key_mgmt=WPA-EAP 
   
   proto=WPA2 
   
   eap=TLS 
   
   pairwise=CCMP 
   
   group=CCMP 
   
   identity="8314582678853535859@easyroam-pca.hm.edu"  # <---- Hier einfach die CN Datei mit einem Editor vim oder vi einlesen 
   
   ca_cert="/etc/easyroam-certs/easyroam_root_ca.pem" 
   
   client_cert="/etc/easyroam-certs/easyroam_client_cert.pem" 
   
   private_key="/etc/easyroam-certs/easyroam_client_key.pem" 
   
   private_key_passwd="Hier kommt das private key Passwort rein was oben festgelegt wurde"
   
}

12. Anschließend den Pi neustarten:
    sudo reboot

Danach sollte der Pi eine WLAN Verbindung mit dem eduroam aufbauen können.
