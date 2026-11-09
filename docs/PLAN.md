# Plan : Scout

> PRD source : `docs/PRD.md`

## Décisions architecturales

Décisions durables qui s'appliquent à toutes les phases :

- **Runtime** : Next.js (App Router, TypeScript) déployé sur Vercel. Aucune interface.
- **Routes** :
  - `POST /api/webhooks/cal` — webhooks Cal.com, signature HMAC vérifiée, dédoublonnage par identifiant d'événement
  - `POST /api/webhooks/telegram` — clics sur les boutons Telegram, secret token vérifié
  - `/api/inngest` — point de service Inngest
  - `/api/desinscription?t=…` — route unique : le lien « ne plus être contacté » (GET) et le bouton natif du client mail (en-tête `List-Unsubscribe`, POST) désinscrivent au premier clic et renvoient « C'est noté ». Risque connu : un scanner de messagerie d'entreprise peut pré-cliquer le lien ; si des désinscriptions fantômes apparaissent, ajouter une confirmation en un bouton.
  - Liens « je décale / j'annule » = liens natifs Cal.com ; Scout est informé par webhook.
- **Authentification** : aucun compte utilisateur. Liens d'action = jetons signés HMAC (finalité + sujet + expiration). Base accessible uniquement côté serveur (service role) ; RLS activée sur toutes les tables, sans policy.
- **Schéma** (Postgres, migrations SQL versionnées dans `supabase/migrations`) — modèles clés :
  - `prospects` — email unique (insensible à la casse), identité, entreprise, site, rôle, date de désinscription
  - `inquiries` (demande) — prospect, source (`routing_form` | `booking` | `direct_link`), réponses (jsonb), date de consentement, statut (`screened_out` | `pending` | `qualified` | `not_qualified` | `in_review` | `replaced`), motif, raison, alternative, auteur de la décision (`rule` | `scout` | `consultant` | `default`), horodatages (reçue, décidée, verdict envoyé), indicateur de demande active
  - `bookings` — demande, uid Cal.com unique, début, fin, statut Cal.com (`pending` | `accepted` | `rejected` | `cancelled`), lien visio, présence (`absent` | `relevant` | `not_relevant`)
  - `consultant_prompts` — sollicitations Telegram (avis, présence, annonce de fiche, alerte) : type, id du message, échéance, résolution
  - `outbound_emails` — type, clé d'idempotence unique, id Resend, statut → garantie « 0 email en double »
  - `webhook_events` — source, id externe unique, payload brut
