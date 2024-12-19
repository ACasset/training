# TP n°01 : copier un fichier

## Introduction

Ce TP a pour but d'introduire les concepts fondateurs d'Ansible : l'inventaire, le playbook et l'idempotence.

Vous trouverez deux fichiers dans ce dossier :
- `inventory.yml` est le fichier d'inventaire. Son but est de recenser les différentes machines (les 'hosts') sur lesquelles on va vouloir effectuer des actions (les 'tasks').
- `playbook.yml` est le fichier qui va décrire les différentes tasks que l'on va jouer sur les machines.

> L'ensemble des fichiers sont écrits en YAML. Il s'agit d'un langage descriptif. La seule règle importante à retenir est que, comme par exemple Python, les différents blocs sont identifiés via leur indentation. Cette indentation est EXCLUSIVEMENT composée d'espaces, jamais de tabulations. Peu importe que vous mettiez 2, 4 ou 8 espaces, tant que la structure du fichier est cohérente. Si vous avez besoin d'un guide, vous pourrez en trouver un [sur le site officiel d'Ansible](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html).

## Prérequis

Pour ce TP, vous allez avoir besoin de deux éléments en plus de ce projet : un contrôleur, et des hôtes distants.

### Contrôleur

Le contrôleur est la machine sur laquelle Ansible va être installé et sur laquelle vont être stockés l'inventaire et le playbook.

Il est infiniment plus facile d'utiliser Ansible à partir d'un système Unix. Si vous utilisez Windows, il vous faudra donc utiliser WSL ou installer une machine virtuelle (locale ou distante). Assurez-vous que le contrôleur est connecté soit à Internet, soit à un dépôt de binaires qui contient Ansible et toutes ses dépendances.

> Il n'y a pas réellement d'exigences de ressources, il faut juste que Python 3 soit installé (ce qui est le cas de la quasi-totalité des distributions Unix).

### Hôtes distants

Pour les hôtes, ce TP nécessite une seule machine virtuelle, tournant elle-aussi sur une distribution Unix, mais les prochains TPs en nécessiteront deux, donc n'hésitez pas à créer les deux dès maintenant.

L'utilisateur qui va être utilisé par Ansible doit avoir les droits `sudo`. Le moyen le plus simple pour y parvenir est d'ajouter l'utilisateur au groupe adéquat. Sur un OS à base RHEL (donc incluant AlmaLinux et CentOS), la commande est `usermod -aG wheel <utilisateur>`. Sur un OS à base Debian (donc incluant Ubuntu), la commande est `usermod -aG sudo <utilisateur>`.

> Les machines virtuelles n'ont pas besoin d'être très puissantes : si on se contente d'un OS en version 'minimale' (comme si c'était un vrai serveur, donc), 1 CPU et 2 Go de RAM devraient suffire.

## Déroulé

### Personnaliser l'inventaire

Le fichier `inventory.yml` est déjà prérempli avec trois hôtes, pour montrer les différentes façons basiques de déclaration.

Aucune n'est strictement meilleure qu'une autre, cela dépend de votre contexte et de votre façon de faire : il est juste recommandé d'éviter de les mélanger, pour une question de cohérence.

Vous allez pouvoir choisir une de ces méthodes de déclaration et la modifier pour qu'elle pointe sur votre machine virtuelle et sur l'utilisateur local de cette machine.

> Bien que rien ne l'empêche, l'utilisation de `root` pour établir une connexion à une machine distance est une très mauvaise pratique en général. Ansible propose des mécanismes si des droits administrateur sont nécessaires, que nous verrons dans les prochains TPs.

Supprimez ensuite les autres déclarations pour ne pas encombrer le fichier, ce qui risquerait de générer des erreurs à l'exécution.

### Compléter le playbook

