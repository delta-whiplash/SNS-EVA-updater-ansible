# Guide complet : Campagne de mise à jour Stormshield EVA via Ansible

## 1. Prérequis

- Cloner le dépôt Git.
- Installer les collections Ansible requises :
  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```
  ou
  ```bash
  ansible-galaxy collection install stormshield.sns community.vmware
  ```

## 2. Préparation des fichiers de configuration

### a. [`inventory.yaml`](../inventory.yaml)
- Définir tous les hôtes Stormshield EVA à mettre à jour.
- Adapter les groupes selon l’infrastructure cible.

### b. [`group_vars/all.yaml`](../group_vars/all.yaml)
Ajuster les variables selon la campagne :
- `firmware_version` : version cible à déployer (ex : "5.0.1")
- `backup_folder` : dossier de sauvegarde des configurations
- `log_folder` : dossier pour les logs d’exécution
- `delete_snapshot` : `true` pour supprimer automatiquement le snapshot après succès, `false` sinon
- `sns_validate_certs` : `false` si certificats auto-signés (recommandé en interne), `true` pour validation stricte
- `sns_timeout` : délai (en secondes) pour les opérations Stormshield standards
- `sns_firmware_timeout` : délai (en secondes) pour la mise à jour firmware (plus long)
- `vmware_validate_certs` : `false` si vCenter utilise un certificat auto-signé, `true` sinon
- `vmware_timeout` : délai (en secondes) pour les opérations VMware

### c. [`secrets.yaml`](../secrets.yaml)
- Contient tous les identifiants sensibles (appliances, vCenter, etc.).
- Toujours chiffré avec Ansible Vault :
  ```bash
  ansible-vault edit secrets.yaml
  ```

### d. Firmware
- Placer le fichier firmware dans le dossier `firmware/` et renseigner le chemin lors de l’exécution.

## 3. Procédure de campagne

### a. Vérification de la configuration
- Relire et ajuster tous les fichiers ci-dessus.
- S’assurer que les chemins de sauvegarde/logs existent et sont accessibles.

### b. Test à blanc (dry-run)
```bash
ansible-playbook -i inventory.yaml eva-update-playbook.yaml --vault-id @prompt --check
```
- Permet de valider la connectivité, la syntaxe et la configuration sans rien modifier.

### c. Lancement de la campagne réelle
```bash
ansible-playbook -i inventory.yaml eva-update-playbook.yaml --vault-id @prompt
```
- Suivre la sortie console pour détecter toute erreur.

### d. Suivi post-campagne
- Vérifier les logs générés dans le dossier défini.
- Contrôler les sauvegardes dans le dossier backup.
- Vérifier l’état des appliances et la version du firmware.

## 4. Cas particuliers et options

- **Timeouts** : Adapter les valeurs si le réseau est lent ou si les appliances sont nombreuses.
- **Certificats** : Si tu passes en production avec des certificats signés, mettre `*_validate_certs: true`.
- **Suppression snapshot** : Mettre `delete_snapshot: true` pour automatiser le nettoyage, sinon gérer manuellement.
- **Ajout d’appliances** : Modifier `inventory.yaml` et `secrets.yaml` puis relancer la campagne.

## 5. Sécurité et bonnes pratiques

- Ne jamais versionner de secrets non chiffrés.
- Toujours tester la configuration avant exécution réelle.
- Garder les collections Ansible à jour.
- Documenter toute modification majeure dans le dépôt.

---