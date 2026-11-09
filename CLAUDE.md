# CLAUDE.md

Ce fichier guide Claude Code (claude.ai/code) lorsqu'il travaille sur le code de ce dépôt.

Scout : service sans interface qui qualifie les demandes d'appel découverte d'un consultant (Cal.com → Claude → email / Telegram).

- Besoin et règles métier : `docs/PRD.md`
- Architecture, schéma, événements, règles temporelles et phases : `docs/PLAN.md`

État : pas encore implémenté. Seul le projet Supabase existe (`supabase/`, aucune migration). L'application Next.js est mise en place en phase 1 du plan. Mettre ce fichier à jour avec les vraies commandes une fois l'application en place.

## Commandes

- `npx supabase start` / `npx supabase stop` : environnement local (Studio :54323, Mailpit :54324)
- `npx supabase migration new <nom>` : nouvelle migration dans `supabase/migrations`
- `npx supabase db reset` : reconstruit la base locale

## Invariants (détails dans PLAN.md)

- Tout email passe par la passerelle unique (consentement, désinscription, clé d'idempotence).
- Aucun envoi en double, ni après une annulation, un décalage, un remplacement ou une désinscription.
- RLS activée sur toutes les tables, sans policy ; accès à la base côté serveur uniquement, avec la clé service role.
- Doute ou échec d'analyse : jamais de refus automatique.
