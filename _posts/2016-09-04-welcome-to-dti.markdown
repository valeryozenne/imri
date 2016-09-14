---
layout: post
title:  "Welcome to DTI!"
date:   2016-09-05 16:48:29 +0200
categories: jekyll update
---
Ceci est une page en construction.

### 1) Introduction

Bienvenue sur ce tutorial consacré au traitement des données Bruker de diffusion et de haute résolution pour l'analyse de la structure cardiaque.

![image1](../../../../../images/image1.png)

#### 1.1) Pré-requis

Aucun pré-requis n'est nécessaire, si toutefois ce tutorial est incomplet et/ou contient des approximations, n'hésitez pas à nous en faire part. Vous pouvez évidemment contribuer, et créer très rapidement votre page.

#### 1.2) Extraction des données de la console Bruker sous format DICOM

(à compléter) Nous allons utliser le logiciel Paravision de Bruker pour extraire les images au format DICOM.

#### 1.3) Copie des données de la console Bruker sous format Bruker

Lors de l'examen, les données Bruker sont stockés dans des dossiers numérotés par ordre d'acquisition des séquences. Par exemple, nous allons extraire le dossier numéro `35` du l'examen intitulé `2016-09-09-examen`.

Pour cela, ouvrir le menu puis aller dans le dossier ou ouvrir un terminal et taper la commande suivante:

{% highlight ruby %}
#se déplacer dans un répertoire
cd /opt/PV6/.../2016-09-09-examen

{% endhighlight %}

#### 1.4) Arborescence des fichiers Bruker

Pour chaque acquisition numérotée de 1 à N, nous retrouvons la même arborescence avec:

* un fichier texte `acqp` qui contient les paramètres d'acquisition.
* un fichier texte `method` qui contient d'autres paramètres d'acquisition.
* un fichier binaire `fid` qui contient les données brutes ie non reconstuite.
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

* la liste est aussi non exclusive, pour tout renseignement supplémentaire, ce [lien](http://imaging.mrc-cbu.cam.ac.uk/imaging/FormatBruker) est plus complet.

### 2) Traitement



#### 2.1) Création de l'arborescence de traitement

Le données sont copiées dans le répertoire `data` et sont classées par espèce, puis numérotées arbitrairement pour chaque échantillon, elles sont généralement classées par ordre d'acquisition.

* Espece_1
  * Coeur_1     (janvier 2016)
  * Coeur_2     (avril 2016)
  * Coeur_3     (mai 2016)
* Espece_2
  * Coeur_1  
    * 35        (données de diffusion)
    * 36        (données de résolution)         
  * Coeur_2

Dans chaque dossier nous retrouvons les données de diffusion notées et les données de haute résolution si les deux sont présentes.

Pour préserver les données acquises, le travail de post traitement est effectués dans un nouveau dossier nommé STDT. Il est généré en lancant le script suivant:

{% highlight ruby %}
cd /blabla/Espece_2/coeur_2/
#génération des sous dossier
sh createfolder.sh
{% endhighlight %}

Si ce fichier n'est pas présent vous pouvez facilement le générer en suivant ces commandes:

{% highlight ruby %}
#génération des sous dossier
cd /blabla/Espece_2/coeur_2/
echo ''  > createfolder.sh
echo ''  >> createfolder.sh
#et enfin
sh createfolder.sh

{% endhighlight %}


#### 2.1) Extraction des données











#### 2.2) Segmentation



#### 2.3) Segmentation

> Text that you put in blockquotes appears like this.

Rendez-vous sur [ANTs](http://http://stnava.github.io/ANTs/) pour tout apprendre à partir de zéro !
