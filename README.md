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
