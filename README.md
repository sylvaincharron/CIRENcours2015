# CIRENcours2015
Analyse de données IRMf, cours du CIREN, mars 2015

## 1. Organisation des données

###Les données en IRM fonctionnelle d'activation
Elles comportent au minimum

  - des données d'imagerie cérébrale : images au format DICOM ou NIFTI ou autre
  - des informaitons sur les paramètres d'acquisition
  - des données comportementales : .csv ou .txt au format UTF-8, habituellement générées automatiquement par votre logiciel de stimulation
  - des informations sur la tâche, que vous devez avoir écrites (préparez-vous à expliquer votre tâche rapidement et précisément, ça aide pour les talks/posters/ascenseurs)


###Toujours savoir ce qu'il y a dans vos données !

Trois sources d'information principales

  - votre cahier de manip, correctement rempli pendant les acquisitions
  - le personnel et les documents au CIREN, voire au CRC. N'hésitez pas à poser vos questions à Stéphanie Lion, notre manip de recherche
  - les fichiers que vous avez récupéré (correctement nommés et organisés si vous avez bien fait votre travail)

Pour les données d'imagerie, utilisez un viewer DICOM pour récupérer les informations sur les séquences que vous avez acquises.
Ex :

 - Mango, gratuit et conçu spécialement pour la recherche en imagerie cérébrale
 - ImageJ, gratuit et léger, génraliste
 - Osirix, pour mac
 - Les consoles constructeur, comme l'AW au CIREN

###Réorganisation des données dans votre répertoire d'analyse

