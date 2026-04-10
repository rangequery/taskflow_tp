# TaskFlow — Projet React Front-End

## Séance 2 — Auth Context & Protected Layout

### Réponses aux questions

**Q2 : Pourquoi le useAuth() lance une erreur si le context est null ? Quel bug ça prévient ?**

Si le context est null, cela signifie que le composant qui appelle `useAuth()` n'est pas enveloppé dans un `<AuthProvider>`. Lancer une erreur permet de détecter immédiatement ce problème au lieu d'avoir des bugs silencieux (comme des valeurs undefined) plus tard dans l'application.

**Q3 : Sans Context, comment feriez-vous pour partager le user entre Header, Sidebar et Login ? Combien de props ?**

Sans Context, il faudrait passer le user et le dispatch via les props depuis App vers chaque composant enfant. On aurait besoin de passer au minimum 2 props (user + setUser ou dispatch) à travers chaque niveau : App → Header, App → Sidebar, App → Login. C'est le problème du "prop drilling" — si un composant intermédiaire ne consomme pas ces props, il les transmet quand même.

**Q4 : Pourquoi e.preventDefault() est indispensable dans handleSubmit ?**

Par défaut, un formulaire HTML recharge la page quand on le soumet. `e.preventDefault()` empêche ce comportement par défaut du navigateur, ce qui permet de gérer la soumission en JavaScript (appel API asynchrone) sans recharger la page et perdre tout le state React.

**Q5 : Que fait la destructuration { password: _, ...user } ? Pourquoi exclure le password ?**

