
# TP 6 - Gestion des disques / Tâches d’administration


## Exercice 1 Disques et partitions

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement**
**alloués ; puis démarrez la VM**

Euh. C'est fait. Je vois pas quoi dire de plus ahah.


**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

```sudo fdisk -l```
permet d'afficher toutes les partitions du système et on remarque le nouveau disque dur qui s'appelle /dev/sdb

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),**
**et une seconde partition de 3 Go en NTFS (n°7)**

On rentre en mode commande avec la commande
```sudo fdisk /dev/sdb```
ensuite on fait
```console
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): o^Hp^C
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G
Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4196352-10485759, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10485759, default 10485759):

Created a new partition 2 of type 'Linux' and of size 3 GiB.

Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 7

Changed type of partition 'Linux' to 'HPFS/NTFS/exFAT'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```



**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.**
**A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

```console
adrien@server:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: ed3f5b95-0a1a-4f0e-9f0a-c53e14f4e38d
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

adrien@server:~$ sudo mkfs.ntfs /dev/sdb2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
```


**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

Cette commande ne fonctionne pas car les partitions ne sont pas montées.

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la**
**machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des**
**UUID en raison de l’impossibilité d’effectuer des copier-coller)**

```
sudo nano /etc/fstab

UUID=316279b3-d46a-40f8-875a-ccfa6b18a42c / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
UUID=d3f5b95-0a1a-4f0e-9f0a-c53e14f4e38d /data ext4 defaults 0 0
UUID=10A498163A70F8B8 /win ntfs defaults 0 0
```


**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

```
sudo mount -a
adrien@server:~$ df -T /dev/sdb1
Filesystem     Type 1K-blocks  Used Available Use% Mounted on
/dev/sdb1      ext4   1998672  6144   1871288   1% /data
adrien@server:~$ df -T /dev/sdb2
Filesystem     Type    1K-blocks  Used Available Use% Mounted on
/dev/sdb2      fuseblk   3144700 16264   3128436   1% /win
```

**8. Montez votre clé USB dans la VM**


**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer**
**les Additions invité de VirtualBox**

## Exercice 2. Partitionnement LVM
**Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler**
**les disques et les partitions.**

**1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de**
fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes
du fichier /etc/fstab

**2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM**
** La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand**
**on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de**
**gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur**
**ce disque.**
** Attention à ne pas supprimer la partition système !**

**3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en**
**utilisant la commande pvdisplay.**

** Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant**
**les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.**

**4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que**
**le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.**
** Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice**
**du groupe de volume, en commençant par 00, puis 01...)**

**5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.**
** On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par**
**exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l**
**60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour**
**utiliser la totalité de l’espace libre.**

**6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans**
**l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.**
** A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par**
**exemple agrandir une partition à l’aide d’un nouveau disque.**

**7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez**
**la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.**

**8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de**
**volumes**

**9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas**
**oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.**
** Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par**
**bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de**
**présenter les fonctionnalités de base.**

## Exercice 3. Exécution de commandes en différé : at et cron

**1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez**
**entre temps que la tâche est bien programmée.**

**2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par**
**exemple en vous aidant des logs, du manuel...)**

**3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,**
**l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.**

**4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure**

**5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18**
**heures les 1er et 15 du mois :**

**6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures**

**7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un**
**fichier de log situé dans votre dossier personnel**

**8. Videz votre crontab**
