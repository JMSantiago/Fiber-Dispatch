# CRM Devis Bugbusters

Éditeur de devis Bugbusters : choix du DO, BPU rattachés et grille Mobility4 dans le panneau Prestations, PDF avec logo, en-tête et pied de page, historique des devis et suivi « Fil de l'eau » (activité, devis, production).

- **Site** : GitHub Pages (fichier `index.html`, aucun serveur à gérer).
- **Données** : Supabase, projet `crm-devis` (région Paris). Les BPU, la liste des DO et les devis sont dans la base, jamais dans ce dépôt.
- **Accès** : un code d'accès partagé par l'équipe. Sans ce code, la page ne charge aucune donnée.

## Comment fonctionne l'accès

Le code d'accès est le mot de passe d'un compte unique Supabase, `devis.equipe@bugbusters.fr`. Les règles de sécurité de la base (RLS) ne renvoient des données qu'à ce compte. La clé « publishable » visible dans `index.html` est publique par conception : seule, elle ne donne accès à rien.

À la connexion, chaque personne indique son prénom. Il est enregistré dans l'historique à chaque création, modification, changement de statut ou suppression.

## Mise en place (une seule fois)

1. **Créer le compte équipe** dans Supabase : *Authentication → Users → Add user → Create new user*.
   - Email : `devis.equipe@bugbusters.fr`
   - Mot de passe : le code d'accès que vous partagerez (12 caractères ou plus)
   - Cochez *Auto Confirm User*
2. **Fermer les inscriptions** : *Authentication → Sign In / Providers*, désactivez *Allow new users to sign up*.
3. **Charger les BPU** : ouvrez le site, entrez avec le code, allez dans *BPU & DO* et importez le fichier `catalog.json` (transmis à part, ne le déposez pas dans ce dépôt).

**Ajouter des BPU plus tard** : *BPU & DO → Importer des BPU (.json)*. L'import est additif : un aperçu indique les BPU nouveaux et mis à jour, rien n'est supprimé, et les rattachements DO ↔ BPU existants sont conservés. Un BPU qui porte le même identifiant qu'un BPU existant le remplace (nouvelle version de la grille).

Pour changer le code d'accès : *Authentication → Users*, ouvrez le compte équipe et changez son mot de passe. Les personnes déjà connectées gardent leur session jusqu'à leur déconnexion.

## Fichiers

| Fichier | Rôle |
|---|---|
| `index.html` | L'application |
| `supabase/schema.sql` | Structure de la base (tables, sécurité, historique, production), pour référence |
| `.nojekyll` | Demande à GitHub Pages de servir les fichiers tels quels |

## Historique

- Onglet **Historique** : toutes les actions récentes (qui, quoi, quand, montant).
- Bouton **Historique** sur chaque devis : ses versions, avec PDF de chaque version et restauration.
- Les devis supprimés vont dans la **Corbeille** (filtre de la liste) et peuvent être restaurés.
- Les enregistrements successifs d'une même personne sur 10 minutes sont regroupés en une ligne.

## Fil de l'eau

Onglet **Fil de l'eau**, reprise du fichier Excel « FIL DE L'EAU AUDIT-DEVIS » :

- **Activité** : tableau de bord calculé en direct (devis par statut, taux de transformation, panier moyen, production par statut, CA et marge, chiffres par CDP, relances à faire, 12 derniers mois).
- **Devis** : tous les devis avec CDP, activité, n° de ticket, date d'envoi, relances, jours sans relance et « À relancer » calculés comme dans l'Excel.
- **Production** : une ligne est créée automatiquement quand un devis passe à « Accepté » (statut « A PLANIFIER »). Date d'intervention, technicien, prix d'achat ; la marge et le % de marge sont calculés.
- **Missionnement** (dans Production) : tant qu'une intervention n'est pas planifiée, les cases Date, Heure et Technicien sont en rouge. Une fois remplies, la ligne passe « Planifié » et le bouton *Envoyer mail de missionnement* prépare le mail à partir du devis (tâches, longueurs, durée, tarif, adresse, formulaire à remplir). Lien du portail, mot de passe et contacts support se règlent dans *Listes* (enregistrés dans Supabase, pas dans ce dépôt).
- **Facturation** (dans Production) : filtres par DO, mois et « À facturer », puis *Extraction facturation* télécharge un Excel des lignes affichées (récapitulatif, détail des prestations des devis, synthèse par DO ; sans prix d'achat ni marge). Le n° de facture se note ligne par ligne ou en une fois sur les lignes affichées.
- **Listes** : répartition DO → CDP, délai de relance, listes des CDP, activités et techniciens.

Le PDF « DO » d'un brouillon le passe automatiquement en « Envoyé – en attente » avec la date du jour.

## Lire une demande (mail du DO ou compte rendu d'audit)

Bouton **« Lire une demande / un compte rendu »** dans l'éditeur de devis : collez le texte, le CRM repère le DO, le client, l'adresse, la référence (BDC, n° de commande), le contact sur place et le matériel (bornes, switch, routeur, postes, baie, PC, caméras, alarme, Starlink, câble, percements…), puis propose les prestations du BPU du DO avec les quantités. Tout est modifiable dans l'aperçu avant de remplir le devis ; plusieurs sites donnent un devis par site ou un devis avec un bloc par site. Un forfait négocié écrit dans le texte (« Site 1 : 1 journée 2 techniciens à 900 € HT », « ½ journée 2 techs à 500 € ») est repris tel quel, avec son prix, en ligne libre cochée « Forfait main d'œuvre » ; le matériel listé (« 1 x … ») y est indiqué comme fourni par le client (fourniture non comprise) et les prestations du BPU sont alors proposées décochées. Aucune IA externe : le texte n'est envoyé nulle part.

## Bons de commande

Onglet **Bons de commande**, séparé des devis DO (tables `bons_commande` et `fournisseurs`, historique propre) :

- **Bon de commande** : fournisseur (adresse et contact repris automatiquement), réf. de l'offre fournisseur, affaire / chantier, devis DO lié (facultatif), livraison (notre adresse mémorisée, chantier ou enlèvement en agence), articles avec prix « pour 1 / 100 / 1000 » (les « /C » de Sonepar ou Rexel), PDF avec logo. Télécharger le PDF d'un brouillon le passe en « Commandé ».
- **Lire une liste de matériel** : collez une liste (mail, compte rendu) ou des lignes d'offre fournisseur ; quantités, unités et prix sont repris (prix « pour 100 » compris), les articles déjà connus sont proposés. Le fournisseur et la livraison restent à choisir à la main.
- **Liste des bons** : filtres fournisseur, statut, mois ; export Excel ; versions et corbeille.
- **Fournisseurs** : créés ou complétés automatiquement à l'enregistrement d'un bon. Les articles déjà commandés sont proposés dans le panneau de droite des bons suivants.
- Dans **Production**, le montant des bons liés au devis s'affiche sous le prix d'achat.

## Gratuité et limites

GitHub Pages et l'offre gratuite de Supabase suffisent. Supabase met en pause un projet gratuit qui a très peu d'activité sur 7 jours ; quelques utilisations par jour suffisent à l'éviter. Un projet en pause se relance depuis le tableau de bord Supabase (*Resume project*) pendant 90 jours, avec ses données. Les sauvegardes de la base ne sont pas téléchargeables sur l'offre gratuite : utilisez *Société → Sauvegarde* dans l'application pour garder une copie des devis.