- **Événements Inngest** : `cal/booking.requested`, `cal/booking.cancelled`, `cal/booking.rescheduled`, `cal/form.submitted`, `scout/inquiry.decided`, `scout/review.answered`, `scout/prospect.opted_out`, `scout/attendance.recorded`. Cron : récap du lundi 8 h (Europe/Paris). Les envois planifiés sont annulables par annulation, décalage, remplacement ou désinscription.
- **Passerelle d'envoi unique** : tout email passe par elle. Elle refuse sans consentement ou après désinscription, impose une clé d'idempotence, ajoute le lien « ne plus être contacté » et les en-têtes List-Unsubscribe, et signe du nom du consultant (vouvoiement). Gabarit HTML simple commun. Transport : Resend en prod, SMTP Mailpit (Supabase local) en dev.
- **Frontières tiers** :
  - **Cal.com** — formulaire de routage (questions du PRD, consentement en choix obligatoire, message de réorientation pour les cas évidents) ; event type « Appel découverte » en confirmation requise avec blocage des créneaux non confirmés, emails aux participants coupés, champs de réservation cachés pré-remplis depuis le formulaire, préavis minimum 4 h ; event type caché « Rendez-vous direct » pour le repêchage ; API v2 pour confirmer, refuser, annuler.
  - **Claude** — SDK `@anthropic-ai/sdk`, modèle `claude-opus-5-5` (variable d'env), verdict en JSON structuré (`qualified` | `not_qualified` | `review` + raison + synthèse + alternative), fiche avec lecture du site public via l'outil serveur `web_fetch`, fallback de refus activé. Analyse impossible → doute.
  - **Telegram** — un bot, un chat (le consultant), boutons inline référant une `consultant_prompts`, message mis à jour après décision.
  - **Resend** — domaine du consultant vérifié ; à défaut pour une démo, expéditeur de test vers l'email de l'hôte.
- **Règles temporelles** (UTC en base, affichage Europe/Paris) : échéance d'un doute = min(réception + 24 h, début − 2 h), relance à +4 h si avant l'échéance ; rappel de la veille = début − 24 h, sauté si réservé moins de 24 h avant ; rappel du jour = début − 1 h ; fiche préparée à début − 3 h et envoyée à début − 1 h ; question de présence à la fin du créneau.
- **Configuration métier versionnée dans le repo** : correspondance options du formulaire → motifs d'écart, ressources gratuites, identité et signature du consultant. Secrets en variables d'env (Vercel / `.env.local`).
- **Tests et CI** : Vitest sur les règles pures (motifs, horaires, jetons, clés d'idempotence) ; tests de bout en bout en local en rejouant des payloads Cal.com et Telegram enregistrés. GitHub Actions sur PR : lint, typecheck, tests unitaires. Au merge sur `main` : `supabase db push` en prod, déploiement Vercel prod, synchronisation Inngest via l'intégration Vercel.
- **Environnements** : local (Supabase Docker, serveur de dev Inngest, Mailpit) et prod. Pas de staging.

---

## Phase 1 : Le tri automatique

**User stories** : US-1, US-3, US-4, US-8, US-9

### Ce qu'on livre

Point de départ : le projet de démarrage Scout (PRD, PLAN, dossier `supabase`). Première étape, la mise en place des comptes et services tiers :

- Projet Supabase prod ; projet Vercel relié au repo GitHub ; app Inngest reliée à Vercel
- Resend : domaine ajouté, vérification DNS lancée en premier (expéditeur de test en attendant)
- Clé API Claude
- Bot Telegram créé, chat_id du consultant récupéré
- Cal.com : formulaire de routage, event type « Appel découverte » (confirmation requise, emails participants coupés), event type caché « Rendez-vous direct », webhook vers `/api/webhooks/cal` (BOOKING_REQUESTED, BOOKING_CANCELLED, BOOKING_RESCHEDULED, FORM_SUBMITTED) avec secret, clé API
- Le lien public actuel du consultant reste inchangé jusqu'à la mise en service (fin de phase 2)

Puis le parcours : le prospect remplit le formulaire de routage Cal.com puis réserve ; la réservation arrive en attente et bloque le créneau. Scout reçoit le webhook signé, enregistre prospect, demande et réservation, fait analyser les réponses par Claude, puis confirme le rendez-vous dans Cal.com et envoie l'email de confirmation (qualifié), ou le refuse — créneau libéré — et envoie un message bienveillant avec l'alternative adaptée : « revenez quand le projet sera mûr » pour un démarrage lointain, les ressources gratuites sinon (non qualifié). En attendant la phase 2, un doute ou une analyse impossible est confirmé par défaut. Tous les emails passent par la passerelle unique ; le lien « ne plus être contacté » désinscrit au premier clic. Le socle (Next.js, migrations, Inngest, CI, déploiement) est posé au passage.

### Critères d'acceptation

- [ ] Comptes et services tiers configurés, secrets en place en local et sur Vercel.
- [ ] En local (`supabase start`, serveur de dev Inngest, `next dev`), rejouer un payload Cal.com enregistré produit le bon email dans Mailpit.
- [ ] Un webhook à la signature invalide est rejeté ; un webhook reçu deux fois ne produit qu'un seul traitement et un seul email.
- [ ] Réservation qualifiée → confirmée dans Cal.com + email de confirmation au prospect.
- [ ] Réservation non qualifiée → refusée dans Cal.com (créneau de nouveau disponible) + email bienveillant avec l'alternative correspondant au motif.
- [ ] Doute ou échec d'analyse → confirmé par défaut, auteur `default` enregistré.
- [ ] Verdict, raison, auteur et horodatages (reçue, décidée, verdict envoyé) enregistrés pour chaque demande.
- [ ] Aucun email Cal.com n'atteint le prospect.
- [ ] Aucun email sans consentement enregistré ; 100 % des emails contiennent le lien « ne plus être contacté » et l'en-tête List-Unsubscribe.
- [ ] Après un clic sur le lien « ne plus être contacté » ou sur le bouton natif du client mail, plus aucun email n'est envoyé à ce prospect.
- [ ] CI verte sur la PR ; au merge, migrations appliquées et Scout déployé en prod. Démo : réservation réelle → email de verdict en moins de 5 minutes.

## Bloquée par

Aucune — démarrable immédiatement à partir du projet de démarrage.

---

## Phase 2 : Le consultant dans la boucle

**User stories** : US-5, US-6, US-7, US-10 (+ compléments de US-8)

### Ce qu'on livre

Quand Claude hésite ou que l'analyse échoue, le consultant reçoit sur Telegram une synthèse avec deux boutons « Accepter » / « Refuser », et le prospect un email d'attente qui annonce quand il aura sa réponse. Sans réponse : relance à 4 h, puis confirmation par défaut à l'échéance ; le message Telegram affiche la décision prise. Chaque rendez-vous confirmé reçoit un rappel la veille avec les liens « je décale » / « j'annule », puis 1 h avant avec le lien de l'appel. Annulation, décalage et désinscription annulent ou replanifient les envois sans jamais de doublon ; un décalage d'un rendez-vous déjà confirmé est reconfirmé sans nouvelle analyse. Une nouvelle réservation d'un même prospect remplace la précédente. Désinscription avec rendez-vous prévu : rendez-vous maintenu, consultant prévenu sur Telegram. Une fois la phase validée : mise en service, le lien public du consultant pointe vers le formulaire de routage.

### Critères d'acceptation

- [ ] Doute → notification Telegram avec synthèse et boutons ; « Accepter » confirme le rendez-vous et envoie la confirmation ; « Refuser » le refuse et envoie le message bienveillant ; le message Telegram est mis à jour ; un second clic est sans effet.
- [ ] Le prospect en doute reçoit un email d'attente indiquant l'heure limite de réponse.
- [ ] Sans réponse : relance à 4 h (si avant l'échéance), confirmation par défaut à l'échéance (vérifié en local avec des horaires rapprochés).
- [ ] Rappel à début − 24 h (absent si réservé moins de 24 h avant) et à début − 1 h avec le lien visio.
- [ ] Après une annulation, aucun rappel ; après un décalage, les rappels suivent le nouvel horaire, jamais en double.
- [ ] Une 2ᵉ réservation du même prospect annule la 1ʳᵉ dans Cal.com et tous ses envois planifiés ; une seule demande active.
- [ ] Désinscription : envois planifiés annulés, rendez-vous maintenu, consultant prévenu sur Telegram.
- [ ] Un appel au webhook Telegram sans le bon secret est rejeté.
- [ ] Mise en service effectuée.

## Bloquée par

- Phase 1

---

## Phase 3 : Avant, après, et le filet

**User stories** : US-2, US-11, US-12, US-13

### Ce qu'on livre

Les cas évidents, déjà réorientés à l'écran par Cal.com, sont reconnus par Scout à partir des réponses du formulaire, enregistrés comme écartés avec leur motif, et reçoivent un email reprenant l'alternative. Trois heures avant chaque rendez-vous confirmé, Scout prépare une fiche (réponses, synthèse du besoin, raison du verdict, 3 questions à poser, quelques lignes sur l'entreprise lues sur son site public, historique éventuel), envoyée par email 1 h avant et annoncée sur Telegram. À la fin du créneau, le consultant indique en un geste « Absent » / « Présent – pertinent » / « Présent – pas pertinent » ; « Absent » déclenche un seul email de reprogrammation, et un prospect déjà absent qui réserve à nouveau passe en doute. Chaque lundi, le consultant reçoit un récap des écartés de la semaine avec leur motif et un lien « Repêcher » (réservation directe, confirmée d'office), et les chiffres de la semaine.

### Critères d'acceptation

- [ ] Pour chaque motif (budget, particulier, indépendant seul, plus de 50 personnes, démarrage lointain ou inconnu), un cas évident est enregistré écarté avec le bon motif et reçoit un seul email avec l'alternative ; « Qui décide » n'écarte jamais ; une soumission non évidente ne crée pas d'écarté.
- [ ] Fiche reçue au moins 55 min avant chaque rendez-vous confirmé, lisible sur téléphone, avec toutes ses rubriques ; site absent ou injoignable → fiche sans la partie entreprise, sans retard ; annonce Telegram envoyée.
- [ ] Rendez-vous annulé ou décalé → fiche annulée ou replanifiée.
- [ ] Question de présence à la fin du créneau ; choix enregistré ; « Absent » → exactement un email de reprogrammation.
- [ ] Prospect déjà marqué absent qui réserve → passe en doute.
- [ ] Le lundi 8 h (Europe/Paris), le récap liste les écartés de la semaine (cas évidents et non qualifiés) avec motif et lien « Repêcher », sauf pour les désinscrits, plus les chiffres : demandes, qualifiés, écartés, avis demandés, présence et pertinence, délai de verdict.
- [ ] Une réservation via un lien « Repêcher » est confirmée sans analyse et suit le parcours normal (rappels, fiche).
- [ ] Le récap est déclenchable à la main en local pour la démo.

## Bloquée par

- Phase 2
