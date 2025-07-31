# Playbook de Mise à Jour Automatisée Stormshield EVA

## Vue d’ensemble

Ce dépôt fournit un processus sécurisé, automatisé et traçable pour la mise à jour des appliances Stormshield EVA sur vCenter 7 à l’aide d’Ansible. 
Toutes les données sensibles (identifiants, IP, ports) sont gérées via Ansible Vault.

---

## Scénarios & Commandes Courants

### 1. Première Installation (Initialisation)

```bash
# Installer les collections Ansible requises
ansible-galaxy collection install stormshield.sns community.vmware

# Chiffrer le fichier des secrets (première fois)
ansible-vault encrypt secrets.yaml

# Tester l’inventaire et la connectivité (simulation)
ansible-playbook -i inventory.yaml eva-update-playbook.yaml --vault-id @prompt --check
```

### 2. Mettre à Jour Toutes les EVA

**Avant d’exécuter la mise à jour, placez le fichier firmware dans le dossier `firmware/` à la racine du projet.  
Le nom du fichier peut être par exemple : `fwupd-4.3.38-SNS-amd64-XL-VM.maj`.  
Lors de l’exécution du playbook, il vous sera demandé de renseigner le chemin exact du fichier (ex : `firmware/fwupd-4.3.38-SNS-amd64-XL-VM.maj`).**

```bash
ansible-playbook -i inventory.yaml eva-update-playbook.yaml --vault-id @prompt
```

- Ajouter `--limit eva1` pour cibler une EVA spécifique.
- Utiliser `--extra-vars "delete_snapshot=true"` pour supprimer automatiquement les snapshots après la mise à jour.

### 3. Rétablir une Mise à Jour (Rollback)

- **Via l’interface vCenter** : Restaurer le snapshot de la VM créé avant la mise à jour.
- **Restaurer la configuration** : Copier le fichier de sauvegarde depuis `/backups` vers l’EVA via les outils Stormshield ou l’API.

### 4. Rotation du Mot de Passe Vault

```bash
ansible-vault rekey secrets.yaml
```

### 5. Ajouter/Supprimer une EVA

- Modifier [`inventory.yaml`](inventory.yaml:1) et [`secrets.yaml`](secrets.yaml:1) pour ajouter/supprimer des hôtes et secrets.
- Re-chiffrer les secrets si nécessaire.

### 6. Nettoyage Manuel des Snapshots

```bash
ansible-playbook -i inventory.yaml tasks/snapshot_cleanup.yaml --vault-id @prompt
```

### 7. Mises à Jour Planifiées (Exemple Cron)

```bash
0 3 * * 1 ansible-playbook -i /chemin/vers/inventory.yaml /chemin/vers/eva-update-playbook.yaml --vault-id /chemin/vers/vault_pass.txt >> /var/log/eva-updates.log 2>&1
```

---

## Structure

- `inventory.yaml` : Hôtes et groupes (tous les secrets référencés comme variables)
- `group_vars/all.yaml` : Variables globales (version firmware, dossiers backup/log, etc.)
- `secrets.yaml` : Toutes les données sensibles (à chiffrer avec Ansible Vault)
- `eva-update-playbook.yaml` : Playbook principal pour mise à jour, snapshot, sauvegarde, logs et nettoyage
- `tasks/health_checks.yaml` : Vérifications de santé avant/après mise à jour
- `tasks/snapshot_cleanup.yaml` : Nettoyage optionnel des snapshots (manuel ou planifié)
- `firmware/` : Dossier dédié pour stocker les fichiers de mise à jour firmware (ex : `firmware/fwupd-4.3.38-SNS-amd64-XL-VM.maj`)

---

## Paramètre du fichier firmware

Le chemin du fichier firmware utilisé lors de la mise à jour est demandé lors de l’exécution du playbook :

```yaml
firmware_file: "firmware/fwupd-4.3.38-SNS-amd64-XL-VM.maj"
```

Assurez-vous que le fichier de mise à jour est bien présent dans ce dossier et renseignez le chemin exact lorsqu’il est demandé.

---

## Sécurité

- Tous les secrets sont chiffrés avec Ansible Vault.
- Utiliser `no_log: true` pour les tâches sensibles.
- Changer régulièrement le mot de passe du vault.
- Ne jamais versionner le mot de passe du vault.

---

## Dépannage

- Vérifier les logs dans `/logs` pour les traces détaillées d’exécution.
- S’assurer que toutes les collections Ansible requises sont installées.
- En cas de problème SSH/API, vérifier les secrets et l’accès réseau.

---
## Ansible Stormshield SNS Collection Setup

To use the `stormshield.sns.sns_command` and other modules, ensure the collection and its Python dependency are installed:

```sh
ansible-galaxy collection install stormshield.sns
pip install stormshield.sns.sslclient
```

Refer to the [official documentation](https://github.com/stormshield/ansible-sns-collection) for advanced usage and module details.