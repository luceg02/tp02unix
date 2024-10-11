# tp02unix
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

#### `PermitRootLogin`
Le paramètre `PermitRootLogin` dans le fichier de configuration SSH contrôle si l'utilisateur root est autorisé à se connecter via SSH et, si oui, par quelles méthodes. Voici les différentes options :

- `PermitRootLogin yes`
  
- **Description** : Autorise toutes les méthodes d'authentification pour l'utilisateur root, y compris le mot de passe.
- **Avantages** : Permet un accès root direct via SSH.
- **Inconvénients** : C’est une option risquée, car elle expose le compte root à des attaques par force brute sur le mot de passe, surtout si le mot de passe est faible.
- **Cas d'utilisation** : À éviter dans la plupart des cas pour des raisons de sécurité, mais peut être utile temporairement lors de configurations initiales ou de débogage en cas de perte d'accès.

- `PermitRootLogin prohibit-password`

- **Description** : Autorise uniquement l'authentification par clé publique pour root ; les méthodes d'authentification par mot de passe et clavier interactif sont désactivées.
- **Avantages** : Améliore la sécurité en autorisant seulement l'accès par clé publique pour root, empêchant ainsi les attaques par force brute via le mot de passe.
- **Inconvénients** : Si les clés SSH sont compromises, le compte root reste vulnérable, et cela peut nécessiter une configuration et une gestion des clés plus rigoureuses.
- **Cas d'utilisation** : Recommandé pour les environnements de production où l'accès root direct est nécessaire. 

- `PermitRootLogin forced-commands-only`

- **Description** : Permet l'accès root par clé publique uniquement si une commande spécifique est définie pour l'authentification. Toutes les autres méthodes d'authentification pour root sont désactivées.
- **Avantages** : Utile pour des cas spécifiques comme les sauvegardes automatisées, car l'accès root est limité à des commandes pré-autorisées.
- **Inconvénients** : Ne permet pas d'accès interactif pour root. Cela nécessite une configuration rigoureuse.
- **Cas d'utilisation** : Utilisé pour des automatisations spécifiques où un accès root est nécessaire, mais un accès complet n’est pas requis, par exemple pour des scripts de sauvegarde.

- `PermitRootLogin no`

- **Description** : Désactive complètement l'accès root via SSH, quel que soit le type d'authentification.
- **Avantages** : Le plus sécurisé, car il empêche tout accès SSH direct au compte root.
- **Inconvénients** : Les utilisateurs devront utiliser `sudo` pour les tâches nécessitant root.
- **Cas d'utilisation** : Fortement recommandé pour les environnements de production. Les utilisateurs se connectent avec des comptes normaux et utilisent `sudo` pour des actions root, ce qui ajoute une couche de sécurité.

### `PasswordAuthentication`

Le paramètre `PasswordAuthentication` détermine si l'authentification par mot de passe est autorisée pour tous les utilisateurs, pas seulement pour root. Voici les options disponibles :

#### a. `PasswordAuthentication yes`
- **Description** : Autorise tous les utilisateurs à se connecter via un mot de passe.
- **Avantages** : Simple pour les utilisateurs car ils peuvent se connecter sans nécessiter de clé SSH.
- **Inconvénients** : Expose le serveur aux attaques par force brute de mot de passe, surtout si les mots de passe sont faibles ou s'il n'existe pas de protection.
- **Cas d'utilisation** : Peut être utile dans un environnement de test ou pour des utilisateurs ayant des restrictions temporaires sur la clé publique, mais à éviter dans la plupart des cas.

#### b. `PasswordAuthentication no`
- **Description** : Désactive l'authentification par mot de passe pour tous les utilisateurs. L'authentification se fait uniquement par clé publique.
- **Avantages** : Réduit considérablement le risque d'attaques par force brute de mot de passe, car aucun mot de passe ne peut être utilisé pour accéder au serveur.
- **Inconvénients** : Exige que tous les utilisateurs disposent de clés SSH pour accéder au serveur, ce qui peut nécessiter une configuration supplémentaire.
- **Cas d'utilisation** : Recommandé pour la plupart des environnements de production. L’authentification par clé est plus sécurisée et protège contre les attaques par force brute de mot de passe.

## 1.2: Authentification par clef / Generation de clefs 
commencer par generer un couple de clef prive public sur la machine hote, avec la commande:
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

### 2. C'est quoi la difference entre la clé publique et la clé privée ?
La clé publique et la clé privée sont deux parties d'une **paire de clés** utilisée dans les systèmes de cryptographie asymétrique, comme SSH. Elles fonctionnent ensemble pour sécuriser les connexions. Voici leurs différences principales :

### 1. Clé publique
- **Utilisation** : La clé publique est **placée sur les serveurs** auxquels von veut accéder. C’est elle qui autorise la connexion de la clé privée correspondante. Elle est destinée à être **partagée** et copiée sur les serveurs. C’est grâce à cette clé publique que le serveur peut vérifier votre identité lorsque vous vous connectez avec la clé privée.
- **Sécurité** : La clé publique ne contient **aucune information sensible** à elle seule. Même si quelqu’un l’obtient, il ne pourra pas s'en servir pour accéder à votre compte sans la clé privée.

### 2. Clé privée
- **Utilisation** : La clé privée reste **sur votre machine locale** et est utilisée pour **authentifier votre identité** auprès du serveur. Elle fonctionne comme une "signature" prouvant que vous êtes bien l’utilisateur autorisé. La clé privée est **strictement personnelle** et **ne doit jamais être partagée**. Elle est gardée en sécurité sur votre appareil local, souvent protégée par une passphrase.
- **Sécurité** : La clé privée contient des informations sensibles. Si quelqu’un y accède, il pourrait se connecter aux serveurs où la clé publique est installée. C'est pourquoi il est recommandé de la protéger avec une passphrase.

Donc la clé publique **va sur le serveur** pour autoriser l’accès, et la clé privée **reste sur l'appareil** pour prouver votre identité. La clé publique peut être partagée sans risque, tandis que la clé privée doit être gardée secrète pour assurer la sécurité des connexions.

## 1.3:  Authentification par clef / Connection serveur (copier la clef)

#### 1. Verification fichier clef :
   - J'ai vérifié la présence de mes clés (`id_rsa` et `id_rsa.pub`) en exécutant la commande `ls`. Cela a affiché les fichiers existants dans le dossier :
     ```powershell
     PS C:\Users\luceg> cd .ssh
     PS C:\Users\luceg\.ssh> ls
     ```
   - Résultat :
     ```
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






