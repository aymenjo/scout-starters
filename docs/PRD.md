# Scout — PRD

## Problème

Un consultant qui dirige seul une agence d'automatisation et d'IA reçoit 5 à 15 demandes d'appel découverte par semaine via son lien de prise de rendez-vous. Environ la moitié lui font perdre du temps : prospects sans budget, besoin flou, hors de sa cible, ou qui ne se présentent pas. Aucun tri n'existe : toute personne qui a le lien voit l'agenda et réserve. Or l'agenda est désormais saturé, et ces appels inutiles prennent les créneaux des clients et des vrais prospects.

## Solution

Scout trie les demandes à la place du consultant, sans interface : le prospect ne voit que le lien de prise de rendez-vous puis des emails, et le consultant ne reçoit que des notifications.

Avant de réserver, le prospect répond à quelques questions simples. Les cas évidents sont réorientés tout de suite, sans voir l'agenda. Les autres réservent ; Scout analyse alors leurs réponses et décide en quelques minutes. Qualifié : confirmation et rappels. Non qualifié : créneau libéré et message bienveillant avec une alternative adaptée. En cas de doute, le consultant tranche en un geste depuis son téléphone. Avant chaque appel, le consultant reçoit une courte fiche sur le prospect.

## Utilisateur cible

**Le consultant.** Consultant solo à la tête d'une agence d'automatisation et d'IA, avec un seul agenda. Vend des projets à partir de 3 000 € HT (automatisation de process, agents et assistants IA, formation IA, développement sur mesure) à des TPE/PME de 5 à 50 personnes dont le dirigeant décide vite. Consulte son téléphone entre deux rendez-vous.

**Le prospect.** Dirigeant ou salarié d'entreprise qui veut parler d'un projet d'automatisation ou d'IA. N'échange avec Scout que par email.

## User Stories

1. **US-1** — En tant que prospect, je veux répondre à quelques questions simples avant de réserver (qui je suis, budget, démarrage, qui décide, ce que je veux résoudre), afin d'obtenir un appel adapté à mon projet.
2. **US-2** — En tant que prospect hors cible évident, je veux être réorienté tout de suite avec une alternative adaptée, afin de ne pas perdre mon temps.
3. **US-3** — En tant que prospect, je veux réserver directement puis savoir en quelques minutes si mon rendez-vous est confirmé, afin de ne pas rester dans le flou.
4. **US-4** — En tant que prospect non qualifié, je veux recevoir un message bienveillant qui m'oriente vers une alternative, afin de repartir avec une piste plutôt qu'avec un refus sec.
5. **US-5** — En tant que prospect dont la demande est en attente d'avis, je veux savoir quand j'aurai une réponse, afin de ne pas rester sans nouvelles.
6. **US-6** — En tant que prospect qualifié, je veux recevoir des rappels avant l'appel, afin de ne pas l'oublier.
7. **US-7** — En tant que prospect, je veux annuler ou décaler en un clic, afin que les rappels s'adaptent d'eux-mêmes, sans jamais arriver en double.
8. **US-8** — En tant que prospect, je veux pouvoir dire « ne plus être contacté » en un clic depuis n'importe quel email, afin que plus rien ne me parvienne.
9. **US-9** — En tant que consultant, je veux que Scout décide seul quand le cas est clair, afin de ne plus trier les demandes moi-même.
10. **US-10** — En tant que consultant, je veux être sollicité en un geste quand Scout hésite, sans que le prospect reste bloqué si je ne réponds pas, afin de garder la main sur les cas limites.
11. **US-11** — En tant que consultant, je veux recevoir une courte fiche sur le prospect avant chaque appel, afin d'arriver préparé.
12. **US-12** — En tant que consultant, je veux indiquer en un geste après l'appel si le prospect est venu et était pertinent, afin qu'un absent reçoive une seule proposition de reprogrammation et que je mesure l'effet du tri.
13. **US-13** — En tant que consultant, je veux un récap hebdomadaire des prospects écartés avec la possibilité d'en repêcher, afin de vérifier que Scout n'écarte pas de bons prospects.