Ansible utilise des 'modules' pour effectuer les différentes actions. Ces modules sont codés en Python et vont exécuter du code sur l'hôte distant pour remplir leur fonction. Dans leur grande majorité, ils sont 'idempotents' : cela signifie qu'ils vont, avant d'agir, effectuer des vérifications et ne rien faire si le système est déjà dans l'état souhaité. L'idempotence permet ainsi de décrire l'état final à atteindre sur la machine distante, et d'exécuter de façon indiscriminée un playbook sans se soucier de l'état actuel, et de si le playbook a déjà été partiellement ou totalement joué contre cette même machine.

Le nommage de ces modules respecte la nomenclature suivante : `<namespace>.<collection>.<module>`. Le namespace représente généralement la société ou le groupe à l'origine de la collection. Les deux plus courants sont `ansible` et `community`. Les collections sont des regroupements thématiques. Il existe par exemple une collection dédiée à la plateforme Azure de Microsoft, une autre dédiée à Google Cloud Platform, et ainsi de suite. De même, les actions de base sur un système sont gérées par l'équipe d'Ansible elle-même, et sont proposées dans une collection spécifique `ansible.builtin`.

Dans ce TP, le fichier `playbook.yml` est déjà structuré pour utiliser le module `ansible.builtin.copy`. L'exercice va consister à aller lire [la documentation du module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html) pour créer un fichier sur l'hôte distant. Pour une première utilisation, le contenu du fichier est directement renseigné dans le playbook, nous verrons comment faire autrement dans les prochains TPs.

Vous allez donc pouvoir ouvrir ce fichier `playbook.yml`, consulter la documentation et remplacer la partie `parametre: valeur` par de vrais paramètres du module. Soyez attentifs au mot clé `required` dans la documentation, qui indique les paramètres obligatoires sans lesquels le module ne fonctionnera pas. N'hésitez pas à rajouter d'autres paramètres qui ne seraient pas `required` et qui vous semblent importants.

> La documentation contient toujours de multiples exemples très utiles à la compréhension en bas de la page.

### Installer Ansible sur le contrôleur

Pour installer Ansible, on va utiliser le gestionnaire de paquets `pip`, avec la commande suivante :
```bash
python3 -m pip install --user ansible
```

Et on testera la réussite de l'installation avec la commande suivante :
```bash
ansible --version
```

> Il est également possible d'utiliser Ansible avec des conteneurs, qu'on appelle `execution environments`, mais ce n'est pas le sujet de ce TP. Pour plus d'informations, consultez le [site officiel](https://docs.ansible.com/ansible/latest/getting_started_ee/index.html)

### Copier les clés SSH

Ansible utilise des protocoles standards pour se connecter aux machines distantes. Dans notre cas, vu que l'on cible des machines Unix, c'est SSH qui sera utilisé.

Il est possible de se connecter aux VMs avec un nom d'utilisateur et un mot de passe, auquel cas on renseignera la variable `ansible_pass`, mais la méthode privilégiée est d'utiliser des clés SSH.

#### Vérification de l'existence d'une clé

Il est vraisemblable qu'une clé SSH existe déjà sur le contrôleur Ansible, ce qu'on peut vérifier avec la commande suivante :
```bash
# Affichage des clés SSH pour l'utilisateur courant
ls -al ~/.ssh
```

> Si plusieurs clés existent, privilégier la clé `id_ed25519`, qui est plus sécurisée que `id_rsa`

#### (optionnel) Création d'une clé

Si une clé n'existe pas déjà, il va falloir la créer avec la commande suivante :
```bash
# Génération d'une clé Ed25519
ssh-keygen -t ed25519 -C "un commentaire pour identifier la clé, par exemple une adresse email"

# Dans ce cas, le nom de la clé SSH pour l'étape de copie devrait être 'id_ed25519'
```

Si la génération de clé échoue, il est vraisemblable que le système d'exploitation du contrôleur soit trop ancien et ne supporte pas l'algorithme Ed25519, et on va donc se rabattre sur une autre commande :
```bash
# Génération d'une clé RSA
ssh-keygen -t rsa -b 4096 -C "un commentaire pour identifier la clé, par exemple une adresse email"

# Dans ce cas, le nom de la clé SSH pour l'étape de copie devrait être 'id_rsa'
```

#### Copie de la clé

Enfin, on va pouvoir copier la clé SSH sur le serveur distant avec la commande suivante :
```bash
# Copie de la clé sur le serveur distant
ssh-copy-id -i ~/.ssh/LE_NOM_DE_LA_CLÉ_SSH   LE_NOM_D_UTILISATEUR_DU_SERVEUR_DISTANT@LE_NOM_TECHNIQUE_DU_SERVEUR_DISTANT
```

### Lancer l'exécution

Maintenant que tous les éléments sont réunis, on va pouvoir lancer l'exécution du playbook avec la commande suivante :
```bash
ansible-playbook --inventory inventory.yml playbook.yml
```

Ansible va alors effectuer les actions suivantes :
- Connexion à l'hôte distant avec la clé SSH préalablement copiée
- Récupération des informations de l'hôte (ce qu'on verra sous le nom 'Gathering Facts')
- Exécution des différentes tasks du playbook dans l'ordre

