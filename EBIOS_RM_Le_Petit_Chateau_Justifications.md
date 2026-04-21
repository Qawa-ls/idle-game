# Étude EBIOS RM — Clinique Le Petit Chateau
## Document de justification et décisions

**Version** : 1.0
**Date** : 2026-04-21
**Outil** : CISO Assistant (intuitem community)
**Référentiel méthodologique** : EBIOS Risk Manager (ANSSI, 2018)
**Matrice de risque** : 4×4 EBIOS-RM (Vraisemblance V1-V4 × Gravité G1-G4)
**Socle compliance** : Guide d'hygiène informatique ANSSI

---

## 1. Contexte et objet de l'étude

### 1.1 Présentation de la clinique

La clinique **Le Petit Chateau** est un établissement de rééducation hébergeant des données de santé à caractère personnel (DCP santé). Elle relève à ce titre de :
- RGPD (catégories particulières, art. 9) ;
- Réglementation santé française (PGSSI-S, HDS le cas échéant) ;
- Obligation de signalement ANS (incidents de sécurité SI de santé) ;
- Programme CaRE (résilience, continuité d'activité).

Le SI combine :
- **Applications métier historiques** (RééducSoft sur VM Windows Server 2008, PharmWin) ;
- **Postes utilisateurs Windows 8** (fin de support Microsoft) ;
- **Virtualisation VMware 6.5** (fin de support éditeur) ;
- **Accès distants** VPN médecins + serveur RDS externe ;
- **Prestataires critiques** : PidouCloud (hébergeur), Archanlait (éditeur métier), Gilbert & fils (maintenance), Orange (opérateur) ;
- **Données sensibles** : DMP/MSSanté, dossiers médicaux, pharmacie, administratif patient, RH/paie.

### 1.2 Objet de l'analyse

L'étude EBIOS RM vise à :
1. Identifier les scénarios de menace cyber les plus critiques pour la continuité des soins et la confidentialité des données patients ;
2. Prioriser les mesures de sécurité du plan de traitement ;
3. Nourrir le Socle de sécurité (Guide d'hygiène ANSSI) et le futur plan d'action PCA/PRA ;
4. Fournir une analyse de risque exploitable par la direction et l'ARS.

### 1.3 Rappel méthode EBIOS RM

| Atelier | Objet |
|---|---|
| 1 | Cadrage, socle, biens essentiels/supports, événements redoutés |
| 2 | Identification des sources de risque (SR) et objectifs visés (OV) — couples RO/TO |
| 3 | Scénarios stratégiques — parcours macroscopiques SR → bien essentiel, via parties prenantes |
| 4 | Scénarios opérationnels — déclinaison technique (modes opératoires, kill chain, vraisemblance) |
| 5 | Traitement du risque — stratégie, mesures, risque résiduel |

---

## 2. Atelier 1 — Cadrage et socle

### 2.1 Périmètre

- **Périmètre métier** : toute activité de prise en charge patient (accueil → soins → facturation).
- **Périmètre SI** : l'ensemble du SI interne, les accès distants, les interfaces avec prestataires.
- **Hors périmètre** : les SI métier des partenaires (CPAM, mutuelles) au-delà de leurs flux d'échange.

### 2.2 Biens essentiels (valeurs métier)

| Bien essentiel | Critère DICT dominant | Justification |
|---|---|---|
| Dossier médical patient (RééducSoft + DMP) | Confidentialité, Intégrité, Disponibilité | Donnée de santé protégée par RGPD art.9 + secret médical CSP L.1110-4 |
| Données pharmaceutiques (PharmWin) | Intégrité, Disponibilité | Erreur = risque vital (posologie) |
| Données administratives patients | Confidentialité, Intégrité | RGPD + facturation Sécu |
| Processus de prise en charge | Disponibilité | Continuité des soins |
| Données RH et paie | Confidentialité | RGPD + obligation employeur |

### 2.3 Biens supports retenus

23 biens supports sont listés dans l'étude, regroupés en 4 familles :
- **Infrastructure** : Cluster VMware 6.5, Contrôleurs de domaine, NAS Synology, Salle serveurs ;
- **Réseau** : Firewall Stormshield SN920, Commutateurs ARUBA, VPN médecins, Serveur RDS externe ;
- **Postes et OS** : Postes Windows 8, Serveur RéducSoft Win2008 ;
- **Applications** : RééducSoft, PharmWin, Sage ;
- **Sécurité** : EDR SentinelOne ;
- **Acteurs** : Personnel médical, Patients.

### 2.4 Socle de sécurité retenu

**Guide d'hygiène informatique ANSSI** (42 règles) — choix justifié par :
- Clinique taille moyenne, maturité cyber émergente : guide calibré pour ce profil ;
- Reconnu par l'ANS et les ARS, réutilisable dans les audits de certification HDS ;
- Couvre les fondamentaux (identités, postes, réseaux, sauvegarde, journalisation) prioritaires avant toute certification ISO 27001.

La conformité au socle est évaluée dans l'assessment **"Socle de sécurité : Le Petit Chateau"** (statut `in_progress`).

### 2.5 Événements redoutés (ER) et gravité

La gravité est cotée sur 4 niveaux (G1 négligeable → G4 critique). Elle traduit l'impact sur les biens essentiels.

| ER | Gravité | Justification |
|---|---|---|
| Indisponibilité de fonctions applicatives médicales | **G3** | Arrêt de prise en charge, transferts patients vers autres établissements, perte de chance |
| Indisponibilité des données de santé | **G3** | Idem — impossibilité de consulter historique, prescriptions |
| Altération des fonctions applicatives médicales | **G2** | Dégradation dégradée mais fonctionnement partiel possible |
| Altération des données de santé | **G3** | Risque vital (posologie erronée, dossier corrompu) + atteinte intégrité RGPD |
| Accès (divulgation) aux données de santé | **G3** | Violation RGPD art.9, sanction CNIL + atteinte réputationnelle + usage illégitime (chantage, discrimination) |

**Pas d'ER en G4** car aucun scénario identifié ne menace l'existence même de l'établissement à court terme (taille moyenne, pas de statut SIIV). Les G3 sont néanmoins très sérieux et justifient le traitement.

---

## 3. Atelier 2 — Sources de risque / Objectifs visés (RO/TO)

### 3.1 Principe

Pour chaque couple (Source de risque × Objectif visé), on évalue :
- **Motivation** (0-4) : intérêt de la SR à atteindre l'OV ;
- **Ressources** (0-4) : capacités techniques, financières, humaines ;
- **Activité** (0-4) : fréquence observée dans l'écosystème (menace temps réel).

La **pertinence** pour la suite de l'étude découle de la combinaison des trois. Un RO/TO à motivation+ressources+activité faibles est écarté (`is_selected=False`) ; les autres alimentent les scénarios stratégiques.

### 3.2 Couples retenus et justifications

| # | Source de risque | Objectif visé | Mot. | Res. | Act. | Décision |
|---|---|---|---|---|---|---|
| 1 | **Organized crime** | Chiffrer les données patients et le SI pour exiger une rançon (double extorsion) | 4 | 3 | 4 | **Retenu** |
| 2 | **Avenger** (patient/ex-salarié rancunier) | Accéder/exfiltrer données patients (revente, curiosité, vengeance) | 4 | 2 | 3 | **Retenu** |
| 3 | **Amateur** (script kiddie, opportuniste) | Accès frauduleux données, vol matériel, perturbation | 3 | 2 | 3 | **Retenu** |
| 4 | **Competitor** (clinique concurrente) | Déstabilisation concurrentielle | 2 | 1 | 2 | Retenu avec réserve |
| 5 | **Other — prestataires maintenance** | Accès non maîtrisé au SI via accès légitimes de maintenance | 1 | 3 | 2 | **Retenu** |
| 6 | **Other — erreur humaine / défaillance involontaire** | Aucun objectif intentionnel (vecteur involontaire) | 1 | 1 | 1 | Retenu (couverture complète) |

### 3.3 Justifications détaillées

#### 3.3.1 Organized crime — le risque n°1 en santé
- **Motivation 4** : les groupes ransomware ciblent explicitement le secteur hospitalier depuis 2019 (Ryuk, Conti, LockBit, Rhysida). Le levier de pression (vies en jeu, données critiques) garantit un taux de paiement supérieur à d'autres secteurs.
- **Ressources 3** : groupes professionnalisés, écosystème RaaS (Ransomware-as-a-Service), accès aux 0days via brokers.
- **Activité 4** : incidents réguliers en France (CHU Corbeil, CH Versailles, Armentières, etc.). Menace considérée comme **certaine à moyen terme**.
- **Conséquence** : scénario SS01 / RS.01 est le pilier du plan de traitement.

#### 3.3.2 Avenger — la menace insider
- **Motivation 4** : un patient ayant subi un préjudice perçu (conflit médical, refus d'acte) peut chercher à nuire. Un ex-salarié (IDE, secrétaire, prestataire) conserve potentiellement des accès ou des connaissances exploitables.
- **Ressources 2** : sans accès privilégié ni outillage offensif, mais connaissance métier forte.
- **Activité 3** : fréquent dans le retour d'expérience hospitalier, souvent sous-détecté.
- **Priorité du traitement** : revue des accès à privilèges, offboarding, journalisation.

#### 3.3.3 Amateur — la surface d'attaque facile
- **Motivation 3** : trophée, défi technique, curiosité.
- **Ressources 2** : outillage public (Metasploit, Nuclei), compétences limitées mais suffisantes face à un SI obsolète.
- **Activité 3** : scans automatisés permanents sur Internet ; les CVE Windows Server 2008 sont triviales à exploiter.
- Ce RO/TO est **cohérent avec la vulnérabilité OS obsolètes** (SS04).

#### 3.3.4 Competitor — faible mais conservé
- **Motivation 2 / Ressources 1 / Activité 2** : peu probable dans le secteur médico-social privé (concurrence régulée), mais impossible à exclure totalement.
- **Justification du maintien** : permet de couvrir les scénarios de réputation (déstabilisation médiatique, fuites orchestrées).

#### 3.3.5 Other — prestataires maintenance (supply chain)
- **Ressources 3** : c'est la particularité — le prestataire possède des **accès légitimes** (maintenance RééducSoft par Archanlait, hébergement par PidouCloud). Le "ressources" capture ici la capacité d'accès.
- **Activité 2** : incidents supply chain en croissance (SolarWinds, Kaseya, Centreon en France 2021).
- **Criticité stakeholders** (voir §4.3) : PidouCloud 5.33 et Archanlait 3.0 dominent largement — justifie un scénario dédié (SS02).

#### 3.3.6 Other — erreur involontaire
- Activité/motivation/ressources toutes 1 : par définition non intentionnel.
- **Pourquoi le garder** : EBIOS RM préconise une couverture complète des vecteurs. Les événements non malveillants (erreur de saisie, défaillance matérielle) sont traités en atelier 3 et écartés en atelier 4 au profit du PCA/PRA (SS06, SS08 non retenus pour scénarios opérationnels).

---

## 4. Atelier 3 — Scénarios stratégiques

### 4.1 Parties prenantes et criticité

La criticité d'une partie prenante dans EBIOS RM se calcule comme :

```
criticité = (dépendance × pénétration) / (maturité × confiance)
```

Plus la clinique **dépend** du prestataire, plus il a **pénétration** dans le SI, moins il est **mature** en sécurité, et moins il inspire **confiance**, plus il est critique.

| Partie prenante | Catégorie | Criticité actuelle | Résiduelle |
|---|---|---|---|
| **PidouCloud** | Contractor (hébergeur) | **5.33** | 5.33 |
| **Archanlait** | Contractor (éditeur RééducSoft) | **3.00** | 3.00 |
| Gilbert & fils | Contractor (maintenance) | 1.33 | 1.00 |
| PCSOFT | Supplier | 0.50 | 0.50 |
| CPAM | Partner | 0.50 | 0.50 |
| Wantel | Contractor | 0.33 | 0.33 |
| Orange | Supplier | 0.25 | 0.25 |
| Mutuelles | Partner | 0.17 | 0.17 |

**Lecture** : PidouCloud et Archanlait concentrent 80% du risque tiers. Ils sont **la cible privilégiée** d'un attaquant visant la clinique via supply chain. D'où les scénarios SS02 / PC.01 / PC.03.

### 4.2 Les 8 scénarios stratégiques

Chaque SS est un couplage **(RO/TO × Événement redouté)** — il décrit *quelle source de risque vise quel impact*, sans encore détailler la technique.

| Réf | Scénario stratégique | RO/TO source | ER focalisé | Retenu |
|---|---|---|---|---|
| **SS01** | Code malveillant (ransomware) | Organized crime → chiffrement données | Indispo données de santé (G3) | ✅ |
| **SS02** | Supply chain attaque | Organized crime → chiffrement | Accès données santé (G3) | ✅ |
| **SS03** | Intrusion informatique | Avenger → exfiltration | Accès données santé (G3) | ✅ |
| **SS04** | Détournement usage logiciel | Amateur → accès frauduleux | Altération fonctions (G2) | ✅ |
| **SS05** | Saturation des applications | Competitor → déstabilisation | Indispo fonctions (G3) | ✅ |
| SS06 | Erreur de saisie / commande | Other involontaire → altération | Altération données (G3) | ❌ |
| SS07 | Perte ou sortie matériel | Amateur → vol données | Accès données (G3) | ❌ |
| SS08 | Défaillance matérielle / manque maintenance | Other involontaire → indispo | Altération fonctions (G2) | ❌ |

### 4.3 Justification du filtre (5 retenus / 3 écartés)

**Retenus** : scénarios cyber à intention malveillante, vraisemblance non négligeable, impact G2-G3 — cœur de la démarche EBIOS.

**Écartés** (SS06, SS07, SS08) :
- **SS06 Erreur de saisie** : traité par contrôle métier (double validation prescription, revue secrétariat). Hors focus cyber.
- **SS07 Perte de matériel** : impact limité si le **chiffrement de disque** est déployé (contrôle existant/prévu). Le risque résiduel est acceptable.
- **SS08 Défaillance matérielle** : relève du **PCA/PRA** (plan continuité / reprise) et non de l'analyse cyber. Traité dans le cadre du programme CaRE D2.

**Attention** : écarter ces scénarios ne signifie pas ignorer le risque — ils sont traités par d'autres dispositifs (plan qualité, PCA, politique poste de travail).

---

## 5. Atelier 3bis — Chemins d'attaque

Un chemin d'attaque (*attack path*) concrétise un scénario stratégique en précisant **le parcours technique** : par où passe l'attaquant, via quelle partie prenante.

| Réf | Chemin d'attaque | SS parent | Partie prenante pivot |
|---|---|---|---|
| **PC.01** | Accès illégitimes via Archanlait (éditeur métier) | SS02 | Archanlait |
| **PC.02** | Ransomware via phishing du personnel soignant | SS01 | — (vecteur direct) |
| **PC.03** | Compromission PidouCloud (hébergeur) | SS02 | PidouCloud |
| **PC.04** | Intrusion via VPN médecins / RDS externe | SS03 | — |
| **PC.05** | Exploitation Windows 8 / Serveur 2008 RééducSoft | SS04 | — |
| **PC.06** | Saturation applicative RDS / VPN | SS05 | — |

### 5.1 Justifications des choix

- **PC.01 lié à Archanlait** plutôt que PidouCloud : Archanlait dispose d'accès spécifiques à la base RééducSoft (données patients) — le chemin d'exfiltration est plus direct. PidouCloud est couvert par PC.03.
- **PC.02 en chemin direct** (pas de stakeholder) : le phishing frappe l'utilisateur final de la clinique, sans intermédiaire tiers.
- **PC.03 PidouCloud** : scénario séparé de PC.01 car le mode opératoire est différent (compromission de l'infrastructure cloud vs abus d'accès maintenance).
- **PC.04 VPN médecins** : la vulnérabilité clé est l'**absence de MFA** sur les accès distants.
- **PC.05 OS obsolètes** : Windows 8 et Serveur 2008 sont hors support ; CVE non patchables (SMBv1, BlueKeep, EternalBlue). Un scan externe suffit à trouver une porte d'entrée.
- **PC.06 saturation** : impact opérationnel (indispo pendant les heures de soins). Pas d'exfiltration, pas de persistance.

---

## 6. Atelier 4 — Scénarios opérationnels et kill chains

### 6.1 Vraisemblance

Cotée sur 4 niveaux (V1 Peu vraisemblable → V4 Quasi certain). Intègre :
- **Difficulté technique** pour l'attaquant ;
- **Exposition** du bien cible ;
- **Contrôles existants** (EDR, pare-feu, formation).

### 6.2 Synthèse des scénarios opérationnels

| Réf | Scénario | Vraisemblance | Menaces MITRE ATT&CK principales |
|---|---|---|---|
| **PC.01** | Supply chain Archanlait | **V2** (Likely) | Trusted Relationship, Valid Accounts, External Remote Services |
| **PC.02** | Ransomware phishing → chiffrement global | **V3** (Very Likely) | Spearphishing Attachment, Valid Accounts, Ransomware, Data Encrypted for Impact, Impair Defenses |
| **PC.03** | Compromission PidouCloud | **V2** | Supply Chain Compromise, Trusted Relationship, Valid Accounts, Data Encrypted for Impact |
| **PC.04** | Intrusion VPN médecins | **V3** | Valid Accounts, External Remote Services, Exploitation of Remote Services |
| **PC.05** | Exploitation OS obsolètes | **V2** | Exploit Public-Facing Application, Exploitation of Remote Services |
| **PC.06** | Saturation applicative | **V2** | Network Denial of Service, Endpoint Denial of Service |

### 6.3 Justification des vraisemblances

- **V3 Ransomware (PC.02)** : secteur hospitalier surciblé, phishing très actif, postes Windows 8 vulnérables aux loaders modernes. Le seul rempart sérieux est l'EDR — mais un EDR peut être désactivé après escalade de privilèges.
- **V3 Intrusion VPN (PC.04)** : l'**absence de MFA** rend le scénario trivial dès qu'un identifiant médecin fuite (phishing, infostealer, réutilisation de mot de passe). C'est la vulnérabilité n°1.
- **V2 Supply chain (PC.01/PC.03)** : attaque sophistiquée mais écosystème de prestataires santé régulièrement ciblé. Moins fréquent qu'un phishing mais impact massif.
- **V2 OS obsolètes (PC.05)** : dépend de l'exposition effective — si les serveurs ne sont pas directement exposés Internet, le chemin requiert une compromission préalable.
- **V2 Saturation (PC.06)** : outillage DDoS accessible, mais sans persistance — impact temporaire, rapidement mitigé par Orange.

### 6.4 Kill chain détaillée — PC.02 (Ransomware)

Mode opératoire : **OM-PC02-01** — *Ransomware opéré (phishing → chiffrement global)*.

| Étape | Action | Stade ATT&CK | Menace MITRE |
|---|---|---|---|
| 1 | Reconnaissance OSINT équipe médicale | Reconnaissance | Gather Victim Host Information |
| 2 | Envoi spearphishing ciblé (PJ Word macro) | Initial Access | Spearphishing Attachment |
| 3 | Exécution loader persistant | Initial Access | Valid Accounts |
| 4 | Dump credentials (Mimikatz, LSASS) | Discovery | Valid Accounts |
| 5 | Reconnaissance AD (BloodHound, DC, NAS) | Discovery | System Owner/User Discovery |
| 6 | Neutralisation EDR (désactivation SentinelOne) | Exploitation | Impair Defenses |
| 7 | Destruction sauvegardes NAS | Exploitation | Data Encrypted for Impact |
| 8 | **Chiffrement global via GPO/PsExec** | Exploitation | Ransomware ⚠️ |

**Points de rupture** (*choke points*) identifiés — là où un contrôle peut casser la chaîne :
- Étape 2-3 : formation anti-phishing, filtrage mail avancé, macro désactivées par GPO ;
- Étape 4 : credential guard, segmentation des comptes, Tier Model AD ;
- Étape 6 : EDR avec protection anti-tampering ;
- Étape 7 : **sauvegarde immuable** (air-gap, WORM) → seul rempart garantissant la reprise sans rançon.

### 6.5 Kill chain détaillée — PC.03 (Supply chain PidouCloud)

Mode opératoire : **OM-PC03-01** — *Compromission hébergeur PidouCloud*.

| Étape | Action | Stade ATT&CK |
|---|---|---|
| 1 | Recon infrastructure PidouCloud | Reconnaissance |
| 2 | Credential stuffing portail admin | Initial Access |
| 3 | Accès console cloud (admin) | Discovery |
| 4 | Identification VMs clinique Le Petit Chateau | Discovery |
| 5 | Snapshot disques dossiers patients | Exploitation |
| 6 | **Exfiltration + double extorsion** ⚠️ | Exploitation |

**Points de rupture** :
- Étape 2 : MFA obligatoire sur le portail admin PidouCloud, rotation des secrets, audit réguliers (PAS - Plan d'Assurance Sécurité) ;
- Étape 3 : cloisonnement des tenants, chiffrement des disques côté client avec clés non détenues par l'hébergeur (BYOK) ;
- Étape 5 : chiffrement de base côté application (niveau rangée) — le snapshot est alors inutilisable.

### 6.6 Scénarios simplifiés (pas de kill chain détaillée)

PC.01, PC.04, PC.05, PC.06 ont un `operating_modes_description` textuel mais **pas de kill chain graphique** — choix pragmatique :
- Profondeur de modélisation priorisée sur les 2 scénarios les plus critiques (ransomware + supply chain hébergeur) ;
- Les autres scénarios restent exploitables pour le plan de traitement via leur description textuelle ;
- Extensible ultérieurement si besoin d'étoffer.

---

## 7. Atelier 5 — Stratégie de traitement

### 7.1 Analyse de risque générée

- **Périmètre rattaché** : "Clinique Le Petit Chateau" (créé car absent initialement).
- **Risk assessment** : `RA-PC-EBIOS-01` — *Analyse de risque EBIOS RM — Le Petit Chateau*.
- **Matrice** : 4×4 EBIOS RM.
- **Tolérance au risque** : 2 (niveau moyen) — c.à.d. tout risque ≥3 (élevé/critique) doit être traité en priorité.

### 7.2 Risk scenarios et niveaux

Chaque risque est coté à trois moments :
- **Inhérent** : sans aucun contrôle (scénario "nu") ;
- **Actuel** : avec les contrôles existants à date ;
- **Résiduel** : après application du plan de traitement.

| Réf | Scénario | Inhérent (P/I/L) | Actuel | Résiduel cible | Stratégie |
|---|---|---|---|---|---|
| **RS.01** | Ransomware chiffrement global | V3 G3 — **critique** | V3 G3 | V1 G3 | Mitigate |
| **RS.02** | Supply chain PidouCloud | V2 G3 — élevé | V2 G3 | V1 G3 | Mitigate |
| **RS.03** | Intrusion VPN médecins | V3 G3 — **critique** | V3 G3 | V1 G3 | Mitigate |
| **RS.04** | Exploitation OS obsolètes | V2 G3 | V2 G3 | V1 G2 | Mitigate |
| **RS.05** | Accès illégitimes via Archanlait | V2 G3 | V2 G3 | V1 G2 | Mitigate |
| **RS.06** | Saturation applicative | V2 G3 | V2 G2 | V1 G2 | Mitigate |

### 7.3 Justification des niveaux

- **Aucun risque accepté en l'état** : la gravité G3 sur des données de santé rend l'acceptation non défendable vis-à-vis du RGPD et de l'ARS.
- **Résiduel maintenu à G3** sur RS.01-RS.03 : l'impact intrinsèque (violation de données santé, indispo soins) ne baisse pas avec les contrôles techniques — seule la vraisemblance peut chuter. C'est conforme à la logique EBIOS : **les contrôles réduisent la probabilité, pas l'enjeu métier**.
- **Résiduel G2 possible** sur RS.04-RS.06 : les contrôles (retrait OS obsolètes, PCA, pare-feu) réduisent aussi l'amplitude de l'impact (un service dégradé plutôt qu'indisponible).
- **Passage V3→V1 sur RS.01/RS.03** : atteignable uniquement avec un bouquet complet (MFA + EDR durci + sauvegarde immuable + formation). Si un seul contrôle manque, la cible n'est pas tenue.

### 7.4 Mesures de traitement mappées

| Risk scenario | Contrôles rattachés |
|---|---|
| **RS.01 Ransomware** | Capacité de sauvegarde immuable ; EDR ; Plan de sensibilisation ; Politique anti-malware ; Antivirus/pare-feu poste |
| **RS.02 PidouCloud** | Politique sécurité fournisseurs tiers ; Plan d'Assurance Sécurité (PAS) |
| **RS.03 VPN médecins** | Système gestion mots de passe ; Contrôle d'accès réseau 802.1X ; Politique contrôle d'accès |
| **RS.04 OS obsolètes** | Procédure GPO ; Système unifié gestion des terminaux |
| **RS.05 Archanlait** | Politique fournisseurs tiers ; Procédure gestion accès à privilèges ; Journalisation des accès |
| **RS.06 Saturation** | Politique de continuité d'activité |

### 7.5 Priorités du plan de traitement

Ordre de traitement recommandé (rapport criticité × effort) :

1. **MFA sur VPN et RDS externes** — quick win le plus impactant (RS.03 V3→V1).
2. **Sauvegarde immuable opérationnelle et testée** — pilier anti-ransomware (RS.01).
3. **Retrait ou isolement Windows 8 / Serveur 2008** — supprime RS.04 à la racine.
4. **Renforcement PAS avec PidouCloud et Archanlait** — clauses de sécurité, audit, reporting, tests d'intrusion annuels (RS.02, RS.05).
5. **Formation anti-phishing et exercices de crise** — double effet sur RS.01 et RS.03.
6. **Supervision SOC et journalisation centralisée** — détection transversale.

---

## 8. Limites et prochaines étapes

### 8.1 Limites assumées de cette version

- **v0.1** de l'étude : la maille des scénarios peut être affinée (séparation exfiltration / chiffrement dans le ransomware par exemple).
- Kill chain détaillée sur 2 scénarios uniquement (PC.02, PC.03). Les 4 autres ont un mode opératoire textuel — suffisant pour le plan de traitement mais à enrichir avant une présentation à l'ARS.
- Parties prenantes : seule Archanlait est rattachée à PC.01 et PidouCloud à PC.03. Les autres chemins pourraient être reliés à des stakeholders internes (direction IT, médecins, personnel administratif) modélisés plus finement.
- Cotations de vraisemblance basées sur le jugement d'expert — à confronter à des données d'incidents réels (CERT Santé, FSSI) dans une v1.0.

### 8.2 Prochaines étapes

1. **Soutenance / validation** par la direction et le RSSI externalisé.
2. **Compléter le Socle ANSSI** (évaluation en cours `in_progress` — objectif 80% de conformité d'ici 6 mois).
3. **Plan d'audit** : tests d'intrusion externe + audit organisationnel annuel.
4. **Exercice de crise ransomware** — simulation du scénario RS.01 avec direction médicale.
5. **Contractualisation** : renforcer les annexes sécurité avec PidouCloud et Archanlait (PAS).
6. **Révision de l'étude** : tous les 12 mois ou à tout changement majeur (nouveau prestataire, migration applicative, incident).

---

## 9. Annexe — Traçabilité des décisions

### 9.1 Pourquoi la matrice 4×4 EBIOS et pas 5×5 ?

La matrice 4×4 est le défaut EBIOS RM ANSSI. Elle suffit à discriminer les niveaux de risque sans sur-raffinement (qui serait illusoire vu l'incertitude des cotations). Une 5×5 serait envisageable pour un acteur SIIV ou OIV — non pertinent ici.

### 9.2 Pourquoi un RA séparé et pas dérivé automatiquement ?

CISO Assistant permet de générer un RiskAssessment depuis une étude EBIOS, mais le lien `ebios_rm_study` a été rempli manuellement pour garder la maîtrise du nommage, de la version et du rattachement au périmètre "Clinique Le Petit Chateau" (qu'il a fallu créer).

### 9.3 Pourquoi ne pas accepter certains risques ?

Acceptation (`treatment=accept`) envisagée uniquement si :
- Risque résiduel ≤ tolérance (ici 2) ;
- Impact métier et réglementaire compatibles (RGPD, secret médical).
Aucun des 6 RS n'entre dans ce cadre. Tous sont en **mitigate**.

### 9.4 Pourquoi le Socle ANSSI et pas ISO 27001 comme framework de base ?

- Maturité actuelle de la clinique ne justifie pas une certification ISO ;
- Le Guide d'hygiène ANSSI (42 règles) est la marche la plus logique — plus pragmatique, reconnu, parfaitement aligné avec les exigences ARS et PGSSI-S ;
- ISO 27001 reste un objectif à 2-3 ans une fois le socle assuré.

---

**Fin du document.**
