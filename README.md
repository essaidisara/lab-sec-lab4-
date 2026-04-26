# Rapport d'Analyse de Sécurité : UnCrackable Level 1 (OWASP)

##  Informations Générales
- **Analyste :** essaidi sara
- **Date :** 26 Avril 2026
- **Cible :** `app-debug.apk`
- **Objectif :** Analyse statique, contournement des protections et extraction de secrets.

---

##  1. Phase d'Analyse Initiale & Intégrité
La première étape a consisté à vérifier l'intégrité du fichier APK et à explorer sa structure interne via PowerShell.

- **Vérification du Magic Number :** `50 4B 03 04` (Format PK/ZIP confirmé).
- **Hash SHA256 :** `1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21`

<img width="555" height="197" alt="image" src="https://github.com/user-attachments/assets/1a4af7d5-209f-4b31-be51-c6aac746b38e" />


<img width="574" height="244" alt="image" src="https://github.com/user-attachments/assets/85f9ebb3-2c6d-44a5-90a8-7fde7825f00c" />


## 🛡️ 2. Constats de Sécurité (Analyse Statique)

### A. Protections Anti-Analyse (Root & Debug)
L'analyse de la classe `MainActivity` via **JADX** a révélé des mécanismes bloquants au démarrage de l'application.

* **Détection Root :** Appels aux méthodes `c.a()`, `c.b()`, `c.c()`.
* **Détection Debug :** Appel à `b.a()`.
* **Action :** Si une condition est remplie, l'application affiche "Root detected!" ou "App is debuggable!" et s'arrête via `System.exit(0)`.
<img width="1275" height="471" alt="image" src="https://github.com/user-attachments/assets/69488e70-e01f-495c-9c3f-e6aa36d323d1" />

### B. Analyse Cryptographique & Extraction du Secret
C'est le point critique de l'application identifié dans la classe `sg.vantagepoint.uncrackable1.a`.

**Données extraites :**
- **Clé AES (Hex) :** `8d127684cbc37c17616d806cf50473cc`
- **Secret Chiffré (Base64) :** `5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=`


<img width="958" height="427" alt="image" src="https://github.com/user-attachments/assets/1b5034d4-cebb-4abd-b018-c27c06980ebb" />


---

##  3. Processus de Décompilation (Task 5 & 6)
Pour valider les découvertes de JADX, nous avons procédé à une décompilation manuelle :
1.  Extraction du `classes.dex`.
2.  Conversion en JAR avec `dex2jar`.

<img width="1411" height="556" alt="image" src="https://github.com/user-attachments/assets/6927b900-258a-476e-b61a-a16f1fc4fa65" />
<img width="1492" height="538" alt="image" src="https://github.com/user-attachments/assets/d75612c8-548f-4ff2-8822-b54c160ea05b" />

3.  Analyse croisée avec **JD-GUI** pour confirmer la structure du code.
<img width="981" height="433" alt="image" src="https://github.com/user-attachments/assets/b505427d-ab34-4fde-8990-154ce0fd216b" />

---

##  4. Synthèse des Vulnérabilités

| Constat | Sévérité | Impact |
| :--- | :--- | :--- |
| **Clé AES en dur** |  Critique | Permet le déchiffrement immédiat du secret par un tiers. |
| **Mode AES-ECB** |  Élevée | Manque de protection contre les analyses de motifs (patterns). |
| **Root Detection basique** |  Moyenne | Facilement contournable via des outils de "hooking" comme Frida. |

---

##  5. Remédiations recommandées
1.  **Android Keystore :** Stocker les clés de chiffrement dans un environnement matériel sécurisé (TEE).
2.  **Chiffrement Authentifié :** Migrer de `AES/ECB` vers `AES/GCM/NoPadding`.
3.  **Obfuscation :** Appliquer **ProGuard/R8** pour masquer la logique de validation et les noms de classes.

---

##  Annexes
- **Permissions :** Aucune permission sensible détectée dans le manifeste.
- **Composants :** `MainActivity` est le seul composant exporté.
