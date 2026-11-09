# 🚀 scout – Points de départ des lives

Récupère le projet au début de chaque live et crée ta propre copie sur ton GitHub.

## ▶️ Session en cours : #1 – PRD & PLAN sont prêts

```bash
npx degit aymenjo/scout-starters#session-1 mon-projet
```

Ensuite :

1. **Installe** : `cd mon-projet && npm install`
2. **Mets-le sur ton GitHub** :

```bash
git init && git add . && git commit -m "Départ session 1"
gh repo create mon-projet --private --source=. --push
```

> Première fois ? Installe [GitHub CLI](https://cli.github.com) puis connecte-toi avec `gh auth login`.

## 📚 Toutes les sessions

| Session | Thème |
|---|---|
| #1 | PRD & PLAN sont prêts |

Pour une autre session, remplace `session-1` par son code dans la commande.
