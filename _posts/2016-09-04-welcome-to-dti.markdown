---
layout: post
title:  "Welcome to DTI!"
date:   2016-09-05 16:48:29 +0200
categories: jekyll update
---
> Ceci est une page en construction.

### 1) Introduction

Bienvenue sur ce tutorial consacré au traitement des données Bruker de diffusion et de haute résolution pour l'analyse de la structure cardiaque.

![image1](../../../../../images/image1.png)

#### 1.1) Pré-requis

Aucun pré-requis n'est nécessaire, si toutefois ce tutorial est incomplet et/ou contient des approximations, n'hésitez pas à nous en faire part. Vous pouvez évidemment contribuer, corriger, et/ou créer très rapidement votre page.

> Comme tout tutoriel, c'est parfois long et un peu fastidieux la première fois. Compter une journée de travail pour traiter votre premier jeu de données. Après quelques jours, 30 minutes suffisent pour tout effectuer.

#### 1.2) Extraction des données de la console Bruker sous format DICOM

(à compléter) Nous allons utliser le logiciel Paravision de Bruker pour extraire les images au format DICOM.

#### 1.3) Copie des données de la console Bruker sous format Bruker

Lors de l'examen, les données Bruker sont stockés dans des dossiers numérotés par ordre d'acquisition des séquences. Par exemple, nous allons extraire le dossier numéro `35` du l'examen intitulé `2016-09-09-examen`.

Pour cela, ouvrir le menu puis aller dans le dossier ou ouvrir un terminal et taper la commande suivante:

{% highlight ruby %}
#se déplacer dans un répertoire
cd /opt/PV6/.../2016-09-09-examen
#taper la commande suivante pour obtenir la liste des dossiers
ls
{% endhighlight %}

#### 1.4) Arborescence des fichiers Bruker

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

### 2) Quelques pré-requis organisationnels

> Avant de commencer quelques conseils. Les données peuvent être copiées sur votre espace personnel de stockage : smb://prenom.nom et utilisées sur le pc de post-traitement de l'équipe imagerie. Il est préférable de ne pas créer de doublons
de vos données car cela risque de prendre beaucoup d'espace.

#### 2.1) Création de l'arborescence des données IRM

Les données sont copiées par exemple dans le répertoire `data-bruker` et sont classées par espèce, puis numérotées arbitrairement pour chaque échantillon, elles sont généralement classées par ordre d'acquisition.

* Espece_1
  * Coeur_1     (janvier 2016)
  * Coeur_2     (avril 2016)
  * Coeur_3     (mai 2016)

Dans chaque dossier nous retrouvons les données de diffusion notées et les données de haute résolution si les deux sont présentes.

  * Espece_2
    * Coeur_1  
      * 35        (données de diffusion)
      * 36        (données de résolution)         
    * Coeur_2

#### 2.1) Création de l'arborescence des données de post-traitées

Pour préserver les données acquises de toute mauvaise manipulation, le travail de post-traitement est effectué dans un nouveau dossier nommé STDT. Il est généré en lancant le script suivant:

{% highlight ruby %}
cd /home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/coeur_2/
#génération des sous dossier
sh createfolder.sh
{% endhighlight %}

Si ce fichier n'est pas présent vous pouvez facilement le générer en suivant ces commandes:

{% highlight ruby %}
#génération des sous dossier
cd /home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/coeur_2/
echo ''  > createfolder.sh
echo ''  >> createfolder.sh
#et enfin
sh createfolder.sh

{% endhighlight %}

Ainsi nous disposons de l'arborescence suivante:

* Espece_2
  * Coeur_1  
    * 35        (données de diffusion)
    * 36        (données de résolution)  
    * STDT      (dossier de post-traitement)
      * ST      
      * DT
      * Stat      

### 3) La diffusion

Bibliographie à renseigner. En attendant, une définition à minima:

"L’IRM de diffusion est une technique basée sur l'imagerie par résonance magnétique (IRM). Elle permet de calculer en chaque point de l'image la distribution des directions de diffusion des molécules d'eau. Cette diffusion étant contrainte par les tissus environnants, cette modalité d'imagerie permet d'obtenir indirectement la position, l’orientation et l’anisotropie des structures fibreuses, notamment les faisceaux de matière blanche du cerveau". Source: [wikipedia](https://fr.wikipedia.org/wiki/IRM_de_diffusion).


#### 3.1) Localisation des algorithmes/programmes/logiciels.

> Plusieurs petits programmes ont été développées pour extraire, calculer et visualiser la structure cardiaque, ce protocole est susceptible d'évoluer et d'être affiné. En particulier la comparaison de multiples écchantillons nécessite quelques nouvelles fonctionnalités non développées à ce jour.  

En effectant ces commandes vous trouverez la liste des programmes en développement

{% highlight ruby %}
#localisation des programmes
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

#### 3.1) Extraction des données de diffusion

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
* `threshold.txt` : renseigne les valeurs pour segmenter l'échantillon
* `axis.txt` : renseigne les coordonnées du "long axis".

Nous commencerons par uniquement renseigner le fichier `info.txt` qui à la structure suivante:

* taille de la matrice suivant x
* taille de la matrice suivant y
* taille de la matrice suivant z
* champ de vue suivant x
* champ de vue suivant y
* champ de vue suivant z

Et enfin nous lançons la commande

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/Vtk/DT_fullC_beta_0.3/build/
# extraction des données avec la commande à 4 arguments
./DT_fullC_beta_0.3 /home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/ coeur_2/ 35 1
{% endhighlight %}

Plusieurs messages s'affichent, noter l'enregistrement de nombreux fichiers dans votre dossier `STDT/DT/` et en particulier dans le sous dossier `DT_PREPROCESSED_VTI`.

#### 3.2) Visualisation des données

Ouvrez un nouveau terminal et lancer le programme Volview.

{% highlight ruby %}
# déplacement si necessaire
cd /home/nelsonleouf/Dev/VolView-3.4-Linux-x86_64/bin/
./Volview
{% endhighlight %}

Puis ouvrer le fichier `/home/nelsonleouf/Reseau/votreprenom/data-bruker/Espece_2/coeur_2/35/STDT/DT/DT_PREPROCESSED_VTI/139_DT_03_intensity.vti`

![image2](../../../../../images/image2.png)

Autre alternative, ouvrez un nouveau terminal et lancer le programme Itk-snap.

{% highlight ruby %}
# déplacement si necessaire
cd /home/choupinetleouf/Dev/itksnap-3.4.0-20151130-Linux-x86_64/bin/
./itksnap
{% endhighlight %}

![image3](../../../../../images/image3.png)


#### 3.3) Segmentation

La segmentation peut-être effectuée plus ou moins finement. L'approche ici est la plus robuste trouvée pour segmenter des échantillons relativement large.

Rendez-vous sur [ANTs](http://http://stnava.github.io/ANTs/) pour tout apprendre à partir de zéro !
