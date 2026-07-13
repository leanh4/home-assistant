# Smart Irrigation Controller for Home Assistant
v1.0



## 0. Organisation des fichiers

```text
/config

├── configuration.yaml
│
├── helpers.yaml
├── templates.yaml
├── scripts.yaml
├── automations.yaml
│
└── dashboards/
    └── irrigation.yaml
```

---

## 1. Rôle des fichiers

### `helpers.yaml`

Contient uniquement les paramètres configurables dans l'UI :

* durée Pelouse ;
* durée Massif ;
* marge sécurité Master ;
* délai ouverture hydraulique ;
* délai purge ;
* timeout Master ;
* mode irrigation.

Ces entités seront visibles dans :

```
Paramètres
 → Appareils et services
 → Assistants
```

---

### `templates.yaml`

Contient les capteurs calculés :

Exemples :

```text
sensor.irrigation_status
```

Valeurs :

```
REPOS
DEMARRAGE
ARROSAGE
PURGE
SECURITE
ERREUR
```

---

```text
sensor.irrigation_active_zones
```

Exemple :

```
Pelouse
```

ou :

```
Pelouse, Massif
```

---

```text
sensor.irrigation_health
```

Exemple :

```
100 %
```

---

### `scripts.yaml`

Tous les scripts seront visibles ici :

```
Paramètres
 → Automatisations et scènes
 → Scripts
```

Scripts prévus :

#### Commandes utilisateurs

```
Irrigation - Démarrer zone
Irrigation - Arrêter zone
Irrigation - Arrêt complet
```

---

#### Sécurité

```
Irrigation - Fermeture sécurité Master
Irrigation - Purge hydraulique
Irrigation - Réparer incohérence
```

---

#### Maintenance

```
Irrigation - Test Pelouse
Irrigation - Test Massif
Irrigation - Test Master
```

---

### `automations.yaml`

Toutes les règles seront visibles dans l'interface :

#### Gestion normale

```
Irrigation - Demande ouverture Pelouse
Irrigation - Demande ouverture Massif
Irrigation - Fin countdown Pelouse
Irrigation - Fin countdown Massif
```

---

#### Sécurité

```
Irrigation - Master ouverte plus de 60 minutes
Irrigation - Fermeture quotidienne minuit
Irrigation - Master seule ouverte
Irrigation - Zone ouverte sans Master
```

---

#### Surveillance

```
Irrigation - Watchdog toutes les 10 secondes
Irrigation - Notification anomalie
```

---

# 2. Modification de configuration.yaml

Nous utiliserons les includes natifs :

```yaml
script: !include scripts.yaml

automation: !include automations.yaml

template: !include templates.yaml

input_number: !include helpers.yaml
```

---

# 3. Entités matérielles utilisées

On fige :

## Master

```yaml
valve.arroseur_devant_valve
```

Durée sécurité :

```yaml
number.arroseur_devant_irrigation_duration
```

---

## Pelouse

Commande :

```yaml
switch.valve_pelouse
```

Countdown :

```yaml
number.valve_jardin_countdown_pelouse
```

---

## Massif

Commande :

```yaml
switch.valve_massif
```

Countdown :

```yaml
number.valve_jardin_countdown_massif
```

---

# 4. Séquence hydraulique finale

## Ouverture

```text
1. Programmer countdown zone
2. Ouvrir zone
3. Attendre 1 seconde
4. Ouvrir Master
5. Arrosage
```

---

## Fermeture normale

```text
1. Fermer Master
2. Attendre purge
3. Fermer zone
```

---

## Arrêt sécurité

```text
1. Fermer Master immédiatement
2. Attendre purge
3. Fermer Pelouse
4. Fermer Massif
5. Réinitialiser countdown
6. Notification
```

---