À la fin de l'exécution, Ansible affiche un résumé de son exécution : on peut alors savoir d'un coup d'œil le nombre de tasks exécutées/ignorées/échouées par hôte.

Dans notre cas, on a deux possibilités :
- Si l'exécution est réussie, on devrait voir un `changed=1`, indiquant qu'une action a eu lieu. En remontant les logs de l'exécution, on va voir que ce `changed` est associé à la task `Copie d'un fichier`. Et en se connectant à l'hôte distant, on pourra retrouver le fichier à l'endroit indiqué, et vérifier que son contenu est bien 'Hello world'.
- En revanche, si l'exécution a échoué, on va voir un `failed=1`. En remontant les logs de l'exécution, l'erreur va être détaillée. Les trois erreurs les plus fréquentes que vous rencontrerez seront :
  - `dest is required` car vous avez oublié de préciser le paramètre obligatoire `dest` au module (ou bien vous l'avez mal indenté). Rajoutez-le ou vérifiez son indentation.
  - `Unsupported parameters for (copy) module` car vous avez laissé la ligne `parametre: valeur` en place, ou fait une faute de frappe sur le paramètre `dest`. Or, Ansible n'accepte pas les clés inconnues. Supprimez la ligne ou corrigez la faute de frappe.
  - `Permission non accordée` ou `Access denied`, car vous avez indiqué dans le paramètre `dest` un chemin où l'utilisateur `ansible_user` n'a pas les droits d'écriture. Essayez avec le chemin `/tmp/fichier_test`.

> Les messages d'erreur Ansible sont présentés sous différentes formes en fonction de l'origine de l'erreur. Si l'erreur advient sur le contrôleur, elle est générée par Ansible lui-même, et on aura un message assez court et explicite (exemple: `FAILED! => {"changed": false, "msg": "dest is required"}`). Si l'erreur advient sur l'hôte distant, c'est souvent le système d'exploitation qui la remonte, et Ansible la restitue telle quelle, avec parfois beaucoup de contenu et une langue différente.

> Si l'exécution échoue avec une autre erreur, vous pouvez 1° essayer de comprendre et de résoudre l'erreur, si celle-ci est assez explicite (ce n'est pas toujours le cas avec Ansible) ; 2° restaurer les fichiers et recommencer le TP, c'est peut-être une faute de frappe ou d'indentation ; ou 3° contacter un ingénieur DevOps autour de vous pour avoir de l'aide. 

Pour démontrer l'idempotence, on va pouvoir relancer le playbook avec la même commande :
```bash
ansible-playbook --inventory inventory.yml playbook.yml
```

On peut maintenant voir que le résumé de l'exécution n'indique plus de `changed`, mais un `ok=1`, indiquant que le système était déjà dans l'état désiré, et qu'aucune modification n'a donc eu lieu.
