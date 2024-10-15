# TP2 UNIX 
*Luce Giovannetti*
## 1.1: Connection ssh root (reprise fin tp-01) 
On avait modifie permitrootlogin et passwordautentication pour pouvoir se connecter depuis notre machine à la MV.
Extrait du man:
```bash
     PermitRootLogin
             Specifies whether root can log in using ssh(1).  The argument must be yes, prohibit-password,
             forced-commands-only, or no.  The default is prohibit-password.

             If this option is set to prohibit-password (or its deprecated alias, without-password), password and
             keyboard-interactive authentication are disabled for root.

             If this option is set to forced-commands-only, root login with public key authentication will be al‐
             lowed, but only if the command option has been specified (which may be useful for taking remote backups
             even if root login is normally not allowed).  All other authentication methods are disabled for root.

             If this option is set to no, root is not allowed to log in.

         PasswordAuthentication
             Specifies whether password authentication is allowed.  The default is yes

  ```

### `PermitRootLogin`
Le paramètre `PermitRootLogin` dans le fichier de configuration SSH contrôle si l'utilisateur root est autorisé à se connecter via SSH et, si oui, par quelles méthodes. Voici les différentes options :

`PermitRootLogin yes`
  
- **Description** : Autorise toutes les méthodes d'authentification pour l'utilisateur root, y compris le mot de passe.
- **Avantages** : Permet un accès root direct via SSH.
- **Inconvénients** : C’est une option risquée, car elle expose le compte root à des attaques par force brute sur le mot de passe, surtout si le mot de passe est faible.
- **Cas d'utilisation** : À éviter dans la plupart des cas pour des raisons de sécurité, mais peut être utile temporairement lors de configurations initiales ou de débogage en cas de perte d'accès.

`PermitRootLogin prohibit-password`

- **Description** : Autorise uniquement l'authentification par clé publique pour root ; les méthodes d'authentification par mot de passe et clavier interactif sont désactivées.
- **Avantages** : Améliore la sécurité en autorisant seulement l'accès par clé publique pour root, empêchant ainsi les attaques par force brute via le mot de passe.
- **Inconvénients** : Si les clés SSH sont compromises, le compte root reste vulnérable, et cela peut nécessiter une configuration et une gestion des clés plus rigoureuses.
- **Cas d'utilisation** : Recommandé pour les environnements de production où l'accès root direct est nécessaire. 

`PermitRootLogin forced-commands-only`

- **Description** : Permet l'accès root par clé publique uniquement si une commande spécifique est définie pour l'authentification. Toutes les autres méthodes d'authentification pour root sont désactivées.
- **Avantages** : Utile pour des cas spécifiques comme les sauvegardes automatisées, car l'accès root est limité à des commandes pré-autorisées.
- **Inconvénients** : Ne permet pas d'accès interactif pour root. Cela nécessite une configuration rigoureuse.
- **Cas d'utilisation** : Utilisé pour des automatisations spécifiques où un accès root est nécessaire, mais un accès complet n’est pas requis, par exemple pour des scripts de sauvegarde.

`PermitRootLogin no`

- **Description** : Désactive complètement l'accès root via SSH, quel que soit le type d'authentification.
- **Avantages** : Le plus sécurisé, car il empêche tout accès SSH direct au compte root.
- **Inconvénients** : Les utilisateurs devront utiliser `sudo` pour les tâches nécessitant root.
- **Cas d'utilisation** : Fortement recommandé pour les environnements de production. Les utilisateurs se connectent avec des comptes normaux et utilisent `sudo` pour des actions root, ce qui ajoute une couche de sécurité.

### `PasswordAuthentication`

Le paramètre `PasswordAuthentication` détermine si l'authentification par mot de passe est autorisée pour tous les utilisateurs, pas seulement pour root. Voici les options disponibles :

`PasswordAuthentication yes`
- **Description** : Autorise tous les utilisateurs à se connecter via un mot de passe.
- **Avantages** : Simple pour les utilisateurs car ils peuvent se connecter sans nécessiter de clé SSH.
- **Inconvénients** : Expose le serveur aux attaques par force brute de mot de passe, surtout si les mots de passe sont faibles ou s'il n'existe pas de protection.
- **Cas d'utilisation** : Peut être utile dans un environnement de test ou pour des utilisateurs ayant des restrictions temporaires sur la clé publique, mais à éviter dans la plupart des cas.

`PasswordAuthentication no`
- **Description** : Désactive l'authentification par mot de passe pour tous les utilisateurs. L'authentification se fait uniquement par clé publique.
- **Avantages** : Réduit considérablement le risque d'attaques par force brute de mot de passe, car aucun mot de passe ne peut être utilisé pour accéder au serveur.
- **Inconvénients** : Exige que tous les utilisateurs disposent de clés SSH pour accéder au serveur, ce qui peut nécessiter une configuration supplémentaire.
- **Cas d'utilisation** : Recommandé pour la plupart des environnements de production. L’authentification par clé est plus sécurisée et protège contre les attaques par force brute de mot de passe.

## 1.2: Authentification par clef / Generation de clefs 
On commence par generer un couple de clef prive public sur la machine hote, avec la commande:

```bash
C:\Users\luceg>ssh-keygen -t rsa -b 2048
  ```
