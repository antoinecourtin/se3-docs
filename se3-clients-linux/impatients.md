# Intégration manuelle d'un `client-linux`

Il est possible d'intégrer manuellement un `client-linux` à un domaine géré par un `se3` mais il est plus simple d'utiliser [le mécanisme d'installation/post-intallation/intégration automatique](../pxe-clients-linux/README.md#installation-de-clients-linux-debian-et-ubuntu-via-se3--intégration-automatique) mis à votre disposition par les `se3` actuels.

**NB :** il est actuellement déconseillé de relancer le script d'intégration sur un `client-linux` déjà intégré. En effet, le script n'est pas totalement idempotent (il reste encore à rendre idempotentes 2 à 3 fonctions du script). Cela aurait des effets de bords néfastes au bon fonctionnement des `clients-linux`.

**Ce qui suit n'est là que pour votre culture générale ;-)**

Dans les anciennes versions des `se3`, la mise en place des `clients-linux` commencait par l'installation
du paquet `se3-clients-linux` sur le serveur. Le mieux est de vous reporter à [la doc de mise en place](../pxe-clients-linux/misenplace.md#mise-en-place-du-dispositif-dinstallation-de-clients-gnulinux) qui donne la marche à suivre pour les versions actuelles des `se3`.

* [Installation du paquet `se3-clients-linux` sur le serveur](#installation-du-paquet-se3-clients-linux-sur-le-serveur)
    * [Quelques précisions](#quelques-pr%C3%A9cisions)
    * [En ligne de commande](#en-ligne-de-commande)
    * [Par l'interface `Web` du serveur `se3`](#par-linterface-web-du-serveur-se3)

Ensuite, sur chaque `client-linux`, il était exécuté le script d'intégration correspondant
à sa version (Jessie ou Trusty, par exemple). Cela, et bien d'autres choses, est maintenant réalisé automatiquement : reportez-vous à [la doc d'utilisation du mécanisme `pxe`](../pxe-clients-linux/utilisation.md#utilisation-du-dispositif-dinstallation-de-clients-gnulinux) qui donne, là encore, la marche à suivre pour les versions actuelles des `se3`.

* [Intégration d'un client GNU/Linux](#int%C3%A9gration-dun-client-gnulinux)
    * [Copier le script d'intégration sur le client-linux](#copier-le-script-dint%C3%A9gration-sur-le-client-linux)
    * [Rendre exécutable le script d'intégration](#rendre-ex%C3%A9cutable-le-script-dint%C3%A9gration)
    * [Lancer le script d'intégration](#lancer-le-script-dint%C3%A9gration)


**Recommendation :** Le script d'intégration sera exécuté sur un `client-linux` qui vient d'être installé. Si le client-linux *a déjà servi*, il se peut qu'apparaissent des effets de bords… C'est pour cela que, d'une part, nous vous conseillons d'utiliser [le mécanisme d'intallation/intégration automatique](../pxe-clients-linux/README.md#installation-de-clients-linux-debian-et-ubuntu-via-se3--intégration-automatique), et d'autre part, il vaut mieux intégrer un client-linux ayant un système venant d'être installé.


## Installation du paquet `se3-clients-linux` sur le serveur

Il faut, bien sûr, que votre réseau local dispose d'une connexion
Internet pour préparer votre serveur `se3` en y installant le paquet `se3-clients-linux`.

Pour ce faire, les 2 modes possibles sont soit `la ligne de commande`, soit `l'interface Web du se3`.

Ces 2 modes sont détaillés un peu plus bas, mais il peut être sage de lire auparavant quelques précisions sur les conséquences de l'installation du paquet `se3-clients-linux`.


### Quelques précisions

L'installation ne fait rien de bien méchant sur votre
serveur. Vous pouvez parfaitement désinstaller le paquet du
serveur afin que celui-ci retrouve très exactement le même
état qu'avant l'installation (voir la page concernant
[la désinstallation](desinstall_reinstall_paquet.md#désinstallationréinstallation-du-paquet-se3-clients-linux)).

L'installation se borne uniquement à effectuer les tâches
suivantes :

* Création d'un nouveau répertoire : le répertoire
`/home/netlogon/clients-linux/`.

* Création d'un partage Samba supplémentaire sur le serveur à
travers le fichier de configuration `/etc/samba/smb_CIFSFS.conf` : il
s'agit du partage CIFS nommé  netlogon-linux correspondant au
répertoire `/home/netlogon/clients-linux/` du serveur.

* Lecture de certains paramètres du serveur afin d'adapter certains
scripts contenus dans le paquet se3-clients-linux à l'environnement
de votre domaine local.

En fait, ces fameux paramètres récupérés lors
de l'installation du paquet sont au nombre de trois :

    1. l'adresse IP du serveur ;
    2. le suffixe de base de l'annuaire LDAP ;
    3. l'adresse du serveur de temps NTP.


### En ligne de commande

Il faut ouvrir une console en tant que `root` et lancer :
```sh
apt-get update
apt-get install se3-clients-linux
```
Ce mode de mise en place est valable quel que soit la version de votre se3.
Cependant, si vous êtes en `Lenny`, nous vous conseillons fortement de passer en `Squeeze`.


### Par l'interface `Web` du serveur `se3`

Vous pouvez aussi faire l'installation du module `se3-clients-linux`
en passant par `l'interface d'administration Web` du serveur
via les menus `Configuration générale` puis `Modules`.

Dans le tableau des modules, le paquet `se3-clients-linux`
correspond à la ligne avec l'intitulé `Support des clients linux`.


## Intégration d'un client `GNU/Linux`

Le répertoire `/home/netlogon/clients-linux/` de votre
serveur contient un script d'intégration par type de
distribution GNU/Linux.

Par exemple, le script d'intégration pour des Debian `Jessie`
se trouve dans le répertoire `/home/netlogon/clients-linux/distribs/jessie/integration/`
et il s'appelle `integration_jessie.bash`.

Avant d'exécuter l'un de ces scripts, en tant que `root`,
**en local** sur le client GNU/Linux que vous souhaitez intégrer,
il faudra le copier depuis le seveur se3.

**Remarque :** pour copier en local sur un client GNU/Linux
le script d'intégration qui se trouve sur le serveur se3,
on pourrait utiliser la bonne vieille clé USB des familles,
mais il est plus pratique d'user et d'abuser de la commande `scp`
qui permet d'effectuer très simplement des copies
entre deux machines (sous GNU/Linux) distantes.


### Copier le script d'intégration sur le `client-linux`

Sur le terminal d'un client `Debian Jessie`,
exécutez les commandes suivantes :

```sh
# Chemin du fichier sur le serveur. Le joker * nous permet simplement 
# d'économiser la saisie de quelques touches sur le clavier (à
# condition d'en saisir suffisamment pour éviter toute ambiguïté 
# sur le nom du fichier).
SOURCE="/home/netlogon/clients-linux/dist*/jes*/int*/int*"

# Répertoire de destination sur le client GNU/Linux en local. Par
# exemple le bureau, histoire de voir apparaître le fichier
# sous nos yeux.
DESTINATION="/home/toto/Bureau/"

# Et enfin la copie du fichier du serveur vers le client GNU/Linux en local.
# Il faudra alors saisir le mot de passe du compte `root` du serveur.
scp root@IP-SERVEUR:"$SOURCE" "$DESTINATION"
```

**Remarque :** si jamais vous avez un doute sur le type de
distribution de votre client GNU/Linux, vous pouvez lancer
dans un terminal la commande suivante (pas forcément en tant
que `root`) : `lsb_release --codename`.

Le résultat vous affichera le nom de code de la distribution
(jessie ou trusty par exemple) ce qui vous indiquera le script
d'intégration à utiliser.


### Rendre exécutable le script d'intégration

Supposons par exemple que vous avez copié le script
d'intégration sur une `Debian Jessie` et que celui-ci se
trouve sur votre bureau. Il faut d'abord le rendre exécutable.

**en tant que root**, utilisez les commandes suivantes :
```sh
# D'abord, on se place sur le bureau (ici, il s'agit du bureau de toto).
cd /home/toto/Bureau

# Ensuite, on rend le script exécutable.
chmod u+x integration_jessie.bash
```

**Remarque :** On peut enchaîner les différentes commandes
de récupération du script d'intégration et de préparation
de la façon suivante :
```sh
SOURCE="/home/netlogon/clients-linux/dist*/jes*/int*/int*" && DESTINATION="/home/toto/Bureau" && scp root@IP-SERVEUR:"$SOURCE" "$DESTINATION" && cd /home/toto/Bureau && chmod u+x integration_jessie.bash
```


### Lancer le script d'intégration

Une fois les opérations de copie et de préparation effectuées,
vous pouvez lancer, toujours **en tant que root**, l'intégration ainsi :

```sh
# Enfin, on lance l'intégration.
./integration_jessie.bash --nom-client="toto-04" --is --ivl --rc
```

Les explications sur les options se trouvent plus loin dans
le document (à [cette page](options_scripts.md#les-options-des-scripts-dintégration)).

Si tout se passe bien, le client finira par lancer un
redémarrage. Une fois celui-ci terminé, vous devriez être en
mesure d'ouvrir une session avec un compte du domaine (comme
le compte `admin` ou un compte de `type professeur` ou de `type élève`).

**Important :** Il est préférable qu'aucun compte local du
client n'ait le même identifiant qu'un compte du domaine.

Or,lorsqu'on installe un client GNU/Linux, on est en général
amené à créer au moins un compte local (en plus du compte
`root`).

Si cela vous arrive, arrangez-vous pour que l'identifiant
de ce compte local ne risque pas de rentrer en conflit avec
l'identifiant d'un compte du domaine. Vous pouvez utiliser
`userlocal` comme identifiant de ce compte local par exemple, ou autre chose…