Cette syntaxe extrait la propriété `password` dans une variable `_` (qu'on ignore) et regroupe toutes les autres propriétés dans l'objet `user`. On exclut le password pour ne pas le stocker dans le state React — c'est une bonne pratique de sécurité, même si ici les mots de passe sont en clair dans json-server.

**Q6 : Pourquoi le Dashboard est un composant séparé et pas tout dans App ?**

Séparer Dashboard de App permet de respecter le principe de responsabilité unique. App gère la logique de routage (login vs dashboard), tandis que Dashboard gère l'affichage du tableau de bord. De plus, cela évite que les hooks comme `useEffect` et `useState` du Dashboard ne soient exécutés quand l'utilisateur n'est pas connecté.

**Q8 : onLogout est un CALLBACK. Dessinez le flux.**

Le flux est : `Header` → l'utilisateur clique sur le bouton "Déconnexion" → `onClick` appelle `onLogout` → `onLogout` appelle `dispatch({ type: 'LOGOUT' })` → le reducer met `user` à `null` → le Context notifie `App` → `App` re-render → `authState.user` est null → `<Login />` est affiché à la place du Dashboard.

**Q9 : Pourquoi le flash disparaît avec useLayoutEffect ?**

Avec `useEffect`, le cycle est : Render → Commit → Paint → Effect. L'utilisateur voit d'abord la position (0,0) puis le repositionnement. Avec `useLayoutEffect`, le cycle est : Render → Commit → Effect → Paint. Le navigateur attend que l'effect ait fini avant de peindre, donc l'utilisateur ne voit jamais la position initiale (0,0).

**Q10 : Pourquoi ne pas utiliser useLayoutEffect partout si c'est mieux ?**

`useLayoutEffect` bloque le rendu visuel — le navigateur ne peut pas peindre tant que l'effect n'est pas terminé. Si l'effect est lourd (appel API, calcul complexe), l'utilisateur verra un freeze de l'interface. `useEffect` est préféré par défaut car il ne bloque pas le paint, ce qui donne une meilleure expérience utilisateur dans la majorité des cas.

---

## Séance 3 — React Router, Axios & CRUD

### Réponses aux questions

**Q1 : Que se passe-t-il si on tape /dashboard sans être connecté ? Pourquoi ?**

On est redirigé automatiquement vers `/login`. C'est grâce au composant `ProtectedRoute` qui vérifie si `state.user` est null. Si l'utilisateur n'est pas connecté, il fait un `<Navigate to="/login">` avec le pathname actuel dans `state.from`, pour pouvoir y revenir après le login.

**Q2 : Différence entre `<Link>` et `<NavLink>` ?**

`<Link>` fait une navigation simple sans rechargement de page. `<NavLink>` fait la même chose mais ajoute automatiquement une classe CSS `active` (ou une classe personnalisée via `className`) quand le lien correspond à la route actuelle. C'est utile pour la sidebar pour mettre en surbrillance le projet sélectionné.

**Q3 : Pourquoi api.get() au lieu de fetch() ?**

Axios apporte plusieurs avantages par rapport à fetch : il transforme automatiquement le JSON (pas besoin de `.json()`), il gère mieux les erreurs (rejette les promesses pour les status >= 400), il supporte les intercepteurs pour ajouter des headers automatiquement (comme le token JWT), et il permet de configurer un `baseURL` et un `timeout` une seule fois.

**Q4 : Le code `axios.isAxiosError(err)` sert à quoi ? Que se passe-t-il si le serveur est éteint ?**

`axios.isAxiosError(err)` vérifie que l'erreur est bien une erreur Axios (et pas une autre erreur JS). Si le serveur est éteint, Axios lance une erreur réseau (`ERR_NETWORK`) — `err.response` est `undefined` dans ce cas car il n'y a pas de réponse HTTP du tout, mais l'erreur est quand même capturée dans le catch.

**Q5 : Pourquoi `setProjects(prev => [...prev, data])` et pas `setProjects([...projects, data])` ?**

La version avec callback (`prev => ...`) utilise toujours la valeur la plus récente du state. Si on utilise directement `projects`, on capture la valeur au moment du rendu, qui pourrait être périmée si plusieurs setState sont en attente. C'est une pratique recommandée quand le nouveau state dépend de l'ancien.

**Q6 : On fait le PUT avec `{ ...project, name: newName }`. Que se passe-t-il si on oublie `...project` ?**

Si on oublie le spread, on envoie seulement `{ name: newName }` au serveur. json-server va remplacer tout l'objet — on perd les autres propriétés comme `id` et `color`. Le projet se retrouverait sans couleur et potentiellement sans id dans la réponse.

**Q7 : Que fait `useParams()` ? Pourquoi c'est mieux que de passer l'id via props ?**

`useParams()` extrait les paramètres dynamiques de l'URL (ici `:id` de `/projects/:id`). C'est mieux que les props car : l'URL est la source de vérité (on peut partager le lien), le composant est découplé de son parent, et le Back/Forward du navigateur fonctionne naturellement.

---

## Séance 4 — MUI vs Bootstrap & Architecture BDD

### Réponses aux questions

**Q1 : Combien de lignes de CSS avez-vous écrit pour le Header MUI ? Comparez avec votre Header.module.css.**

Avec MUI : 0 ligne de CSS externe. Tout le style est en inline via la prop `sx={{}}` directement sur les composants MUI. Avec le Header CSS Modules, on avait environ 20 lignes de CSS dans `Header.module.css`. MUI élimine complètement le besoin de fichiers CSS séparés grâce à son système de style intégré.

**Q2 : Comparez le code du Header MUI vs Bootstrap. Lequel est plus lisible ? Plus court ?**

Bootstrap est légèrement plus court et plus lisible car il utilise des classes utilitaires simples (`className="ms-3 fw-bold"`). MUI est plus verbeux avec les objets `sx={{}}` mais offre plus de contrôle et de type-safety (TypeScript vérifie les propriétés). Bootstrap est plus familier pour les développeurs web classiques, MUI est plus "React-natif".

**Q3 : Le Login MUI utilise sx={{}} pour le style. Le Login Bootstrap utilise des classes CSS (className). Quel système préférez-vous ? Pourquoi ?**

Je préfère MUI avec `sx={{}}` car le style est co-localisé avec le composant — pas besoin de naviguer entre fichiers. De plus, on bénéficie de l'autocomplétion TypeScript pour les propriétés CSS. Cependant, Bootstrap avec les classes est plus rapide à prototyper et le code est plus compact.

**Q4 : Si vous deviez choisir UNE seule library pour TaskFlow en production, laquelle et pourquoi ?**

Je choisirais Material UI car : (1) le système de thème permet une personnalisation globale cohérente, (2) les composants sont plus complets (TextField avec labels flottants, Alert avec icônes), (3) le `sx` prop offre un contrôle fin sans fichiers CSS séparés, et (4) la documentation est très complète avec des exemples TypeScript.

### Tableau comparatif

| Critère | Material UI | React-Bootstrap |
|---------|------------|----------------|
| Installation | 4 packages (@mui/material, @emotion/react, @emotion/styled, @mui/icons-material) | 2 packages (react-bootstrap, bootstrap) |
| Nombre de composants utilisés | 7 (AppBar, Toolbar, Typography, IconButton, Button, Box, TextField) | 6 (Navbar, Container, Button, Nav, Card, Form) |
| Lignes de CSS écrites | 0 (tout en sx={{}}) | 0 (classes utilitaires Bootstrap) |
| Système de style | sx={{}} (CSS-in-JS) | Classes CSS utilitaires (className) |
| Personnalisation couleurs | Très flexible via sx et createTheme | Via variables CSS Bootstrap ou style inline |
| Responsive | Grid + breakpoints dans sx | Grid Bootstrap + classes responsive |
| Lisibilité du code | Verbeux mais explicite | Plus court et lisible |
| Documentation | Excellente, avec exemples TS | Bonne, proche de Bootstrap classique |
| Votre préférence | Pour les apps complexes | Pour le prototypage rapide |

### Architecture Base de Données

**Architecture actuelle de TaskFlow :**

```
React (Vite :5173) --HTTP (GET/POST/PUT/DELETE)--> json-server (:4000) --> db.json
```

**a) Avec Firebase :**
```
React (Vite :5173) --HTTPS (SDK Firebase)--> Firebase Cloud --> Firestore/Realtime DB
```

