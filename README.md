# PKI_FullGuide
Infrastructure à Clé Publique
# Guide de Configuration de PKI (Infrastructure à Clé Publique)

Ce guide décrit les étapes pour configurer une infrastructure à clé publique (PKI) en utilisant OpenSSL. Il couvre la création d'une Autorité de Certification (CA) racine, d'une CA intermédiaire, et d'un certificat d'entité finale. Il aborde également les listes de révocation de certificats (CRL), l'utilisation d'OCSP pour vérifier l'état des certificats, la mise en place d'un référentiel de certificats, la création d'une autorité de marquage de temps (TSA), et la gestion des clés.

## 1. Générer une Clé Privée pour la CA Racine et un Certificat Auto-signé

Pour créer une Autorité de Certification (CA) racine, suivez ces étapes :

### Générer la Clé Privée pour la CA Racine
```bash
openssl genrsa -out rootCA.key 4096