avec ça on obtient comme resultat: 
```bash
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\luceg/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\luceg/.ssh/id_rsa.
Your public key has been saved in C:\Users\luceg/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:3l6vegQZqSCLbKoTj9ECs1iVeyM2CzNJ7sGBFkoMW9E luceg@LAPTOP-343ASROG
The key's randomart image is:
+---[RSA 2048]----+
|+++o .     .     |
|+=o E .   o      |
|+= = + . . o     |
|o & * o . o      |
|oO * = .S  .     |
|B o .  . .  .    |
|.*      . ...    |
|+ .      . ...   |
| .        oo...  |
+----[SHA256]-----+
  ```

#### Pourquoi est-ce une mauvaise idée de ne pas utiliser de passphrase dans un cas réel ?
La passphrase sert à protéger la clé privée, c'est pas bien de ne pas l'utiliser pour plusieurs raisons:

- **Sécurité réduite** : Si quelqu'un accède à votre machine et trouve votre clé privée, il peut l'utiliser immédiatement pour accéder à tous les serveurs auxquels votre clé donne accès, sans aucune protection supplémentaire.
- **Accès non autorisé** : Une passphrase ajoute une couche de protection. Sans passphrase, si votre clé privée est volée, elle peut être utilisée immédiatement par une autre personne pour se connecter à vos serveurs.
- **Perte de contrôle** : La passphrase fonctionne comme un second facteur de sécurité. Même si un attaquant obtient la clé, il a besoin de la passphrase pour s’en servir, ce qui augmente les chances de garder les connexions SSH sécurisées.

#### C'est quoi la difference entre la clé publique et la clé privée ?
La clé publique et la clé privée fonctionnent ensemble pour sécuriser les connexions. Voici leurs différences principales :

**1. Clé publique**
- **Utilisation** : La clé publique est **placée sur les serveurs** auxquels on veut accéder. C’est elle qui autorise la connexion de la clé privée correspondante. Elle est destinée à être **partagée** et copiée sur les serveurs. C’est grâce à cette clé publique que le serveur peut vérifier votre identité lorsque vous vous connectez avec la clé privée.
- **Sécurité** : La clé publique ne contient **aucune information sensible** à elle seule. Même si quelqu’un l’obtient, il ne pourra pas s'en servir pour accéder à votre compte sans la clé privée.

**2. Clé privée**
- **Utilisation** : La clé privée reste **sur votre machine locale** et est utilisée pour **authentifier votre identité** auprès du serveur. Elle fonctionne comme une "signature" prouvant que vous êtes bien l’utilisateur autorisé. La clé privée est **strictement personnelle** et **ne doit jamais être partagée**. Elle est gardée en sécurité sur votre appareil local, souvent protégée par une passphrase.
- **Sécurité** : La clé privée contient des informations sensibles. Si quelqu’un y accède, il pourrait se connecter aux serveurs où la clé publique est installée. C'est pourquoi il est recommandé de la protéger avec une passphrase.

Donc la clé publique **va sur le serveur** pour autoriser l’accès, et la clé privée **reste sur l'appareil** pour prouver votre identité. La clé publique peut être partagée sans risque, tandis que la clé privée doit être gardée secrète pour assurer la sécurité des connexions.

## 1.3:  Authentification par clef / Connection serveur (copier la clef)
J'ai voulu utiliser la commande ssh-copy-id, mais celle-ci n'étant pas disponible sur mon système, j'ai changé de méthode et suivi les étapes suivantes :
#### 1. Verification fichier clef :
   - J'ai vérifié la présence de mes clés (`id_rsa` et `id_rsa.pub`) en exécutant la commande `ls`. Cela a affiché les fichiers existants dans le dossier :
     ```powershell
     PS C:\Users\luceg> cd .ssh
     PS C:\Users\luceg\.ssh> ls
     ```
   - Résultat :
```bash
    Directory: C:\Users\luceg\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        09/10/2024     15:15           1831 id_rsa
-a----        09/10/2024     15:15            404 id_rsa.pub
-a----        09/10/2024     14:42            535 known_hosts

 ```

#### 2. Copie de la clé publique sur le serveur
1. J'ai utilisé la commande `scp` pour copier le fichier de clé publique `id_rsa.pub` vers le répertoire `/root` du serveur distant :
   
   ```powershell
   PS C:\Users\luceg\.ssh> scp .\id_rsa.pub root@192.168.184.226:/root/
   ```


#### 3. Connexion initiale au serveur
1. J'ai ensuite initié une connexion SSH au serveur pour finaliser la configuration :
   ```powershell
   PS C:\Users\luceg\.ssh> ssh root@192.168.184.226
   ```

#### 4. Configuration de la clé publique pour l'authentification SSH
1. Une fois connecté au serveur, j'ai accédé au dossier `.ssh` de l'utilisateur root. Si ce dossier n'existait pas, j'aurais dû le créer avec `mkdir .ssh`.
   ```bash
   root@serveur1:~# cd .ssh/
   ```
2. Dans le dossier `.ssh`, j'ai modifié le fichier `authorized_keys` pour inclure ma clé publique :
   ```bash
   root@serveur1:~/.ssh# cat ../id_rsa.pub >> authorized_keys
   ```
   - **Signification** : Cette commande ajoute le contenu de ma clé publique `id_rsa.pub` au fichier `authorized_keys`. Ce fichier permet au serveur de reconnaître les clés autorisées pour la connexion SSH.