**b) Avec Express + MongoDB :**
```
React (Vite :5173) --HTTP--> Express API (:3000) --Mongoose--> MongoDB (:27017)
```

**Q5 : Pourquoi React ne peut-il PAS se connecter directement à MySQL ?**

React s'exécute dans le navigateur (côté client). MySQL utilise un protocole TCP binaire qui nécessite un driver natif — impossible à exécuter dans un navigateur pour des raisons de sécurité. De plus, exposer la base de données directement au client exposerait les credentials de connexion et permettrait n'importe quelle requête SQL (injection, suppression de tables, etc.).

**Q6 : json-server est parfait pour notre TP. Donnez 3 raisons pour lesquelles on ne l'utiliserait PAS en production.**

1. **Pas d'authentification** : json-server ne vérifie pas les tokens JWT, n'importe qui peut faire des requêtes CRUD.
2. **Pas de validation** : aucune validation des données envoyées — on peut envoyer n'importe quel JSON et il sera accepté.
3. **Pas scalable** : les données sont stockées dans un fichier JSON, pas de gestion de concurrence, pas d'index, pas de relations complexes.

**Q7 : Firebase permet à React de se connecter directement (pas de backend Express). Comment est-ce possible alors que MySQL ne le permet pas ?**

Firebase expose une API REST/WebSocket accessible via HTTPS depuis le navigateur. Les règles de sécurité (Firebase Security Rules) sont définies côté serveur Firebase pour contrôler qui peut lire/écrire quoi. MySQL utilise un protocole TCP binaire non-web. Firebase agit comme un "backend-as-a-service" — le backend existe, mais Google le gère pour nous.

**Q8 : Votre TaskFlow utilise json-server. Un client vous demande de passer en production. Quelles étapes ?**

1. Remplacer json-server par un vrai backend (Express/NestJS + PostgreSQL/MongoDB)
2. Implémenter une vraie authentification JWT avec hachage des mots de passe (bcrypt)
3. Ajouter la validation des données (Joi, Zod)
4. Configurer HTTPS et CORS
5. Déployer le frontend (Vercel/Netlify) et le backend (AWS/Railway/Render)
6. Ajouter des tests (unitaires + intégration)

**Q9 : MUI et Bootstrap sont des libraries externes. Quel est le risque d'en dépendre ?**

Le risque principal est la taille du bundle — MUI ajoute environ 80-100 KB gzippé et Bootstrap environ 25 KB. De plus, les breaking changes lors des mises à jour majeures (ex: MUI v4 → v5) nécessitent une migration coûteuse. Si la library est abandonnée, on se retrouve avec du code legacy difficile à maintenir.

**Q10 : App de chat en temps réel — json-server, Firebase ou Backend custom ?**

Firebase, car il offre nativement le temps réel via Firestore/Realtime Database — les messages sont synchronisés instantanément entre tous les clients via WebSocket sans code backend. json-server ne supporte pas le temps réel. Un backend custom (Express + Socket.io + MongoDB) serait possible mais demanderait beaucoup plus de développement.

