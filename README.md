# PKI_FullGuide
Infrastructure à Clé Publique
# Guide de Configuration de PKI (Infrastructure à Clé Publique)

Ce guide décrit les étapes pour configurer une infrastructure à clé publique (PKI) en utilisant OpenSSL. Il couvre la création d'une Autorité de Certification (CA) racine, d'une CA intermédiaire, et d'un certificat d'entité finale. Il aborde également les listes de révocation de certificats (CRL), l'utilisation d'OCSP pour vérifier l'état des certificats, la mise en place d'un référentiel de certificats, la création d'une autorité de marquage de temps (TSA), et la gestion des clés.

## 1. Générer une Clé Privée pour la CA Racine et un Certificat Auto-signé

Pour créer une Autorité de Certification (CA) racine, suivez ces étapes :

### Générer la Clé Privée pour la CA Racine
``bash
openssl genrsa -out rootCA.key 4096

### Créer le Certificat Auto-signé pour la CA Racine
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/C=FR/ST=État/L=Ville/O=Organisation/OU=UnitéOrg/CN=RootCA"

## 2. Configurer une CA Intermédiaire
Une CA intermédiaire est configurée pour émettre des certificats, réduisant ainsi le risque pour la CA racine.


### Générer la Clé Privée pour la CA Intermédiaire
openssl genrsa -out intermediateCA.key 4096

### Créer une Demande de Signature de Certificat (CSR) pour la CA Intermédiaire
openssl req -new -key intermediateCA.key -out intermediateCA.csr -subj "/C=FR/ST=État/L=Ville/O=Organisation/OU=UnitéOrg/CN=IntermediateCA"

### Signer le Certificat de la CA Intermédiaire avec la CA Racine
openssl ca -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediateCA.csr -out intermediateCA.crt -cert rootCA.crt -keyfile rootCA.key


## 3. Créer un Certificat d'Entité Finale (par exemple, pour un Serveur)
Les certificats d'entité finale sont utilisés pour les appareils tels que les serveurs, les utilisateurs ou d'autres entités réseau.

### Générer la Clé Privée pour le Serveur
openssl genrsa -out server.key 2048

### Créer une CSR pour le Serveur
openssl req -new -key server.key -out server.csr -subj "/C=FR/ST=État/L=Ville/O=Organisation/OU=Département IT/CN=www.exemple.com"

### Signer le Certificat du Serveur avec la CA Intermédiaire
openssl ca -in server.csr -out server.crt -cert intermediateCA.crt -keyfile intermediateCA.key -days 365 -extensions server_cert


## 4. Générer une Liste de Révocation de Certificats (CRL)
Une CRL liste les certificats qui ont été révoqués avant leur date d'expiration.

### Générer la CRL
openssl ca -gencrl -keyfile intermediateCA.key -cert intermediateCA.crt -out intermediateCA.crl


## 5. Utiliser OCSP pour Vérifier l'État des Certificats
OCSP (Online Certificate Status Protocol) est utilisé pour vérifier l'état de révocation d'un certificat.

### Démarrer un Répondeur OCSP
Dans un scénario réel, cela serait hébergé sur un serveur.
openssl ocsp -index /etc/pki/CA/index.txt -port 2560 -rsigner intermediateCA.crt -rkey intermediateCA.key -CA intermediateCA.crt -text

### Vérifier l'État d'un Certificat
openssl ocsp -CAfile intermediateCA.crt -url http://localhost:2560 -issuer intermediateCA.crt -cert server.crt


## 6. Mettre en Place un Référentiel de Certificats
Un référentiel de certificats est l'endroit où les certificats de CA, les CRL et d'autres fichiers sont stockés.

### Créer les Répertoires Nécessaires
mkdir -p /etc/pki/CA/{certs,crl,newcerts,private}
Fichier d'Index et Numéro de Série


touch /etc/pki/CA/index.txt
echo 1000 > /etc/pki/CA/serial


## 7. Autorité de Marquage de Temps (TSA)
Une TSA fournit une marque de temps fiable pour prouver l'existence d'un fichier ou d'un message à un moment précis.

### Générer le Certificat TSA
openssl req -new -key intermediateCA.key -out tsa.csr -subj "/C=FR/ST=État/L=Ville/O=Organisation/OU=Horodatage/CN=TSA"


### Signer le Certificat TSA avec la CA Intermédiaire
openssl ca -extensions v3_tsa -days 3650 -notext -md sha256 -in tsa.csr -out tsa.crt -cert intermediateCA.crt -keyfile intermediateCA.key

### Créer une Requête de Marque de Temps
openssl ts -query -data server.crt -out server.tsr

### Créer une Réponse de Marque de Temps
openssl ts -reply -queryfile server.tsr -out server.tsr -signer tsa.crt -keyfile tsa.key -config /etc/ssl/openssl.cnf


## 8. Gestion des Clés
La gestion des clés est essentielle pour sécuriser et organiser les clés cryptographiques.

### Générer une Nouvelle Clé Privée RSA
openssl genpkey -algorithm RSA -out server.key -aes256

### Extraire la Clé Publique d'une Clé Privée
openssl rsa -in server.key -pubout -out server_pub.key

### Convertir une Clé Privée dans un Format Différent (par exemple, PKCS#8)
openssl pkcs8 -topk8 -in server.key -out server_pkcs8.key -nocrypt

## 9. Audit et Surveillance (Commandes de Journalisation)
La surveillance et l'audit sont essentiels pour la sécurité et la conformité.

### Examiner le Journal d'Émission des Certificats

cat /etc/pki/CA/index.txt