3. J'ai ensuite vérifié le contenu de `authorized_keys` pour m'assurer que la clé a bien été ajoutée :
   ```bash
   root@serveur1:~/.ssh# cat authorized_keys
   ```
   - Résultat : La clé publique s'y trouvait bien, confirmant la configuration correcte.

4. J'ai restreint les permissions de `authorized_keys` pour des raisons de sécurité en utilisant la commande suivante :
   ```bash
   root@serveur1:~/.ssh# chmod 600 authorized_keys
   ```
   - **Signification** : Limiter les permissions à `600` signifie que seul le propriétaire a les droits de lecture et d’écriture sur ce fichier, conformément aux exigences de sécurité d’OpenSSH.

#### 5. Test de connexion SSH avec la clé publique
1. Après avoir déconnecté la session, j'ai tenté de me reconnecter via SSH pour tester la connexion sans mot de passe :
   ```powershell
   PS C:\Users\luceg> ssh root@192.168.184.226
   ```
   - **Résultat** : La connexion a été établie avec succès sans demande de mot de passe, confirmant que l’authentification par clé publique fonctionne correctement.


## 1.4: Acceder avec la clef depuis machine hote

### Connexion initiale via SSH
1. **Commande** : 
   ```bash
   ssh root@192.168.184.226
   ```
   - **Explication** : Dans cette connexion, la clé privée par défaut située dans `C:\Users\luceg\.ssh\id_rsa` est utilisée automatiquement pour l’authentification. Le client SSH recherche la clé par défaut dans ce répertoire sans qu'il soit nécessaire de la spécifier avec l'option `-i`, mais ce n'est pas ce qu'on veut.

### Changement de nom des fichiers de clé
2. **Commandes** :
   ```bash
   PS C:\Users\luceg\.ssh> mv .\id_rsa maclef
   PS C:\Users\luceg\.ssh> mv .\id_rsa.pub .\maclef.pub
   ```
   - **Signification** : Ces commandes `mv` renommant les fichiers de clés privés et publics (`id_rsa` et `id_rsa.pub`) en `maclef` et `maclef.pub` pour pouvoir indiquer notre cle lors de la connexion.

### Connexion SSH avec la clé personnalisée
3. **Commande** :
   ```bash
   ssh -i .\maclef root@192.168.184.226
   ```
   - **Signification** : Cette commande spécifie la clé privée `maclef` pour établir la connexion SSH avec `root@192.168.184.226`. Le paramètre `-i` indique au client SSH de rechercher spécifiquement ce fichier de clé privé pour l’authentification.
   - **Résultat** : La connexion au serveur est établie avec succès en utilisant la clé `maclef`, confirmant que la clé privée a été correctement utilisée pour l’authentification.

### Vérification des fichiers dans le dossier `.ssh`
4. **Commande** :
   ```bash
   ls
   ```
   - **Signification** : Cette commande liste les fichiers présents dans le dossier `.ssh`. Elle montre la présence de `maclef`, `maclef.pub`, et `known_hosts` dans le répertoire `.ssh`, confirmant que les fichiers ont bien été renommés.

## 1.5: Securisez
Pour sécuriser l'accès à ma machine via SSH pour l'utilisateur `root` en n'autorisant que l'authentification par clé, j'ai suivi les étapes suivantes :

1. **Modification de la configuration SSH** :
   ```bash
   root@serveur1:~# nano /etc/ssh/sshd_config
   ```
   - J'ai ouvert le fichier de configuration SSH.

2. **Changements effectués dans le fichier** :
   - J'ai modifié la ligne `PermitRootLogin` pour qu'elle soit réglée sur `prohibit-password` :
     - **Signification** : Cela signifie que j'ai permis la connexion en tant que `root` uniquement via des clés SSH et interdit les connexions par mot de passe. Cela rend l'accès root plus sécurisé, car les tentatives de connexion par mot de passe sont bloquées.
   - J'ai aussi réglé `PasswordAuthentication` sur `no` :
     - **Signification** : Cela désactive l'authentification par mot de passe pour tous les utilisateurs, ce qui signifie que seules les connexions SSH par clé sont acceptées. Cela empêche les attaques par force brute, où un attaquant essaie plusieurs mots de passe jusqu'à trouver un valide.

3. **Redémarrage du service SSH** :
   ```bash
   root@serveur1:~# systemctl restart sshd
   ```
   - J'ai redémarré le service SSH pour appliquer les modifications que j'avais faites dans le fichier de configuration.

4. **Test de la connexion** :
   ```bash
   PS C:\Users\luceg\.ssh> ssh root@192.168.184.226
   ```
   - J'ai reçu le message suivant :
     ```
     root@192.168.184.226: Permission denied (publickey).
     ```
   - **Explication** : Ce message signifie que ma tentative de connexion sans clef à echoue.

### Attaques par Force Brute SSH
Les attaques par force brute SSH consistent à essayer systématiquement de nombreux mots de passe pour découvrir celui qui permettrait d'accéder à un compte. Les attaquants utilisent souvent des outils automatisés pour générer et tester des combinaisons de mots de passe jusqu'à ce qu'ils trouvent le bon.

## 2.1 : Étude des processus UNIX

- Pour afficher **la liste des processus**, j'ai utilisé la commande suivante :

```bash
ps axu
```

J'ai obtenu comme reponse:

```bash
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.6 102168 12164 ?        Ss   12:54   0:00 /sbin/init
root           2  0.0  0.0      0     0 ?        S    12:54   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   12:54   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   12:54   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I<   12:54   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I<   12:54   0:00 [netns]
root           9  0.0  0.0      0     0 ?        I    12:54   0:00 [kworker/u2:0-ext4-rsv-conversion]
root          10  0.0  0.0      0     0 ?        I<   12:54   0:00 [mm_percpu_wq]
root          11  0.0  0.0      0     0 ?        I    12:54   0:00 [rcu_tasks_kthread]
.......
root         643  0.0  0.0      0     0 ?        I    13:26   0:00 [kworker/u2:2-events_unbound]
root         664  0.0  0.0      0     0 ?        I    13:31   0:00 [kworker/0:0-ata_sff]
root         667  0.0  0.0      0     0 ?        I    13:33   0:00 [kworker/u2:1-events_unbound]
root         668  0.0  0.0      0     0 ?        I    13:33   0:00 [kworker/u2:3]
root         684  0.0  0.0      0     0 ?        I    13:36   0:00 [kworker/0:2-ata_sff]
root         686  0.0  0.2  11040  4352 pts/0    R+   13:36   0:00 ps axu
```

- À l'aide du **manuel** (`man ps`), voici la signification des colonnes de notre réponse :

USER : Nom de l’utilisateur propriétaire du processus
PID : Numéro d’identification du processus
%CPU : Pourcentage d’utilisation CPU
%MEM : Pourcentage d’utilisation mémoire
STAT : État du processus
START : Date et heure de début du processus
TIME : Temps CPU utilisé par le processus
COMMAND : Commande utilisée pour lancer le processus

- A quoi correspond l’information **TIME** ?
TIME indique le temps total CPU utilisé par le processus depuis son lancement, exprimé en minutes et secondes. Cela reflète le temps pendant lequel le processus a été actif sur le CPU.

- Quel est le **processus** ayant le plus utilise le **processeur** sur votre machine ?
Pour trouver le processus utilisant le plus de CPU, j'ai utilisé la commande `top`

Le résultat nous indique que le processus ayant le plus utilisé le processeur est le 816.

```bash
    816 root      20   0   11552   5048   3168 R   0.3   0.3   0:00.10 top
```

- Quel a été le **premier processus** lancé après le démarrage du système ?
Pour connaître le premier processus lancé, j'ai utilisé `ps -p 1`, et obtenu le resultat:

    PID TTY          TIME CMD
      1 ?        00:00:00 systemd

  Le premier processus lance est systemd.

- À quelle heure votre machine a-t-elle **démarré** ?
Pour afficher l'heure de démarrage de la machine, j'ai utilisé `ps -o lstart -p 1`:

Resultat:
```bash
                 STARTED
Fri Oct 11 12:54:25 2024
```
Cela nous indique que la machine a démarré le 11 octobre à 12h54.

- Trouvez une autre commande permetant de trouver le **temps** depuis lequel votre serveur tourne
Pour obtenir le temps écoulé depuis le démarrage, j'ai utilisé `ps -p 1 -o etime`:
Resultat:
```bash
    ELAPSED
   01:47:38
```
Cela nous indique que le serveur tourne depuis 1 heure, 47 minutes et 38 secondes.

- Pouvez-vous établir le nombre approximatif de **processus créés** depuis le démarrage (“boot”) de votre machine ?
Pour compter le nombre de processus actifs, j'ai utilisé `ps -ef | wc -l`:

Le résultat indique que les processus créés depuis le démarrage sont environ 79.


#### 2 : Afficher le PPID des processus
Pour afficher la liste des processus en cours d'exécution avec leurs **PPID** (Parent Process ID), j'ai utilisé la commande suivante :

`ps -ef`

Explication :
- `ps` c'est l'acronyme de "process status". Cette commande permet de visualiser les processus en cours sur le système.

-`e` indique à ps de lister tous les processus du système, pas seulement ceux de l'utilisateur qui exécute la commande.

-`f` signifie "full format". Elle demande à ps d'afficher des informations détaillées sur chaque processus, y compris le PID (Process ID), le PPID (Parent Process ID), l'utilisateur (UID) qui a lancé le processus, le temps d'exécution, et la commande qui a démarré le processus.


Voici un extrait des résultats obtenus :

```bash 
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 Oct13 ?        00:00:00 /sbin/init
root           2       0  0 Oct13 ?        00:00:00 [kthreadd]
root           3       2  0 Oct13 ?        00:00:00 [rcu_gp]
root           4       2  0 Oct13 ?        00:00:00 [rcu_par_gp]
root           5       2  0 Oct13 ?        00:00:00 [slub_flushwq]
root           6       2  0 Oct13 ?        00:00:00 [netns]
root          10       2  0 Oct13 ?        00:00:00 [mm_percpu_wq]
....
root        1837    1831  0 20:22 pts/0    00:00:00 -bash
root        1840    1837  0 20:23 pts/0    00:00:00 ps -ef
```

#### 3 : Utilisation de la commande pstree 
Pour afficher la liste ordonnée de tous les **processus ancêtres** de la commande ps en cours d'exécution, nous allons utiliser la commande `pstree`.

Voici les **étapes** que j'ai suivi :
1. **Mettre à jour les paquets :**
`apt update `

2. **Rechercher le paquet pstree :**
`apt search pstree`

3. **Installer psmisc :**
`apt install psmisc`

4. **Je peux maintenant utiliser la commande pstree, qui donne comme resultat l'arbre des processus en cours d'exécution :**

