# SPEC.md

## 1. Objectif

Piloter un système d'irrigation Home Assistant comprenant :

```
          Eau réseau
              │
              │
      Master Valve (Tuya)
              │
      ┌───────┴────────┐
      │                │
 Pelouse (TS0601)   Massif (TS0601)
```

Le système doit :

* être sûr hydrauliquement
* supporter les commandes physiques
* supporter les commandes Home Assistant
* être auto-réparable
* être extensible

---

## 2. Matériel

### Master
C'est un appareil de type Valve Controller (1fcnd8xk) intégré par Tuya

```
valve.arroseur_devant_valve
```

Type :

```
Valve
```

Actions :

```
open_valve

close_valve
```

---

### Vannes en aval - 2 Zones: 
Appareil TS0601 intégré avec zigbee2mqtt qui gère deux vannes
Type :

```
Switch
```

#### Zone Pelouse

```
switch.valve_pelouse
```



---

#### Zone Massif

```
switch.valve_massif
```

---

## 3. Etats du contrôleur

```
IDLE
```

Toutes les vannes fermées.

---

```
OPENING
```

Ouverture en cours.

---

```
RUNNING
```

Arrosage.

---

```
STOPPING
```

Arrêt hydraulique.

---

```
PURGING
```

Décompression.

---

```
ERROR
```

Anomalie.

---

```
MAINTENANCE
```

Mode manuel.

---

## 4. Etats des zones

Chaque zone possède :

```
OFF

STARTING

RUNNING

STOPPING

ERROR
```

---

## 5. Séquence d'ouverture

```
Utilisateur

↓

Zone ON

↓

Attendre délai configurable

↓

Master ON

↓

Timer démarre

↓

RUNNING
```

---

## 6. Séquence d'arrêt

```
Fin timer

↓

Master OFF

↓

Attendre délai purge

↓

Zone OFF

↓

IDLE
```

---

## 7. Si une autre zone est encore active

```
Pelouse OFF

↓

Massif ON

↓

Master reste ON
```

---

## 8. Si Master ouverte seule

```
Master ON

↓

aucune zone

↓

ouvrir Massif
```

Protection hydraulique.

---

## 9. Si zone ouverte sans Master

```
Pelouse ON

↓

Master OFF

↓

ouvrir Master
```

---

## 10. Watchdog

Toutes les 10 secondes.

Contrôle :

✅ Master

✅ Pelouse

✅ Massif

✅ Timers

✅ Synchronisation

✅ Disponibilité Zigbee

✅ Disponibilité Tuya

---

## 11. Health Monitor

Le système calcule :

```
100 %
```

si tout est normal.

Sinon :

```
82 %

65 %

40 %
```

selon le nombre d'erreurs.

---

## 12. Notifications

Au minimum :

```
Début arrosage

Fin arrosage

Erreur Zigbee

Erreur Tuya

Watchdog

Temps dépassé
```

---

## 13. Dashboard

Deux vues.

### Exploitation

```
Master

Pelouse

Massif

Timers

STOP

START
```

---

### Diagnostic

```
Historique

RSSI

Compteurs

Erreurs

Health

Journal
```

---

## 14. Paramètres

```
Durée Pelouse

Durée Massif

Délai ouverture

Délai purge

Intervalle watchdog

Timeout Zigbee

Timeout Tuya
```

---

## 15. Journalisation

Chaque transition est enregistrée.

Exemple :

```
08:12:41

OPEN_MASTER

08:12:43

RUNNING

08:58:41

PURGING

08:58:44

IDLE
```

---

## Extensibilité - Evolutivité
Au lieu de coder les zones en dur (`Pelouse`, `Massif`), on va rendre le contrôleur **entièrement générique**.

Par exemple, dans un helper (ou un fichier de configuration), on définirait simplement :

```yaml
zones:
  - name: Pelouse
    entity: switch.valve_pelouse
    duration: 48

  - name: Massif
    entity: switch.valve_massif
    duration: 48
```

Toute la logique (scripts, timers, watchdog, dashboard) utiliserait cette liste.

Ainsi, si on ajoute demain :

```yaml
- name: Potager
  entity: switch.valve_potager
  duration: 30
```

le contrôleur fonctionnerait sans modification du code métier.

👉 Cette approche rend le système beaucoup plus évolutif et réutilisable. C'est également celle que l'on retrouve dans les logiciels industriels de gestion d'irrigation.
