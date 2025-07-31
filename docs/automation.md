# Intégration Automatisation & Orchestration

## Ansible Tower / AWX

- Importez ce projet comme modèle de tâche.
- Stockez le mot de passe du vault de manière sécurisée dans les identifiants Tower.
- Planifiez des exécutions récurrentes (ex. chaque nuit ou chaque semaine).
- Utilisez les fonctionnalités de journalisation et de notification de Tower pour l’audit et l’alerte.

## Exemple de Tâche Cron

Ajoutez à la crontab (exécuter en tant qu’utilisateur dédié à l’automatisation) :

```bash
0 3 * * 1 ansible-playbook -i /chemin/vers/inventory.yaml /chemin/vers/eva-update-playbook.yaml --vault-id /chemin/vers/vault_pass.txt >> /var/log/eva-updates.log 2>&1
```

- Stockez le mot de passe du vault dans un fichier avec des permissions restreintes (`chmod 600`).
- Surveillez `/var/log/eva-updates.log` pour les résultats.

## Recommandations de Sécurité

- Ne jamais exposer les mots de passe du vault dans des scripts ou des logs.
- Utiliser un compte d’automatisation dédié avec le minimum de privilèges.
- Faire tourner régulièrement les identifiants et mots de passe du vault.
- Intégrer la supervision (ELK, Prometheus) pour l’alerte en cas d’échec.

---