```bash
systemd─┬─cron
        ├─dbus-daemon
        ├─dhclient
        ├─login───bash
        ├─sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        └─wpa_supplicant
```
- Chaque processus est indenté pour représenter les relations parent-enfant. Par exemple, le processus login est un enfant de systemd, et bash est un enfant de login.
  
#### 4 : commandes top et htop
La commande `top` (tableau des processus) affiche une vue en temps réel des processus en cours d'exécution sous Linux et montre les tâches gérées par le noyau. Elle fournit également un résumé des informations système qui indique l'utilisation des ressources, y compris l'utilisation du CPU et de la mémoire.

Voici un extrait du resultat obtenu :
```bash
top - 20:30:52 up 1 day,  7:44,  2 users,  load average: 0.00, 0.00, 0.00
Tasks:  75 total,   1 running,  74 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 98.3 id,  1.4 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1967.3 total,   1628.6 free,    222.8 used,    258.1 buff/cache
MiB Swap:   6173.0 total,   6173.0 free,      0.0 used.   1744.4 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  102168  12200   9204 S   0.0   0.6   0:00.93 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 slub_flushwq
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns
     10 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_percpu_wq
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_kthread
     12 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_rude_kthread
     13 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_trace_kthread
     14 root      20   0       0      0      0 S   0.0   0.0   0:00.10 ksoftirqd/0
     15 root      20   0       0      0      0 I   0.0   0.0   0:00.41 rcu_preempt
     16 root      rt   0       0      0      0 S   0.0   0.0   0:00.17 migration/0
     ...............
     48 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mld
     49 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 ipv6_addrconf
     54 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kstrp
```

- La touche `?` permet d'afficher un resume de l'aide :

```bash
Help for Interactive Commands - procps-ng 4.0.2
Window 1:Def: Cumulative mode Off.  System: Delay 3.0 secs; Secure mode Off.

  Z,B,E,e   Global: 'Z' colors; 'B' bold; 'E'/'e' summary/task memory scale
  l,t,m,I   Toggle: 'l' load avg; 't' task/cpu; 'm' memory; 'I' Irix mode
  0,1,2,3,4 Toggle: '0' zeros; '1/2/3' cpu/numa views; '4' cpus two abreast
  f,X       Fields: 'f' add/remove/order/sort; 'X' increase fixed-width fields

  L,&,<,> . Locate: 'L'/'&' find/again; Move sort column: '<'/'>' left/right
  R,H,J,C . Toggle: 'R' Sort; 'H' Threads; 'J' Num justify; 'C' Coordinates
  c,i,S,j . Toggle: 'c' Cmd name/line; 'i' Idle; 'S' Time; 'j' Str justify
  x,y     . Toggle highlights: 'x' sort field; 'y' running tasks
  z,b     . Toggle: 'z' color/mono; 'b' bold/reverse (only if 'x' or 'y')
  u,U,o,O . Filter by: 'u'/'U' effective/any user; 'o'/'O' other criteria
  n,#,^O  . Set: 'n'/'#' max tasks displayed; Show: Ctrl+'O' other filter(s)
  V,v,F   . Toggle: 'V' forest view; 'v' hide/show children; 'F' keep focused

  d,k,r,^R 'd' set delay; 'k' kill; 'r' renice; Ctrl+'R' renice autogroup
  ^G,K,N,U  View: ctl groups ^G; cmdline ^K; environment ^N; supp groups ^U
  W,Y,!,^E  Write cfg 'W'; Inspect 'Y'; Combine Cpus '!'; Scale time ^E'
  q         Quit
          ( commands shown with '.' require a visible task display window )
Press 'h' or '?' for help with Windows,
Type 'q' or <Esc> to continue
```

- **SHIFT+M** permet de trier la liste de processus par mémoire résidente (RSS) de manière décroissante, extrait du resultat :
  
```bash
top - 20:34:18 up 1 day,  7:48,  2 users,  load average: 0.00, 0.00, 0.00
Tasks:  76 total,   1 running,  75 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1967.3 total,   1627.9 free,    223.5 used,    258.1 buff/cache
MiB Swap:   6173.0 total,   6173.0 free,      0.0 used.   1743.8 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  102168  12200   9204 S   0.0   0.6   0:00.93 systemd
   1831 root      20   0   17996  11332   9512 S   0.0   0.6   0:00.15 sshd
    531 root      20   0   18972  10632   8904 S   0.0   0.5   0:00.03 systemd
    521 root      20   0   15432   9464   8144 S   0.0   0.5   0:00.01 sshd
   1781 root      20   0   32952   8264   7232 S   0.0   0.4   0:00.16 systemd-journal
   1267 root      20   0   25376   7864   6868 S   0.0   0.4   0:00.07 systemd-logind
   1262 systemd+  20   0   90080   6660   5780 S   0.0   0.3   0:00.60 systemd-timesyn
......................
     21 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 inet_frag_wq
     22 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.01 khungtaskd
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reaper
     27 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 writeback
```

- Le processus en haut de la liste, le plus **gourmand**, est :

PID : 1, commande systemd

- Pour decouvrir ce que mon processus plus gourmand fais, j utilise le **man** (`man systemd`)