## Critères de succès

1. Après 2 mois, moins de 25 % des appels marqués « Absent » ou « Présent – pas pertinent » (environ 50 % avant Scout).
2. Au plus 1 demande d'avis par semaine en moyenne sur 4 semaines.
3. 95 % des réservations hors doute reçoivent leur verdict par email en moins de 5 minutes.
4. 0 email en double et 0 rappel après une annulation, un décalage ou un « ne plus être contacté ».
5. 0 email envoyé sans consentement coché ; 100 % des emails contiennent le lien « ne plus être contacté ».
6. Fiche reçue au moins 55 minutes avant 100 % des appels confirmés.

## Hors périmètre

- Toute interface, tableau de bord ou écran de réglages.
- Plusieurs consultants ou plusieurs agendas.
- Autres langues que le français.
- Autres canaux que l'email pour le prospect : SMS, messageries.
- Suivi commercial après l'appel et synchronisation avec un CRM.
- Paiement ou acompte à la réservation.
- Offre d'entrée, orientation vers un confrère, relance des prospects invités à revenir plus tard.
- Recherche sur le prospect au-delà du site public de son entreprise.

## Décisions d'implémentation

- **Questionnaire** : identité, entreprise, site (facultatif), rôle ; taille, budget, démarrage et décideur en choix fermés ; besoin en texte libre obligatoire ; case de consentement obligatoire ; mention « demandes pré-analysées par un assistant ».
- **Cas évidents** : budget sous 3 000 € HT, particulier, indépendant seul ou plus de 50 personnes, démarrage dans plus de 6 mois ou inconnu. « Qui décide » n'élimine jamais d'office.
- **Qualifié** : besoin clair et dans le périmètre du consultant. Une analyse impossible ou un prospect déjà absent par le passé passe en doute.
- **Doute** : notification « Accepter » / « Refuser » avec une synthèse ; relance à 4 h ; sans réponse sous 24 h (ou avant le créneau si plus proche), le rendez-vous est confirmé par défaut. Le prospect reçoit entre-temps un email d'attente.
- **Rappels** : la veille avec liens « je décale » / « j'annule », puis 1 h avant avec le lien de l'appel. Pas de rappel de la veille si la réservation a lieu moins de 24 h avant.
- **Alternatives** : démarrage lointain → invitation à revenir quand le projet sera mûr ; tous les autres motifs → ressources gratuites.
- **Une seule demande active par prospect** : une nouvelle réservation remplace la précédente.
- **Ne plus être contacté** : arrêt immédiat de tous les envois ; un rendez-vous déjà prévu est maintenu et le consultant est prévenu.
- **Fiche** : par email 1 h avant, annoncée par notification. Elle contient les réponses, une synthèse du besoin, la raison du verdict, 3 questions à poser, quelques lignes sur l'entreprise et l'historique éventuel. Lisible sur téléphone.
- **Après l'appel** : notification « Absent » / « Présent – pertinent » / « Présent – pas pertinent ». « Absent » déclenche un seul email de reprogrammation.
- **Récap** : chaque lundi par email, avec les écartés et leur motif, un bouton « Repêcher » (lien de réservation direct) et les chiffres de la semaine.
- **Ton** : emails en français, au vouvoiement, signés du nom du consultant. Aucune notification pour les confirmations ou les refus.

## Notes complémentaires

- L'outil de prise de rendez-vous actuel doit pouvoir poser les questions avant l'agenda, réorienter les cas évidents et prévenir Scout des réservations, annulations et décalages.
- Le consultant fournit les ressources gratuites.
- Risque : un bon prospect écarté à tort ; le récap et le bouton « Repêcher » servent de filet.
- Le seuil de 3 000 € HT, la cible et le plafond d'avis sont à revoir après 2 mois.
