# PKI_FullGuide
Infrastructure à Clé Publique
# Guide de Configuration de PKI (Infrastructure à Clé Publique)

Ce guide décrit les étapes pour configurer une infrastructure à clé publique (PKI) en utilisant OpenSSL. Il couvre la création d'une Autorité de Certification (CA) racine, d'une CA intermédiaire, et d'un certificat d'entité finale. Il aborde également les listes de révocation de certificats (CRL), l'utilisation d'OCSP pour vérifier l'état des certificats, la mise en place d'un référentiel de certificats, la création d'une autorité de marquage de temps (TSA), et la gestion des clés.

## 1. Générer une Clé Privée pour la CA Racine et un Certificat Auto-signé

Pour créer une Autorité de Certification (CA) racine, suivez ces étapes :

### Générer la Clé Privée pour la CA Racine
```bash
openssl genrsa -out rootCA.key 4096

##Créer le Certificat Auto-signé pour la CA Racine
```bash
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/C=FR/ST=État/L=Ville/O=Organisation/OU=UnitéOrg/CN=RootCA"

## 2. Configurer une CA Intermédiaire
Une CA intermédiaire est configurée pour émettre des certificats, réduisant ainsi le risque pour la CA racine.

## Générer la Clé Privée pour la CA Intermédiaire