```bash
TEMD(1)                                             systemd                                             SYSTEMD(1)

NAME
       systemd, init - systemd system and service manager

SYNOPSIS
       /lib/systemd/systemd [OPTIONS...]

       init [OPTIONS...] {COMMAND}

DESCRIPTION
       systemd is a system and service manager for Linux operating systems. When run as first process on boot (as PID
       1), it acts as init system that brings up and maintains userspace services. Separate instances are started for
       logged-in users to start their services.

       systemd is usually not invoked directly by the user, but is installed as the /sbin/init symlink and started
       during early boot. The user manager instances are started automatically through the user@.service(5) service.

       For compatibility with SysV, if the binary is called as init and is not the first process on the machine (PID
       is not 1), it will execute telinit and pass all command line arguments unmodified. That means init and telinit
       are mostly equivalent when invoked from normal login sessions. See telinit(8) for more information.

       When run as a system instance, systemd interprets the configuration file system.conf and the files in
       system.conf.d directories; when run as a user instance, systemd interprets the configuration file user.conf
       and the files in user.conf.d directories. See systemd-system.conf(5) for more information.

CONCEPTS
       systemd provides a dependency system between various entities called "units" of 11 different types. Units
       encapsulate various objects that are relevant for system boot-up and maintenance. The majority of units are
       configured in unit configuration files, whose syntax and basic set of options is described in systemd.unit(5),
       however some are created automatically from other configuration files, dynamically from system state or
       programmatically at runtime. Units may be "active" (meaning started, bound, plugged in, ..., depending on the
       unit type, see below), or "inactive" (meaning stopped, unbound, unplugged, ...), as well as in the process of
       being activated or deactivated, i.e. between the two states (these states are called "activating",
       "deactivating"). A special "failed" state is available as well, which is very similar to "inactive" and is
       entered when the service failed in some way (process returned error code on exit, or crashed, an operation
       timed out, or after too many restarts). If this state is entered, the cause will be logged, for later
       reference. Note that the various unit types may have a number of additional substates, which are mapped to the
       five generalized unit states described here.

       The following unit types are available:

 Manual page systemd(1) line 1 (press h for help or q to quit)
```
**Explication** : systemd est un gestionnaire de système et de services pour Linux, qui s'exécute en tant que premier processus au démarrage. Il a pour rôle de lancer et de gérer les services qui tournent sur l'ordinateur. systemd utilise des fichiers de configuration pour organiser différents objets appelés "unités", qui peuvent être en cours d'exécution, arrêtés, ou dans un état de transition. Si un service échoue, systemd enregistre l'erreur pour qu'on puisse la consulter plus tard. En gros, il assure un démarrage rapide et une gestion efficace des services du système.

- Pour changer couleur de l'affichage, j'ai :
  1. Ouvert les options pour changer couleur avec la touche **Z**
  2. Choisi **l'element** que je veux changer :
S. Summary Data area.
M. Messages and prompts.
H. Column headings.
T. Task information in the process list.
  3. Choisi la **couleur**  que je voulais :
0. Black.
1. Red.
2. Green.
3. Yellow.
4. Blue.
5. Magenta.
6. Cyan.
7. White.