Suivre les recommandantions de [Open fMRI](https://openfmri.org/content/data-organization) qui vous aideront à ne pas oublier d'information et à toujours conserver une structure propre pour vos données.

Créer un répertoire qui sera celui de base pour toute l'analyse de données IRMf pour ce cours
ex : /home/sc/Documents/cours_irmf_CIREN_2015

L'arborescence peut se faire en utilisant votre navigateur de fichier préféré ou en ligne de commande de la façon suivante :

```bash
mkdir ~/Documents/cours_irmf_CIREN_2015
cd ~/Documents/cours_irmf_CIREN_2015
mkdir ds000001
touch ds000001/README
touch ds000001/demographics.txt
touch ds000001/task_key.txt
touch ds000001/scan_key.txt
touch ds000001/model_key.txt
mkdir ds000001/models ds000001/group
mkdir ds000001/sub001 
mkdir ds000001/sub001/anatomy ds000001/sub001/behav ds000001/sub001/BOLD ds000001/sub001/model
```

Si vous avez plusieurs sujets, vous pouvez facilement créer toute l'arborescence d'un coup avec une petite boucle for

```bash
for suj in {1..25};
do
    echo "arborescence pour sujet numero $suj"
    if [[ $suj -lt 9 ]];
    then
        mkdir ds000001/sub00$suj 
        mkdir ds000001/sub00$suj/anatomy ds000001/sub00$suj/behav ds000001/sub00$suj/BOLD ds000001/sub00$suj/model
    else
        mkdir ds000001/sub0$suj 
        mkdir ds000001/sub0$suj/anatomy ds000001/sub0$suj/behav ds000001/sub0$suj/BOLD ds000001/sub0$suj/model
    fi
done
```

Évidemment vous pouvez faie la même chose en python ou autre langage de votre choix.



###Il vous reste à remplir cette arborescence avec les données que vous avez.

Je vous conseille :

1. de ne **jamais** modifier vos données brutes, vous pouvez monter le volume qui contient vos données en lecture seule pour être tranquille, par exemple avec tout système qui supporte un shell avec `mount -o rdonly`
2. de toujours utiliser un utilitaire de copie fiable, par exemple rsync, qui conserve les propriétés des fichiers de départ et vous aidera à reprendre votre copie proprement si elle est mar malheur interrompue.

```bash
sudo mount -o rdonly /dev/identifiant_de_votre_partition /media/point_de_montage
# creation du repertoire d'analyse de données IRMf
cd ~/Documents/cours_irmf_CIREN_2015/
# pour notre run d'images fonctionnelles
rsync -a /media/point_de_montage/mon_stock_de_data/sujet01/DICOM/PA0/ST0/SE2 ds000001/sub001/BOLD/
mv ds000001/sub001/BOLD/SE2 ds000001/sub001/BOLD/task001_run001_dicom
# on prépare déjà un petit répertoire pour la conversion
mkdir ds000001/sub001/BOLD/task001_run001_nifti3d
# et pour l'image structurelle
rsync -a '/media/point_de_montage/mon_stock_de_data/sujet01/DICOM/PA0/ST0/SE9' ds000001/sub001/anatomy/
mv ds000001/sub001/anatomy/SE9 ds000001/sub001/anatomy/anatomy_dicom

```

Sous mac pour monter une partition en lecture seule, vous pouvez utiliser la commande `diskutil`

```bash
# pour identifier une partition (de toute façon diskutil prend en compte tout le disque pour monter et démonter)
diskutil list
diskutil info /dev/ma_partition
# pour monter un disque lecture seule
diskutil unmountDisk /dev/mon_disque
diskutil mountDisk readOnly /dev/mon_disque
```

Remplissez bien aussi les fichiers texte qui décrivent vos données, dans l'idéal, rédigez la partie méthode d'acquisition de votre article à ce moment-là !
cf  
[Poldrack R. A., Fletcher P. C., Henson R.N., Worsley K. J., Brett M., and Nichols T. E., Guidelines for reporting an fMRI study. Neuroimage. 2008 Apr 1; 40(2): 409–414 | doi:  10.1016/j.neuroimage.2007.11.048](http://www.sciencedirect.com/science/article/pii/S1053811907011020)

## 2. Conversion des données avec SPM12 standalone

Malheureusement avec SPM, il faut procéder en deux étapes.
Comme SPM n'est pas bien conçu pour le traitement des nifti 4D (contrairement à FSL...), il vaut mieux faire la conversion depuis les DICOM directement avec SPM, sans passer par un autre logiciel, sinon l'opération de concaténation des images en une seul fichier 4D risque de comporter des erreurs.
Par ailleurs si vous avez des données nifti compressées (.nii.gz), SPM ne les supporte pas trop (contrairement à FSL...), donc décompressez vos données en .nii avant de les copier.

### lancement de SPM12 standalone

1. Lancez SPM12 standalone
2. Cliquez sur le bouton `fMRI`
3. vous voyez apparaître trois fenêtres :
  - "SPM12 (6225): Menu" : le menu, celle qui est toute verte, généralement en haut à gauche, avec les boutons regroupés par phase de traitement des données
  - "SPM12 (6225): Graphics" : celle à droite où s'afficheront toutes les sorties graphiques, qui porte le logo suivi de quelques informations et liens
  - "SPM12 (6225)" généralement en bas à gauche avec écrit "SPM12" en filigrane, qui sert à afficher la barre de progression des traitement et plus tard à beaucoup d'autres choses.
3. Je vous conseille d'entrée de jeu de vous placer dans votre répertoire d'analyse de données, pour ce faire, dans la fenêtre de menu, cliquez sur la liste `Utils` et sélectionnez `CD`
4. Un pop-up "Select new working directory" s'ouvre, il est constitué de deux panneaux
  - à gauche le panneau de **navigation**, où un clic simple sur un nom de répertoire permet d'y descendre
  - à droite le panneau de **sélection**, où un clic simple sur un nom de répertoire ou de fichier le sléectionne, il disparaît alors du panneau de sélection et s'affiche 
  - en bas, dans le panneau qui récapitule les objects sélectionnés
5. Sélectionnez donc votre répertoire d'analyses et cliquez sur le bouton `Done` quand vous avez fini

Remarques :

  - le champ "Filter" s'utilise avec des expressions régulières
  - les trois champs en haut : "Dir", "Up" et "Prev" aident à la navigation dans l'arborescence.
  - la première ligne de la liste déroulante "Prev" contient toujours le path vers le répertoire d'installation de SPM12 standalone.
 
### Conversion des dicom en nifti 3D

####Image anatomique
1. Dans la fenêtre de menu , cliquez sur le bouton `DICOM Import`, ce qui ouvre une fenêtre pop-up "Batch Editor"
2. Dans la colonne de droire, il va falloir spécifier les valeurs de paramètres nécessaires. Ce qu'il faut remplir est indiqué par `<-X`
3. Sélectionner le champs "DICOM files" et appuyez sur le bouton `Specify...`
4. Une fenêtre pop-up de sélection de fichiers s'ouvre. Naviguez dans votre arborescence avec la fenêtre de gauche pour sélectionner le répertoire qui continent les images DICOM, c'est-à-dire, depuis votre répertoire de travail :
`ds000001/sub001/anatomy/anatomy_dicom`
5. Sélection des images DICOM : dans la fenêtre de sélection, faites un clic droit puis cliquez sur le petit `Select All` qui pop-up. Vérifiez que vous avez bien sélectionné 176 images.
6. Sélection du répertoire : même principe mais sélectionnez le répertoire où seront les images anatomiques : `ds000001/sub001/anatomy`
7. Vérifiez que vous avez bien sélectionné le format d'image : ".Output image format         Single file (nii) NIfTI"
8. Lancez le batch avec le petit triangle vert sur la ligne d'icônes en haut de la fenêtre "Batch Editor".
9. Une fois la conversion effectuée, vous devez voir un nouveau fichier dans le répertoire `ds000001/sub001/anatomy` : `stest-0011-00001-000001-01.nii`
10. donner-lui un nom plus convenable, par exemple :
  - En ligne de commande `mv ds000001/sub001/anatomy/stest-0011-00001-000001-01.nii ds000001/sub001/anatomy/anat.nii `
  - avec l'interface de "Batch Editor" : dans la fenêtre menu, cliquez sur `Batch`, puis parmi les onglets de la fenêtre "Batch Editor", sur `BasicIO -> File/Dir Operations -> File Operations -> Move/Delete Files`, puis remplissez les paramètres de la façon suivante :
```
Files to move/copy/delete     ...ds000001/sub001/anatomy/stest-0011-00001-000001-01.nii
Action
.Move and Rename
..Move to                    .../cours_irmf_CIREN_2015/ds000001/sub001/anatomy/
..Pattern/Replacement List
...Pattern/Replacement Pair
....Pattern                  stest-0011-00001-000001-01.nii
....Replacement              anat
..Unique Filenames           Don't Care
```
Remarques :

  - vous pouvez faire l'étape 3 en cliquant d'abord sur le bouton `Batch` du menu, puis en sélectionnant parmi les onglets de la fenêtre "Batch Editor" `SPM -> Util -> Import -> DICOM Import`.
  - pour modifier une sélection, il faut à nouveau cliquer que le bouton `Specify...`, puis il suffit de cliquer sur un des item de la liste de sélection en bas de la fenêtre pour le retirer de la liste, sélectionnez ensuite un autre fichier comme précédemment.

### Conversion des nifti 3D en nifti 4D
Intérêt : un seul fichier pour toute la série temporelle ! C'est beaucoup moins pénible à manipuler et ça n'encombre pas l'ordi avec des tas de fichiers qui sont une plaie pour les transferts et les sauvegardes.

Batch(programmation avec interface graphique) puis dans les onglets choisir SPM puis Util  puis 3D to 4D conversion
 
Choisirles images en 3D puis le output
 
On a maintenant deux fichiers
dans la fenêtre menu, cliquez sur `Batch`, puis parmi les onglets de la fenêtre "Batch Editor", sur `BasicIO -> File/Dir Operations -> File Operations -> Move/Delete Files`, puis remplissez les paramètres de la façon suivante :

  - un fichier .mat (avec l’information d’orientation de l’image dans l’espace) : c’est celui ci qui va être modifié par les opérations de recalage des prétraitements. 
 - un fichier image .nii

