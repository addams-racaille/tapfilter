# HomeCam 🎥

Système de surveillance maison via navigateur. Aucune application à installer.

## Architecture

- **Téléphone** → ouvre `/` → devient une caméra en direct
- **Viewer** → ouvre `/viewer` → voit la liste de toutes les caméras actives et se connecte

La vidéo passe en **WebRTC peer-to-peer** directement entre les appareils.  
Le serveur sert uniquement à coordonner la connexion (signalisation).

---

## Déploiement sur Railway (gratuit)

### 1. Créer un compte Railway
→ https://railway.app (connecte-toi avec GitHub)

### 2. Nouveau projet
- Clique **"New Project"**
- Choisis **"Deploy from GitHub repo"**
- Si pas de repo : clique **"Deploy from local"** et glisse le dossier

### 3. Ou via GitHub (recommandé)
1. Crée un repo GitHub (public ou privé)
2. Upload tous les fichiers du projet
3. Dans Railway → New Project → Deploy from GitHub → sélectionne ton repo
4. Railway détecte automatiquement Node.js et lance `npm start`

### 4. Obtenir l'URL
- Dans Railway, va dans **Settings → Networking → Generate Domain**
- Tu obtiens une URL comme `homecam-production.up.railway.app`

---

## Utilisation

| Qui | Ouvre |
|-----|-------|
| Vieux téléphone (caméra) | `https://ton-url.railway.app/` |
| Toi à distance (viewer)  | `https://ton-url.railway.app/viewer` |

### Étapes :
1. Sur le vieux téléphone → `/` → **"Démarrer la caméra"** → autorise la caméra
2. Le téléphone apparaît comme **"Caméra 1"** sur le viewer
3. Sur ton autre appareil → `/viewer` → clique **"VOIR →"** sur Caméra 1
4. Le flux démarre en direct
5. Depuis le viewer tu peux cliquer **🔄 Basculer** pour changer avant/arrière

### Plusieurs caméras :
Si tu connectes un 2e téléphone sur `/`, il apparaît comme **"Caméra 2"** automatiquement.

---

## Notes
- Fonctionne sur Chrome et Firefox mobile
- HTTPS obligatoire pour accéder à la caméra (Railway fournit HTTPS automatiquement)
- La caméra frontale est utilisée par défaut (l'arrière est cassée)
