# Scout

Scout qualifie les demandes d'appel découverte avant qu'elles n'arrivent dans l'agenda du consultant.

C'est un service sans interface. Le prospect ne voit que le lien de prise de rendez-vous Cal.com, puis des emails. Le consultant ne reçoit que des notifications sur Telegram.

## Statut

En cadrage. Le PRD et le plan sont rédigés et le projet Supabase est initialisé (sans migration). L'application n'est pas encore implémentée : elle sera posée en phase 1.

## Comment ça marche

1. Le prospect remplit le formulaire de routage Cal.com : qui il est, budget, démarrage, qui décide, besoin. Les cas évidemment hors cible sont réorientés sans voir l'agenda.
2. Les autres réservent. Le rendez-vous reste en attente et bloque le créneau.
3. Cal.com prévient Scout par un webhook signé, et Claude analyse les réponses.
4. Trois issues possibles :
   - **qualifié** : le rendez-vous est confirmé et le prospect reçoit un email de confirmation ;
   - **non qualifié** : le créneau est libéré et le prospect reçoit un message bienveillant avec une alternative ;
   - **doute** : le consultant tranche d'un geste sur Telegram. Sans réponse, le rendez-vous est confirmé par défaut.
5. Ensuite : rappels au prospect, fiche sur le prospect envoyée au consultant 1 h avant l'appel, suivi de présence, et récap des prospects écartés chaque lundi.

## Stack

- Next.js (App Router, TypeScript) déployé sur Vercel
- Supabase (Postgres)
- Inngest (événements et envois planifiés)
- Claude via `@anthropic-ai/sdk`
- Cal.com (formulaire de routage, API v2, webhooks)
- Telegram (bot du consultant)
- Resend en production, Mailpit en local

## Démarrage local

Prérequis : Node.js et Docker (Docker est nécessaire à `supabase start`).

```bash
npm install            # installe la CLI Supabase (dépendance de dev)
npx supabase start     # démarre l'environnement local
npx supabase stop      # l'arrête
```

| Service  | URL locale                |
| -------- | ------------------------- |
| API      | http://localhost:54321    |
| Postgres | localhost:54322           |
| Studio   | http://localhost:54323    |
| Mailpit  | http://localhost:54324    |

Base de données :

```bash
npx supabase migration new <nom>   # nouvelle migration dans supabase/migrations
npx supabase db reset              # reconstruit la base locale
```

Les commandes de Next.js et d'Inngest seront ajoutées en phase 1.

## Structure

```
docs/PRD.md     besoin, user stories, règles métier
docs/PLAN.md    architecture, schéma, événements, règles temporelles, phases
supabase/       configuration Supabase et migrations
CLAUDE.md       consignes pour Claude Code
```

## Feuille de route

1. **Le tri automatique** : réception des réservations, analyse, confirmation ou refus, passerelle d'envoi.
2. **Le consultant dans la boucle** : doutes sur Telegram, rappels, annulations et décalages, mise en service.
3. **Avant, après, et le filet** : cas évidents, fiche avant l'appel, suivi de présence, récap du lundi.

Détail et critères d'acceptation : [docs/PLAN.md](docs/PLAN.md).

## Garde-fous

- Tout email passe par une passerelle unique, qui vérifie le consentement, gère la désinscription et impose une clé d'idempotence.
- Aucun envoi en double, ni après une annulation, un décalage, un remplacement ou une désinscription.
- RLS activée sur toutes les tables, sans policy. La base n'est accessible que côté serveur.
- En cas de doute ou d'échec d'analyse, jamais de refus automatique.

Pour le besoin complet, voir [docs/PRD.md](docs/PRD.md).
