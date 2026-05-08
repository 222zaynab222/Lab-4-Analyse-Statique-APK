# Rapport d'analyse statique — UnCrackable Level 1

## Informations générales

|  | Valeur |
|-------|--------|
| **Date d'analyse** | 08/05/2026 |
| **Analyste** | Zaynab Ahbibi |
| **APK analysé** | UnCrackable-Level1.apk |
| **Version** | 1.0 (versionCode: 1) |
| **Provenance** | [OWASP MAS Crackmes](https://mas.owasp.org/crackmes/Android/) |
| **Hash SHA-256** | `1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21` |
| **Outils utilisés** | JADX GUI, dex2jar v2.4, JD-GUI |

## Résumé exécutif

Cette analyse statique a révélé **4 vulnérabilités/observations** dans l'application UnCrackable Level 1.  
Les principales préoccupations concernent la sauvegarde non protégée des données, une détection de root contournable par analyse statique, et l'utilisation d'AES avec une clé potentiellement récupérable dans le code.  
Le niveau de risque global est évalué comme  **Élevé**.

**Actions prioritaires recommandées :**
1. Désactiver `android:allowBackup` pour protéger les données utilisateur
2. Renforcer la protection de la clé de chiffrement AES
3. Mettre à jour `targetSdkVersion` vers une version récente

## Constats détaillés

### Constat #1 : Sauvegarde ADB activée

| | |
|--|--|
| **Sévérité** | Moyenne |
| **Localisation** | `AndroidManifest.xml` — attribut de `<application>` |

**Description :**  
L'attribut `android:allowBackup="true"` est présent dans le manifest. Cela permet à n'importe qui ayant accès physique à l'appareil d'extraire les données de l'application via ADB (`adb backup`).

**Impact potentiel :**  
Extraction des données privées de l'application sans root.

**Remédiation recommandée :**  
Définir `android:allowBackup="false"` dans le manifest.

---

### Constat #2 : Détection de root contournable statiquement

| | |
|--|--|
| **Sévérité** |  Élevée |
| **Localisation** | `sg.vantagepoint.uncrackable1.MainActivity.onCreate()`, `sg.vantagepoint.a.c` |

**Description :**  
L'application effectue 3 vérifications de root via `c.a()`, `c.b()`, `c.c()` dans `onCreate()`. L'une d'elles (`c.b()`) vérifie la présence de `"test-keys"` dans les tags du build. Ces vérifications sont entièrement visibles par décompilation et peuvent être contournées par patching ou Frida.

**Impact potentiel :**  
Un attaquant peut contourner la protection et exécuter l'app sur un appareil rooté pour analyser son comportement à l'exécution.

**Remédiation recommandée :**  
Utiliser une solution d'attestation d'intégrité côté serveur (Google Play Integrity API) plutôt que des vérifications locales.

---

### Constat #3 : Clé de chiffrement AES potentiellement récupérable

| | |
|--|--|
| **Sévérité** |  Élevée |
| **Localisation** | `sg.vantagepoint.a.a.a(byte[], byte[])`, `sg.vantagepoint.uncrackable1.a` |

**Description :**  
L'application utilise AES (`javax.crypto.spec.SecretKeySpec`) pour chiffrer/vérifier le secret. La clé et les données chiffrées sont présentes dans le code. Par décompilation, il est possible de retrouver la clé et de déchiffrer le secret sans exécuter l'application.

**Impact potentiel :**  
Récupération du secret en clair sans jamais lancer l'application.

**Remédiation recommandée :**  
Ne jamais stocker de clés cryptographiques dans le code. Utiliser Android Keystore pour la gestion des clés.

---

### Constat #4 : Version SDK cible obsolète

| | |
|--|--|
| **Sévérité** |  Faible |
| **Localisation** | `AndroidManifest.xml` — `<uses-sdk>` |

**Description :**  
`targetSdkVersion="28"` (Android 9) est obsolète. Les applications modernes doivent cibler au minimum Android 13 (API 33).

**Impact potentiel :**  
L'application ne bénéficie pas des protections de sécurité introduites dans les versions récentes d'Android.

**Remédiation recommandée :**  
Mettre à jour `targetSdkVersion` à 33 ou supérieur.

---
 ## Annexes

### Permissions demandées
> Aucune permission `<uses-permission>` déclarée.

### Composants exportés
- `sg.vantagepoint.uncrackable1.MainActivity` — exportée implicitement via `<intent-filter>` (MAIN/LAUNCHER)
- Aucun service, receiver ou provider déclaré.

### Structure de l'APK

| Fichier | Rôle |
|---------|------|
| `AndroidManifest.xml` | Configuration et permissions |
| `classes.dex` | Bytecode compilé |
| `META-INF/CERT.RSA` | Certificat de signature |
| `res/layout/activity_main.xml` | Interface utilisateur |
| `resources.arsc` | Ressources compilées |

### Comparaison des outils de décompilation

| Aspect | JADX GUI | JD-GUI |
|--------|----------|--------|
| **Ressources Android** | Accès complet (manifest, XML, assets) | Aucun accès |
| **Références ressources** | Résout en noms (`R.layout.activity_main`) | Affiche les entiers bruts (`2130903040`) |
| **Noms de variables** | Noms reconstruits et lisibles | Noms génériques (`paramBundle`, `param1Int`) |
| **Usage recommandé** | Analyse Android complète | Lecture rapide de code Java |

### Images demonstratif
<img width="600" height="464" alt="apk" src="https://github.com/user-attachments/assets/62376eb6-9083-42f2-9ac8-2dc96e584414" />
<img width="591" height="371" alt="debug" src="https://github.com/user-attachments/assets/186d0a8a-9da3-46cf-940a-e951993b30f6" />
<img width="600" height="464" alt="hash lvl1" src="https://github.com/user-attachments/assets/6cac8629-0074-490c-9a1f-431fbfdd9b13" />
<img width="867" height="310" alt="jar task5" src="https://github.com/user-attachments/assets/4b5eacd7-e763-425f-981f-359d2bba6257" />
<img width="591" height="371" alt="key" src="https://github.com/user-attachments/assets/a32348fd-59c4-4424-9318-d89f6862f88d" />
<img width="600" height="464" alt="lv1" src="https://github.com/user-attachments/assets/04b3321f-6340-4a5e-87e8-3a51669a6492" />
<img width="591" height="371" alt="password" src="https://github.com/user-attachments/assets/fd419759-67a2-4158-bf5c-82f64431a9ff" />
<img width="575" height="499" alt="task6" src="https://github.com/user-attachments/assets/25f740d1-002e-4911-9e1d-60f075c41c9c" />
<img width="591" height="371" alt="htttp" src="https://github.com/user-attachments/assets/02e312f7-c525-4fc4-960b-765b96198e5d" />
<img width="591" height="371" alt="token" src="https://github.com/user-attachments/assets/acf07c08-0f09-4851-b04f-277661596db4" />

