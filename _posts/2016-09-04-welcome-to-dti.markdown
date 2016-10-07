---
layout: post
title:  "Welcome to DTI!"
date:   2016-09-05 16:48:29 +0200
categories: jekyll update
---
> Ceci est une page en construction.

## 1) Introduction

Bienvenue sur ce tutorial consacré au traitement des données Bruker de diffusion et de haute résolution pour l'analyse de la structure cardiaque.

![image1](../../../../../images/image1.png)

&nbsp;

### 1.1) Sommaire


Les questions ( dans le désordre ) que vous vous posez.

* [Par où commencer ?](#prerequis)

* [Est-ce que c'est compliqué ?](#prerequis)

* [Comment extraire les données en format DICOM ?](#dicomExtraction)

* [Qu'est ce que la diffusion ?](#diffusion)

* [Quels logiciels utiliser pour effectuer la segmentation ?](#logiciel)

* [Comment exporter ma segmentation sous forme de mesh ? ](#nomAncre1)

* [Comment créer un gif traversant l'échantillon en short axis ? ](#nomAncre2)

* [Quelques exemples de figures à présenter.](#joliesfigures)

* [Le lexique.](#lexique)

&nbsp;

### 1.2) Pré-requis <a id="prerequis"></a>

Aucun pré-requis n'est nécessaire, si toutefois ce tutorial est incomplet et/ou contient des erreurs/approximations, n'hésitez pas à nous en faire part. Vous pouvez évidemment contribuer, corriger, et/ou créer très rapidement votre page.

Compter une demi-journée de travail pour traiter votre premier jeu de données. Après quelques jours, 30 minutes suffisent pour tout effectuer.

&nbsp;

### 1.3) (à compléter) Extraction des données de la console Bruker sous format DICOM  <a id="dicomExtraction"></a>

Nous allons utliser le logiciel `Paravision` de Bruker pour extraire les images au format DICOM.

&nbsp;

### 1.4) (à compléter) Copie des données de la console Bruker sous format Bruker

Lors de l'examen, les données Bruker sont stockés dans des dossiers numérotés par ordre d'acquisition des séquences. Par exemple, nous allons extraire le dossier numéro `35` du l'examen intitulé `2016-09-09-examen`.

Pour cela, ouvrir le menu puis aller dans le dossier ou ouvrir un terminal et taper la commande suivante:

{% highlight ruby %}
#se déplacer dans un répertoire
cd /opt/PV6/.../2016-09-09-examen
#taper la commande suivante pour obtenir la liste des dossiers
ls
{% endhighlight %}

&nbsp;


### 1.5) Arborescence des fichiers Bruker

Pour chaque acquisition numérotée de 1 à N, nous retrouvons la même arborescence avec:

* un fichier texte `acqp` qui contient les paramètres d'acquisition.
* un fichier texte `method` qui contient d'autres paramètres d'acquisition.
* un fichier binaire `fid` qui contient les données brutes ie non reconstruites.
* une sous dossier `pdata` pour `Processed Data`, qui contient les données reconstruites. Attention plusieurs reconstructions peuvent être effectuées à partir des mêmes données.
* la liste est non exclusive, pour tout renseignement supplémentaire, ce [lien](http://imaging.mrc-cbu.cam.ac.uk/imaging/FormatBruker) est plus complet.

{% highlight ruby %}
#toujours à partir du terminal, pour vérifier quelle séquence a été jouée,
cd /opt/PV6/.../2016-09-09-examen
grep -R  '##$Method=' 35/
{% endhighlight %}

Dans le sous-dossier `pdata` (par ex. `2016-09-09-examen/35/pdata`), nous trouvons les sous dossiers numérotées (`1`,`2`,`3` etc) pour chaque nouvelle reconstruction. Nous serons généralement intéressé par les reconstructions numérotées `1` ou `2` suivant les cas.

Dans chaque dossier de reconstruction (par ex. `2016-09-09-examen/35/pdata/1`), il y a:

* un fichier binaire `2dseq`. C'est le bloc de données qui contient les images en format 3D ou 4D. Par exemple, les données correspondant au direction de diffusion sont stockées successivement à l'intérieur de ce bloc. Nous verrons plus tard comment les extraire.
* un fichier texte `reco` qui contient les détails concernant la reconstruction effectuées. Par ex. le champ de vue, le nombre de pixels et la résolution
* un fichier texte `visu_pars` qui contient d'autres détails concernant la reconstruction effectuées.
* la liste est aussi non exclusive, pour tout renseignement supplémentaire, ce [lien](http://imaging.mrc-cbu.cam.ac.uk/imaging/FormatBruker) est plus complet.

&nbsp;


### 1.6) Contribuer à ce tutoriel

&nbsp;

## 2) Quelques pré-requis organisationnels

> Avant de commencer quelques conseils. Les données peuvent être copiées sur votre espace personnel de stockage : smb://prenom.nom et utilisées sur le pc de post-traitement de l'équipe imagerie. Il est préférable de ne pas créer de doublons de vos données pour préserver l'espace dsique.

### 2.1) Création de l'arborescence des données IRM

Les données sont copiées par exemple dans le répertoire `/media/nelsonleouf/sdb1/DICOM/` et sont classées par espèce, puis numérotées arbitrairement pour chaque échantillon, elles sont généralement classées par ordre d'acquisition.

* `Kadence`
  * `Control`
    * `Heart_1`     (janvier 2016)
    * `Heart_2`     (avril 2016)
    * `Heart_3`     (mai 2016)
  * `Infarct`
Dans chaque dossier nous retrouvons les données de diffusion notées et les données de haute résolution si les deux sont présentes.

  * `Control`
    * `Heart_1`  
      * `30`        (données de diffusion)
      * `4`        (données de haute-résolution)         
    * `Heart_2`

### 2.2) Création de l'arborescence des données de post-traitées

Pour préserver les données acquises de toute mauvaise manipulation, le travail de post-traitement est effectué dans un nouveau dossier nommé `STDT`. Il est généré en lancant le script suivant:

{% highlight ruby %}
cd /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/
#génération des sous dossier
sh createfolder.sh
{% endhighlight %}

Si ce fichier n'est pas présent vous pouvez facilement le générer en suivant ces commandes:

{% highlight ruby %}
#génération des sous dossier
cd /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/
echo ''  > createfolder.sh
echo ''  >> createfolder.sh
#et enfin
sh createfolder.sh

{% endhighlight %}

Ainsi nous disposons de l'arborescence suivante:

* `Kadence`
  * `Control`
    * `Heart_1`  
      * `30`        (données de diffusion)
      * `4`        (données de haute-résolution)  
      * `STDT`      (dossier de post-traitement)
        * `ST`      
        * `DT`
        * `Stat`      
&nbsp;


### 2.3) Les librairies Visualisation Tool Kit (VTK) et Image Tool Kit (ITK)




### 2.4) Les formats de données

Pour permettre l'intéraction entre différents logiciels, nous allons utiliser de nombreux formats de données.

* les formats propriétaires `Bruker`: les données sont stockés en binaire en 16 bit unsign pour les fid et 32 bit unsign pour les images reconstruites.

* le format `VTK`: c'est celui ci que nous allons principalement utiliser. Ils comprends plusieurs sous format:
  * `.vti` : le format `image data`, comprend un en-tête en htlm suivi des données stockées en binaire
  * `.vtk` : le format `DATASET STRUCTURED_POINTS`, comprend un en-tête en `ASCII` suivi des données stockées en binaire ou en `ASCII` (ancien format).
  *

* le format matlab

&nbsp;

## 3) La diffusion <a id="diffusion"></a>

Bibliographie à renseigner. En attendant, une définition à minima:

"L’IRM de diffusion est une technique basée sur l'imagerie par résonance magnétique (IRM). Elle permet de calculer en chaque point de l'image la distribution des directions de diffusion des molécules d'eau. Cette diffusion étant contrainte par les tissus environnants, cette modalité d'imagerie permet d'obtenir indirectement la position, l’orientation et l’anisotropie des structures fibreuses, notamment les faisceaux de matière blanche du cerveau". Source: [wikipedia](https://fr.wikipedia.org/wiki/IRM_de_diffusion).

&nbsp;

### 3.1) Localisation des algorithmes/programmes/logiciels.

> Plusieurs petits programmes ont été développées pour extraire, calculer et visualiser la structure cardiaque, ce protocole est susceptible d'évoluer et d'être affiné. En particulier la comparaison de multiples écchantillons nécessite quelques nouvelles fonctionnalités non développées à ce jour.  

En effectant ces commandes vous trouverez la liste des programmes en développement

{% highlight ruby %}
#localisation des programmes <a id="logiciels"></a>
cd /home/nelsonleouf/Dev/Vtk
ls
{% endhighlight %}

Nous nous interesserons particulièrement à ces derniers:

* `DT_fullC_beta_0.3`: programme de traitement données de diffusion.
* `Alignement`: programme pour orienter nos données.
* `MakeFigure` : programme facilitant la création de figures avec Paraview.

En effet, nous allons utiliser plusieurs logiciels de visualisation dont:

* `Volview` : c'est un logiciel très léger qui permet de visualiser les données en 3D. Téléchargeable [ici](http://www.kitware.com/opensource/vvdownload.html).
* `ITK-Snap`: c'est un logiciel de segmentation. Téléchargeable [ici](http://www.itksnap.org/pmwiki/pmwiki.php?n=Downloads.SNAP3).
* `Seg3D2`: c'est aussi un logiciel de segmentation. Téléchargeable [ici](https://github.com/SCIInstitute/Seg3D/releases).
* `Paraview`: C'est un logiciel de rendu 3D extrêmement au top. Téléchargeable [ici](http://www.paraview.org/download/).
* `ImageJ`: déjà bien connu.
* `Blender`: logiciel de rendu 3D, plus que extrêmement au top, la preuve en [images](https://www.blender.org/features/). Téléchargeable [ici](https://www.blender.org/).

NB: Ils sont tous déjà installés sur le pc de post-traitement de l'équipe imagerie.

&nbsp;


### 3.2) Extraction des données de diffusion

Se déplacer dans le dossier suivant pour accéder à l'executable `DT_fullC_beta_0.3`  

{% highlight ruby %}
#déplacement
cd /home/nelsonleouf/Dev/Vtk/DT_fullC_beta_0.3/build/
{% endhighlight %}

Afin d'extraire les données, plusieurs argument doivent être transmis au programme, ainsi que trois fichiers texte qui contient des paramètres.

Les arguments sont les suivants:

* argument 1 : lien vers votre espace de stockage
* argument 2 : nom du dossier, par ex. `coeur_2`
* argument 3 : numero d'acquisition, par ex. `35`
* argument 4 : mode `1`,`2`,`3`,`4` ou `0`, nous y reviendrons.

Les fichiers texte sont les suivants:

* `info.txt` : renseigne la taille et la résolution des images
* `threshold.txt` : renseigne les valeurs pouSer segmenter l'échantillon
* `axis.txt` : renseigne les coordonnées du "long axis".

Ces trois fichiers sont stockées dans la racine de chaque acquisition, par exemple `/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/30/`. Vous regarder s'il extsite en tapant la commande

{% highlight ruby %}
#ouverture du fichier info.txt s'il existe
gedit /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/30/info.txt &
{% endhighlight %}

 Nous commencerons par uniquement renseigner le fichier `info.txt` qui à la structure suivante:

* taille de la matrice suivant x
* taille de la matrice suivant y
* taille de la matrice suivant z
* champ de vue suivant x
* champ de vue suivant y
* champ de vue suivant z
* seuil FA min  (mettre 0)
* seuil FA max  (mettre 0)
* seuil Trace min (mettre 0)
* seuil Trace max (mettre 0)
* seuil DWI min (mettre 0)
* seuil DWI max (mettre 0)
* drapeau N4 (mettre 0)

Pour extaire ces informations, vous pouvez ouvrir le fichier suivant, le fichier `visu_par`. Ils sont spécifiques à la reconstruction.  

{% highlight ruby %}
#ouverture du fichier visu_par et methode de l'acquisition numero 30
gedit /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/30/2/visu_pars &


#ligne 20 et 21, vous obtenez la taille de la matrice pour les trois directions:
$VisuCoreSize=( 3 )
200 166 200

#ligne 20 et 21, vous obtenez la taille du champ de vue pour les trois directions:
$VisuCoreExtent=( 3 )
120 100 120

{% endhighlight %}


{% highlight ruby %}
#ouverture du fichier info.txt s'il existe
gedit /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart1/30/info.txt &
{% endhighlight %}

Et enfin nous lançons la commande

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/Vtk/DT_fullC_beta_0.3/build/
# extraction des données avec la commande à 4 arguments
./DT_fullC_beta_0.3 /media/nelsonleouf/sdb1/DICOM/Kadence/Control/ Heart_1/ 30 1
{% endhighlight %}

Plusieurs messages s'affichent, noter l'enregistrement de nombreux fichiers dans votre dossier `STDT/DT/` et en particulier dans le sous dossier `DT_PREPROCESSED_VTI`.
&nbsp;

### 3.2) Visualisation des données

Ouvrez un nouveau terminal et lancer le programme Volview.

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/VolView-3.4-Linux-x86_64/bin/
./Volview
{% endhighlight %}

Puis ouvrez le fichier `30_DT_04_diffusion_weighted_image.vti` correspondant à l'image pondérée en diffusion. `/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image.vti`.
Puis cliquer sur suivant plusieurs fois.

![image2](../../../../../images/image2.png)

Autre alternative, ouvrez un nouveau terminal et lancer le programme Itk-snap.

{% highlight ruby %}
# déplacement si necessaire
cd /home/choupinetleouf/Dev/itksnap-3.4.0-20151130-Linux-x86_64/bin/
./itksnap
{% endhighlight %}

![image3](../../../../../images/image3.png)
&nbsp;



### 3.3) Advanced Normalization Tools (ANTs) <a id="ants"></a>

Par la suite, nous allons utliser une librairie nommée `ANTs` pour Advanced Normalization Tools. `ANTs` est expliqué plus en détail [ici](http://stnava.github.io/ANTs/). Quelques remarques.  

* `ANTs` vient du monde de la neuro.
* `ANTS` est une librairie contenant des fonctions et un ensemble de scripts appelant ces fonctions. Il fonctionne exclusivement en ligne de commande, et permet un traitement complètement automatisé d'un large spectre de données d'imagerie.
* Idéalement `ANTs` devrait être utilisé pour traiter nos données.
* Nous l'utiliserons pour plusieurs aspects: la correction des inhomogénéités du champs, pour la registration et la création de template.

NB: `ANTs` est déjà installé sur le pc de post-traitement de l'équipe imagerie.

Sinon cloner ou télécharger le code [ici](https://github.com/stnava/ANTs.git), et on lance ces commandes:

{% highlight ruby %}
git clone git://github.com/stnava/ANTs.git
mkdir ants-build
cd ants-build
cmake ../ANTs
make -j 8
#puis attendez longtemps
{% endhighlight %}

Rendez-vous sur [ANTs](http://http://stnava.github.io/ANTs/) pour en savoir plus!

&nbsp;

### 3.4) Correction de biais (N4 ITK bias correcction)

Cette étape est relativement longue (pour l'ordinateur), pour pouvoir passer outre si ce calcul a été fait, un drapeau a été ajouté dans le fichier de configuration `info.txt` à la ligne treize. `1` code pour effectuer ce calcul, `0` code pour ne pas effectuer ce calcul. Dans un premier temps, nous choisirons d'activer ce calcul en mettant le drapeau à `1`.

{% highlight ruby %}
#ouverture du fichier info.txt et changement de la treizième ligne
gedit /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1/30/info.txt &
{% endhighlight %}


Nous lançons la commande suivante

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/Vtk/DT_fullC_beta_0.3/build/
# lancement de la commande à quatre argument avec le mode n°2
./DT_fullC_beta_0.3 /home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/ coeur_2/ 35 2
{% endhighlight %}

 Une fois ce calcul effectué, vous pouvez regarder avec le logiciel `Volview` les fichiers résultants et noter les différences en terme d'intensité. Pour cela ouvrez le logiciel `Volview` comme ceci:

{% highlight ruby %}
#ouvrer un terminal
cd
cd Dev/Volview/bin
./Volview
#en haut à gauche, cliquer sur menu, puis ouvrir et charger les fichiers suivants:
 /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image.vti
 /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image_cut_N4.vtk
{% endhighlight %}

![image4](../../../../../images/image4.png)

Vous pouvez maintenant désactiver la correction de biais N4 dans le fichier de configuration `info.txt` en mettant le drapeau à `0`.


Discussion scientifique : ceci est coupe short axis.... regarder les différences de contrastes avant et après application du filtre. Modification du contraste global mais pas de moficiation à l'échelle de la structure , vérifier la présence de fibre cardiaque visible à l'oeil nu. Noter par ailleurs la présence de graisse autour du ventricule gauche qu'il faudrait segmenter par la suite. La diffussion pas effective sur le tissu graisseux.




### 3.5) Segmentation <a id="segmentation"></a>


La segmentation peut-être effectuée plus ou moins finement. L'approche ici est la plus robuste trouvée pour segmenter des échantillons relativement large. Pour cela, l'échantillon est divisé en 3 segments noté apex, mid, et base selon l'axe principal du coeur. Si ce n'est pas le cas veuillez effectuer les rotations nécessaires.

La première étape est un seuillage sur les images pondérée en diffusion, la fraction d'anistropie, la trace.

Vous allez premièrement segmenter trois régions de l'échantillon de l'apex à la base. Pour cela nous utiliserons le logiciel Seg3D, nous ajouterons alors chaque seuil obtenu dans le fichier de configuration `threshold.txt` selon la procédure suivante.

{% highlight ruby %}
#ouvrer un terminal puis ouvrez le logiciel de segmentation Seg3D2
cd
cd Dev/Seg3D2/bin/
./Seg3D
#en haut à gauche, cliquer sur menu, puis ouvrir et charger le fichier suivant:

/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part1.vtk

{% endhighlight %}

Créer si nécessaire un nouveau projet que vous stocker dans votre répertoire personnel sur le réseau.

![image5](../../../../../images/image5.png)


L'écran est divisé en 4 panneaux, si ce n'est pas le cas cliquer sur `View` puis `Two And Two`. Vous pouvez maintenant observer votre échantillon et faire défiler les coupes, notons que nous nous situons à l'apex. L'eujeux est de seuiller le plus finement possible. Pour cela, cliquer dans le menu outil et sélectionner l'option threshold et ajouter des petites croix sur la zone que vous souhaitez conserver. Ajouter au tant de petits croix que nécessaire en particulier à l'extrémité de l'apex afin de conserver l'anatomie d'origine. Le contraste étant plus faible à cette extrémité, cette zone est facilement oubliée lors de la segmentation.

Aller maintenant dans la rubrique `Tools`, puis sélectionner l'option `Threshold`, une fenêtre s'ouvre sur la droite. Cliquer le bandeau `Clear Seeds` puis cliquer sur l'image sur le tissu que vous souhaitez selectionner. Une petite croix apparaît répéter cette opération antant que nécessaire. Vous devez obtenir un résultat similaire aux fenêtres suivantes.

![image6](../../../../../images/image6.png)

![image7](../../../../../images/image7.png)

Maintenant notez les deux valeurs (minimales et maximales, ici 0.14 et 0.83) situées dans la fenêtre `Upper` and `Lower` sur la première ligne du fichier de configuration `threshold.txt` selon cette nomenclature:

**FA_min_apex** **FA_max_apex** Trace_min_apex Trace_max_apex DWI_min_apex DWI_max_apex
FA_min_mid  FA_max_mid  Trace_min_mid  Trace_max_mid  DWI_min_mid  DWI_max_mid
FA_min_base FA_max_base Trace_min_base Trace_max_base DWI_min_base DWI_max_base

Puis passez à la région mid ventriculaire en ouvrant le fichier:

{% highlight ruby %}
/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part2.vtk
{% endhighlight %}

Utilisez l'option threshold et les petites croix pour segmenter la région mid-ventriculaire. N'hésitez pas enlever la graisse.

![image8](../../../../../images/image8.png)


Maintenant notez les valeurs minimales maximales sur la seconde ligne du fichier selon cette nomenclature:

FA_min_apexFA_max_apex Trace_min_apex Trace_max_apex DWI_min_apex DWI_max_apex
**FA_min_mid** **FA_max_mid**  Trace_min_mid  Trace_max_mid  DWI_min_mid  DWI_max_mid
FA_min_base FA_max_base Trace_min_base Trace_max_base DWI_min_base DWI_max_base

Répèter cette opération pour chaque région et chaque contraste afin de remplir les 24 valeurs, 12 minimales et 12 maximales en chargeant les fichiers suivants.

{% highlight ruby %}

/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part3.vtk

/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part1.vtk
/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part2.vtk
/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_01_fractional_anisotropy_gaussian_part3.vtk

/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image_part1.vtk
/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image_part2.vtk
/home/nelsonleouf/DICOM/Heart1/STDTdata/DT/DT_PREPROCESSED_VTI/30_DT_04_diffusion_weighted_image_part3.vtk

{% endhighlight %}


![image9](../../../../../images/image9.png)



Une fois ces étapes effectuées,  les valeurs de segmentation sont sauvegardées, vous obtiendrais un fichier similaire aux lignes ci-dessous. Si la segmentation n'est pas satisfaisante vous pouvez rejouer cette étape autant que nécessaire pour ajuster au mieux les seuillages.

0.23 0.72 0.53 1.14 228580000 794832000     
0.19 0.89 0.51 1.10 245244992 778752000     
0.20 0.88 0.54 1.06 311708000 966128000

Maintenant, relancer le programme `DT_fullC_beta_3` avec la commande suivante:

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/Vtk/DT_fullC_beta_0.3/build/
# lancement de la commande à quatre argument avec le mode n°2
./DT_fullC_beta_0.3 /home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/ Heart_1/ 30 2
{% endhighlight %}

![image10](../../../../../images/image10.png)

Noter la création de plusieurs masques que vous pouvez ouvrir avec Seg3D ou Volview pour vérifier la qualité de la segmentation:
{% highlight ruby %}

/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_3_layers_fractional.vtk

/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_3_layers_trace.vtk

/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_3_layers_dwi.vtk

/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_3_layers_combine.vtk

après utilisation d'un kernel [3,3,3] pour boucher les trous dans le mask
/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_threshold_3_layers.vtk

{% endhighlight %}






##### d) Quelques remarques sur cette étape.

 Les volumes que vous venez de charger ont préalablement été filtrés avec un filtre gaussien 3D de kernel [1 1 1] afin d'enlever le bruit présent dans l'image et d'homogénéiser le seuillage.

 Je rajoute cette phrase


### 3.6) Titre <a id="nomAncre"></a>

##### a) Définition de l'axe principal du coeur

Nous allons maintenant regarder notre segmentation et définir deux points par lesquel passe l'axe principal du coeur, ces points se situent à l'apex et dans la partie basale du coeur. Nous considérons pour cela uniquement le ventricule gauche Nous ajouterons alors les coordonnées correspondantes dans le fichier de configuration `axis.txt`.

Pour cela , nous chargeons dans Seg3D le fichier `/media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1//STDTdata/DT/MASK/30_DT_mask_threshold_3_layers_1px.vtk`.
Et nous utilisons le curseur, les coordonnées se mettent à jour directement en bas à droite

![image11](../../../../../images/image11.png)

nedit /media/nelsonleouf/sdb1/DICOM/Kadence/Control/Heart_1/30/info.txt &

Maintenant notez les valeurs de x,y,z du fichier selon cette nomenclature:

X_apex
Y_apex
Z_apex
X_base
Y_base
Z_base


##### b) Calcul des angles helix ...

{% highlight ruby %}
./DT_fullC_beta_0.3 /media/nelsonleouf/sdb1/DICOM/Kadence/Control/ Heart_1/ 30 3
{% endhighlight %}



##### c) Ouverture des angles dans Volview

![image12](../../../../../images/image12.png)


![image13](../../../../../images/image13.png)

##### d) Ouverture des vecteurs dans Paraview

{% highlight ruby %}
#ouvrer un terminal
cd
cd Dev/ParaView-4.2.0-Linux-64bit/bin
./paraview
#en haut à gauche, cliquer sur menu, puis ouvrir et charger les fichiers suivants:

{% endhighlight %}


![image14](../../../../../images/image14.png)

![image15](../../../../../images/image15.png)

![image16](../../../../../images/image16.png)

![image17](../../../../../images/image17.png)

![image18](../../../../../images/image18.png)

![image19](../../../../../images/image19.png)


Tutorials/BiventriclesFitting [lien](http://www.continuity.ucsd.edu/Continuity/Documentation/Tutorials/BiventriclesFitting)

##### e) Segmentation plus fine du coeur sur ITK-SNAP

Tutorials/Segmentation [lien](http://continuity.ucsd.edu/Continuity/Documentation/Tutorials/Segmentation)



### 9) Annexe <a id="annexe"></a>

#### 9.1) Abréviation <a id="abreviation"></a>

* ANTs : Advanced Normalization Tools
* DTI :
* DWI : diffusion weighted image
* FA  : fraction anasotropy
* ITK :
* STI :
* VTK :



#### 9.2) Le lexique <a id="lexique"></a>