---

## Séance 5 — Sécurité JWT, Redux Toolkit & Performance

### Réponses aux questions

**Q1 : Le script s'exécute-t-il ? Pourquoi ? Que fait React avec les strings dans le JSX ?**

Non, le script ne s'exécute pas. React échappe automatiquement toutes les strings insérées dans le JSX via `{}`. Le HTML malveillant `<img src=x onerror=alert("HACK")>` est affiché comme du texte brut, pas interprété comme du HTML. C'est la protection XSS native de React.

**Q2 : Que se passe-t-il avec dangerouslySetInnerHTML ?**

Avec `dangerouslySetInnerHTML`, React injecte le HTML tel quel dans le DOM sans l'échapper. L'image avec l'onerror s'exécute et l'alerte s'affiche — c'est une faille XSS. C'est pour ça que cette prop s'appelle "dangerously" : elle désactive la protection de React. Il ne faut JAMAIS l'utiliser avec des données provenant d'un utilisateur ou d'une API.

**Q3 : Ouvrez Network (F12). Voyez-vous le header Authorization: Bearer ... ?**

Oui, après login, chaque requête GET/POST/PUT/DELETE vers json-server contient le header `Authorization: Bearer <token>` grâce à l'intercepteur Axios configuré dans `setAuthToken()`. Le token est ajouté automatiquement via `api.defaults.headers.common['Authorization']`.

**Q4 : Pourquoi stocker le token en mémoire (state React) et PAS dans localStorage ?**

localStorage est accessible par TOUT script JavaScript de la page. En cas de faille XSS, un attaquant peut lire le token avec `localStorage.getItem('token')`. Le state React est isolé dans le composant — il n'est pas accessible depuis la console ou un script injecté. Le compromis est que le token est perdu au refresh, mais c'est plus sécurisé.

**Q5 : Comparez authSlice.ts avec votre ancien authReducer.ts. Qu'est-ce qui a changé ?**

1. **Plus de switch/case** : Redux Toolkit utilise des reducers nommés dans un objet `reducers: {}`.
2. **Plus de string constants** : les action types sont générés automatiquement (ex: `auth/loginStart`).
3. **Mutations "directes"** : on écrit `state.user = action.payload.user` au lieu de `return { ...state, user: ... }`. RTK utilise Immer en coulisse pour créer un nouvel objet immutable.
4. **Export simplifié** : les action creators sont exportés automatiquement via `authSlice.actions`.

**Q6 : Combien de composants se re-rendent quand on toggle la sidebar ? Lesquels ne DEVRAIENT PAS ?**

Sans optimisation, Header, Sidebar ET MainContent se re-rendent tous quand on toggle la sidebar. Seul Sidebar devrait se re-rendre (car `isOpen` change). MainContent ne devrait PAS car ses props (`columns`) n'ont pas changé.

**Q7 : Pourquoi MainContent ne se re-rend plus avec React.memo ? Que compare React.memo ?**

`React.memo` effectue une comparaison superficielle (shallow comparison) des props. Si les props n'ont pas changé (même référence), le composant ne se re-rend pas. Comme `columns` est le même tableau en mémoire (pas re-créé), la comparaison retourne `true` et le re-render est évité.

**Q8 : Quelle différence entre useMemo et useCallback ? Quand utiliser chacun ?**

- `useMemo(() => computeValue(), [deps])` mémorise une **valeur** calculée.
- `useCallback((args) => doSomething(), [deps])` mémorise une **fonction**.

`useCallback(fn, deps)` est équivalent à `useMemo(() => fn, deps)`. On utilise `useCallback` pour les fonctions passées en props à des composants mémoisés (`React.memo`), et `useMemo` pour des calculs coûteux qu'on ne veut pas refaire à chaque render.

**Q10 : Profiler — quels composants se re-rendent pour chaque action ?**

a) **Toggle sidebar** : avec React.memo, seul Dashboard et Sidebar se re-rendent. MainContent est évité.
b) **Ajouter un projet** : Dashboard, Sidebar et MainContent (car les données changent).
c) **Naviguer vers ProjectDetail** : le routeur unmount Dashboard et mount ProjectDetail.
d) **Se déconnecter** : tout est unmount, Login est monté.
