Setup Containerd
================

Installe, met à jour et contrôle containerd (paquet `containerd.io` du dépôt apt Docker) sur des nœuds Kubernetes Debian/Ubuntu. Un seul chemin pour trois situations :

- **nœud neuf** : dépôt Docker, paquet en version exacte, `config.toml` depuis le template (format version 3, `SystemdCgroup = true`, runsc optionnel) ;
- **nœud à jour** : rien ne bouge, gel du paquet et contrôles seuls ;
- **nœud en retard** : upgrade roulant — sauvegarde de `config.toml`, drain, paquet, redémarrage de containerd puis de kubelet, contrôles, uncordon.

Le `config.toml` existant n'est jamais écrasé (sauf fichier absent, config par défaut du paquet avec le plugin CRI désactivé, ou `containerd_manage_config: true`) : containerd 2.x convertit en mémoire les configs version 2 à chaque démarrage.

Prérequis
---------

- Debian ou Ubuntu, sudo.
- Pour le drain et les contrôles Ready : le premier hôte de `containerd_control_plane_group` porte `/etc/kubernetes/admin.conf`.
- Le playbook appelant passe les nœuds en `serial: 1`.

Viser une nouvelle version
--------------------------

Changer `containerd_version` (format `X.Y.Z-R` ou `X.Y.Z`, sans suffixe de distribution) dans les group_vars, puis relancer le playbook. Le rôle :

1. résout la version dans le dépôt de chaque nœud (`~debian.13~trixie`, `~ubuntu.24.04~noble`...) et refuse, en listant les versions disponibles, si elle n'existe pas — avant tout drain ;
2. refuse un downgrade sauf `containerd_allow_downgrade: true` ;
3. ne draine que les nœuds dont la version change.

En cas d'échec après le drain, le nœud reste cordonné : on ne renvoie pas de pods sur un runtime dans un état inconnu. La sauvegarde de `config.toml` est dans `/var/backups/containerd/`.

Exemple
-------

```yaml
- hosts: k8s_workers
  serial: 1
  become: true
  roles:
    - role: setup_containerd
      vars:
        containerd_version: "2.4.2-3"
        containerd_control_plane_group: k8s_control_plane
        containerd_runsc: true
```

Les autres variables sont documentées dans `defaults/main.yml`.

Author Information
------------------

Jeremie Payet (jpcweb)