Affichage avec les couleurs modifies: 
![image](https://github.com/user-attachments/assets/0b176387-043e-476d-b478-f26d6ea843af)


- Pour mettre en avant le colonne de trie et changer la colonne de tri il y a different methodes:
1. **Utiliser les touches de tri :** Dans l'interface de `top`, on peut directement changer le critère de tri en appuyant sur certaines touches :
   - **P** : Trier par utilisation du CPU (%CPU).
   - **M** : Trier par utilisation de la mémoire (%MEM).
   - **N** : Trier par PID (identifiant de processus).
   - **T** : Trier par le temps CPU cumulatif (TIME+).

2. **Appuyer sur la touche `o` :** Cela permet de définir un champ spécifique pour le tri.
   - Après avoir appuyé sur `o`, entrer le nom du champ (comme `TIME`, `MEM`, etc.) par lequel trier.
   - On peut aussi préfixer le nom du champ avec `+` pour un tri descendant ou `-` pour un tri ascendant.

3. **Appuyer sur la touche O :** Ajouter un filtre à la sortie et définir les critères de filtrage. 
Il faut indiquer :
  - **FLD** : le nom du champ (par exemple, `%MEM`, `TIME`, etc.)
  - **VAL** : la valeur à comparer.

- Pour utiliser `htop` j'ai du l'installer à l aide de la commande `apt install htop `. Apres j'ai lance `htop`, et obtenu :
![image](https://github.com/user-attachments/assets/1871240a-f455-4ff6-b281-c2eb5d47a89c)

- **Avantages et inconvenients** htop, difference avec top :

1. **Installation :**
   - **top :** Préinstallé sur toutes les distributions Linux modernes, il est donc immédiatement disponible.
   - **htop :** Nécessite une installation manuelle.
     
2. **Support de la souris :**
   - **top :** Ne supporte pas l'utilisation de la souris, ce qui peut rendre l'interaction moins intuitive.
   - **htop :** Prend en charge la souris, offrant une expérience utilisateur plus agréable.

3. **Apparence visuelle**
- **top :** Présente une interface simple, se concentrant sur l'affichage des informations essentielles.
- **htop :** Offre une interface colorée, incluant des barres de progression pour chaque cœur de processeur et permettant d'utiliser la souris pour naviguer.

**Fonctionnalités**
- **htop** propose des fonctionnalités avancées telles que :
  - Affichage des processus en format arbre.
  - Possibilité de tuer des processus facilement avec la touche F9.
  - Filtres pour affiner la liste des processus.
  - Options de configuration supplémentaires pour personnaliser l'affichage.
  
- Bien que **htop** soit plus riche en fonctionnalités et en interactivité, il peut consommer légèrement plus de ressources système que **top**. Cela pourrait être un inconvénient sur des systèmes plus anciens ou avec des ressources limitées. 

Bien sûr ! Voici une version révisée en utilisant la première personne ou en adoptant un ton inclusif :

## 3 : Arrêt d'un processus

### Création des scripts

1. **Création de `date.sh`** :
   - J'ouvre un terminal et je crée le fichier avec :
     ```bash
     nano date.sh
     ```
   - J'écris le script suivant dans le fichier :
     ```bash
     #!/bin/sh
     while true; do 
         sleep 1; 
         echo -n 'date '; 
         date +%T; 
     done
     ```

2. **Création de `date-toto.sh`** :
   - Je crée le second fichier :
     ```bash
     nano date-toto.sh
     ```
   - J'écris le script suivant :
     ```bash
     #!/bin/sh
     while true; do 
         sleep 1; 
         echo -n 'toto '; 
         date --date '5 hour ago' +%T; 
     done
     ```

### Exécution des scripts

- Je lance le premier script :
  ```bash
  ./date.sh
  ```

- Ensuite, je lance le second script :
  ```bash
  ./date-toto.sh
  ```

- Je mets les deux processus en arrière-plan avec `Ctrl+Z`.

### Affichage des processus en arrière-plan

- J'utilise la commande `jobs` pour afficher les processus :
  ```bash
  jobs
  ```
  Résultat :
  ```
  [1]-  Stopped                 ./date.sh
  [2]+  Stopped                 ./date-toto.sh
  ```

### Remise au premier plan et arrêt des processus

- Je remets le premier processus au premier plan :
  ```bash
  fg %1
  ```

- J'arrête le processus avec `Ctrl+C`. Voici ce que je vois :
  ```
  date 17:45:25
  date 17:45:26
  date 17:45:27
  ^C
  ```

- Je remets le deuxième processus au premier plan :
  ```bash
  fg %2
  ```

- J'arrête ce processus également avec `Ctrl+C`. Résultat :
  ```
  toto 12:46:14
  toto 12:46:15
  toto 12:46:16
  ^C
  ```

### Vérification des processus en cours

- Je relance les scripts `date.sh` et `date-toto.sh`, puis je vérifie les processus en cours avec la commande `ps` :
  ```bash
  ps
  ```
  Résultat :
  ```
      PID TTY          TIME CMD
     2754 pts/3    00:00:00 bash
     8725 pts/3    00:00:00 date.sh
     8766 pts/3    00:00:00 sleep
     8797 pts/3    00:00:00 date-toto.sh
     8806 pts/3    00:00:00 sleep
     8823 pts/3    00:00:00 ps
  ```

### Arrêt forcé des processus

- Pour arrêter les processus, j'utilise la commande `kill` avec le préfixe `-9` et le PID du processus :
  ```bash
  kill -9 8725
  ```
  Résultat :
  ```
  [1]+  Killed                  ./date.sh
  ```

- Ensuite, j'arrête le second processus :
  ```bash
  kill -9 8797
  ```
  Résultat :
  ```
  [2]+  Killed                  ./date-toto.sh
  ```
- Je relance la commande `ps` pour vérifier que les deux processus ne sont plus présents

### Explication des scripts

**Script 1 : `date.sh`**
- while true; do ... done` s'exécute indéfiniment.
- `sleep 1` suspend l'exécution pendant 1 seconde pour éviter une surcharge.
- `echo -n 'date '` affiche le mot `date` sans saut de ligne.
- `date +%T` affiche l'heure actuelle au format `HH:MM:SS`.

**Script 2 : `date-toto.sh`**
- `sleep 1` pour un intervalle de 1 seconde.
- `echo -n 'toto '` affiche le mot `toto` sans saut de ligne.
- `date --date '5 hour ago' +%T` affiche l'heure qu'il était il y a 5 heures au format `HH:MM:SS`.

## 4 : les tubes

- **`cat`** : Cette commande lit le contenu d'un ou plusieurs fichiers et l'affiche sur la sortie standard (généralement le terminal). Elle est principalement utilisée pour visualiser le contenu d'un fichier, mais ne permet pas d'enregistrer cette sortie dans un autre fichier.

- **`tee`** : À la différence de `cat`, `tee` lit une entrée et écrit simultanément cette entrée dans un fichier tout en l'affichant sur la sortie standard. Cela permet à l'utilisateur de visualiser les données tout en les enregistrant.

### Explication des commandes

1. **`$ ls | cat`**
   - Cette commande exécute `ls` pour lister les fichiers dans le répertoire courant, puis redirige cette sortie vers `cat`. Le résultat affiché est identique à celui de `ls`, car `cat` ne modifie pas la sortie.

2. **`$ ls -l | cat > liste`**
   - Ici, `ls -l` génère une liste détaillée des fichiers. La sortie est ensuite redirigée vers `cat`, qui enregistre le contenu dans le fichier `liste`. Cette opération pourrait être simplifiée en utilisant simplement `ls -l > liste`.

3. **`$ ls -l | tee liste`**
   - Cette commande exécute `ls -l` pour afficher la liste détaillée des fichiers. `tee` permet de visualiser cette sortie sur le terminal tout en l'enregistrant dans le fichier `liste`. Cela est utile pour conserver une trace de la sortie tout en l'affichant.

4. **`$ ls -l | tee liste | wc -l`**
   - Dans cette commande, `ls -l` génère la liste détaillée des fichiers, qui est ensuite passée à `tee`. Ce dernier affiche la sortie tout en l'écrivant dans le fichier `liste`. Ensuite, `wc -l` compte le nombre de lignes dans la sortie, fournissant ainsi le total de fichiers listés tout en conservant cette liste dans le fichier.


Voici une version améliorée et bien structurée de votre texte sur le journal système rsyslog :

## 5 : Journal système rsyslog

### Service rsyslog

- **Le service rsyslog est-il lancé sur votre système ? Quel est le PID du démon ?**
  
  Pour vérifier si le service rsyslog est actif, j'utilise la commande suivante :
  ```bash
  systemctl status rsyslog
  ```
  Comme le service n'était pas installé, je l'ai installé avec :
  ```bash
  apt-get install rsyslog
  ```

  Ensuite, je vérifie à nouveau s'il est actif :
  ```bash
  service rsyslog status
  ```
  Résultat :
  ```bash
  ● rsyslog.service - System Logging Service
       Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
       Active: active (running) since Tue 2024-10-15 18:09:36 UTC; 23s ago
       TriggeredBy: ● syslog.socket
       Docs: man:rsyslogd(8)
             man:rsyslog.conf(5)
             https://www.rsyslog.com/doc/
     Main PID: 16357 (rsyslogd)
        Tasks: 4 (limit: 2315)
       Memory: 3.3M
          CPU: 4ms
       CGroup: /system.slice/rsyslog.service
               └─16357 /usr/sbin/rsyslogd -n -iNONE
  ```

  Le PID du démon est 16357:
  Main PID: 16357 (rsyslogd)
  

### Fichiers de configuration

- **Dans quel fichier rsyslog écrit-il les messages issus des services standards ? Et la plupart des autres messages ?**
  
  Les messages issus des services standards et les autres sont généralement écrits dans :
  - `/var/log/syslog`
  - `/var/log/auth.log` pour les messages d'authentification
  - `/var/log/kern.log` pour les messages du noyau

  Pour vérifier le contenu de ces fichiers, on peut utiliser :
  ```bash
  cat /var/log/syslog
  ```

  Extrait du contenu de `/var/log/syslog` :
  ```
  2024-10-15T18:09:36.998236+00:00 serveur1 systemd[1]: Listening on syslog.socket - Syslog Socket.
  2024-10-15T18:09:36.998477+00:00 serveur1 systemd[1]: Starting rsyslog.service - System Logging Service...
  2024-10-15T18:09:36.997445+00:00 serveur1 rsyslogd: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.2302.0]
  ```

### Service cron

- **À quoi sert le service cron ?**
  
  Le service cron permet de planifier l'exécution de tâches de manière répétée à des intervalles réguliers. Cela est utile pour automatiser des tâches telles que des sauvegardes, des mises à jour ou des scripts de maintenance. Je n'avais pas ce service, donc je l'ai installé.

### Commande `tail -f`

- **Que fait la commande `tail -f` ?**
  
  La commande `tail -f` affiche les dernières lignes d'un fichier et continue à surveiller ce fichier pour afficher de nouvelles lignes ajoutées en temps réel. C'est particulièrement utile pour visualiser les journaux système.

  Pour visualiser en temps réel le fichier `/var/log/syslog`, j'utilise :
  ```bash
  tail -f /var/log/syslog
  ```

- **Que voyez-vous si vous redémarrez le service cron depuis un autre shell ?**
  
  Après avoir redémarré le service avec la commande :
  ```bash
  systemctl restart cron
  ```
  je ne vois rien dans le terminal.

### Fichier `logrotate.conf`

- **À quoi sert le fichier `/etc/logrotate.conf` ?**
  
  Ce fichier est utilisé pour configurer la rotation des journaux. Il permet de définir des règles sur la fréquence de rotation des fichiers journaux, le nombre de fichiers de sauvegarde à conserver, et d'autres paramètres pour gérer la taille et la durée de conservation des journaux.

### Sortie de la commande `dmesg`

- **Examiner la sortie de la commande `dmesg`. Quel modèle de processeur Linux détecte-t-il sur votre machine ? Quels modèles de cartes réseaux détecte-t-il ?**
  
  Pour examiner la sortie, j'exécute :
  ```bash
  dmesg
  ```
  Extrait de la sortie :
  ```bash
  [    0.000000] Linux version 6.1.0-25-amd64 (debian-kernel@lists.debian.org) ...
  [    0.163442] smpboot: CPU0: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz ...
  [    0.709485] e1000: Intel(R) PRO/1000 Network Driver
  [    1.121306] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
  ```

  - **Info processeur** :

    [    0.163442] smpboot: CPU0: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz (family: 0x6, model: 0x8c, stepping: 0x1)

  - **Info cartes réseau** :
  [    0.709485] e1000: Intel(R) PRO/1000 Network Driver
[    1.121306] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection



- Sources: https://www.baeldung.com/linux/ps-command, https://www.digitalocean.com/community/tutorials/linux-ps-command, https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server#step-2-copying-an-ssh-public-key-to-your-server, https://linuxhandbook.com/top-vs-htop/, https://phoenixnap.com/kb/top-command-in-linux, https://doc.ubuntu-fr.org/logrotate
