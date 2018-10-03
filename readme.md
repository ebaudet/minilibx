# miniLibX

La **miniLibX** (ou **mlx** pour les intimes) est la librairie utilisée à [Epitech](http://www.epitech.eu) et [42](http://www.42.fr) pour les projets d’infographie: *fil de fer*, *raytracer*, ...

Ses caractéristiques principales sont l’aspect très simple des fonctions disponibles: ouvrir une fenêtre, afficher un pixel à l’écran, enregistrer un appel de touche (clavier ou souris). C’est une librairie basée sur l’environnement graphique [X.org](http://www.x.org/). Cet article est principalement là pour regrouper des ressources utiles sur la miniLibX, sur ses manuels et fichier associés. Les sources sont également fournies pour vous permettre de les installer chez vous et de faire tourner vos projets sur vos ordis persos.

## Les fichiers de la miniLibX

En dehors des sources de ce repos, on peut aussi retrouver la miniLibX ici :

### Archive des sources:
- minilibX originale de 2014 [mlx-2014-01-06.tgz](sources/mlx-2014-01-06.tgz)
- minilibX version native MacOS X (< El Capitan) [mlx-macosx-2015-03-10.tgz](sources/mlx-macosx-2015-03-10.tgz)
- minilibX pour MacOS X El Capitan [mlx-macosx-el-capitan-2015-11-05.tgz](sources/mlx-macosx-el-capitan-2015-11-05.tgz) 
- minilibX pour MacOS X Sierra [mlx-macosx-sierra-2016-10-17.tgz](sources/mlx-macosx-sierra-2016-10-17.tgz)

### Documentation
Toutes les fonctions et la documentation se trouve dans la version PDF des manuels (attention, la version native Mac OSX possède certaines différences, je vous invite à lire les headers):
- man 3 [mlx](manuels/mlx.pdf)
- man 3 [mlx_loop](manuels/mlx_loop.pdf)
- man 3 [mlx_new_image](manuels/mlx_new_image.pdf)
- man 3 [mlx_new_window](manuels/mlx_new_window.pdf)
- man 3 [mlx_pixel_put](manuels/mlx_pixel_put.pdf)

## Instalation avec XQuartz ou Xorg

Voici un tutoriel pour installer la miniLibX (sur un MacOS X ou tout autre distribution proche). Si vous voulez vous économiser du temps, il existe un package pour MacOS X qui vous fera ça automatiquement: [minilibx.pkg](http://files.achedeuzot.me/42/mlx/minilibx.pkg) (créé par [Mehdi Laouichi](https://fr.linkedin.com/pub/mehdi-laouichi/2a/9b1/49b)). **Attention**, il semblerait que cela ne fonctionne pas très bien voir plus du tout sur les dernières versions de MacOS X (> El Capitan). Je vous conseille de prendre les versions natives MacOS X pour vos projets.

Avant de commencer, n’oubliez pas d’installer le serveur [X.org](http://www.x.org/) si vous ne l’avez pas déjà. Sous MacOS X, le plus simple est d’installer [XQuartz](http://xquartz.macosforge.org/landing/) (miroir de [XQuartz 2.7.11](http://files.achedeuzot.me/42/mlx/XQuartz-2.7.11.dmg))

Il va nous falloir compiler la miniLibX puis copier les fichiers générés dans divers dossiers du système.

```bash
~ $ cd minilibx
~/minilibx $ make
~/minilibx $ cd test
~/minilibx/test $ ./mlx-test # Vous permet de savoir si la compilation de la lib a réussi
~/minilibx/test $ cd ..
```

Maintenant que la miniLibX est compilée et testée, nous allons l’installer sur le système, c’est à dire copier les fichiers vers des dossiers qui sont accessibles pour tout le monde.

```bash
~/minilibx $ sudo cp mlx.h /usr/X11/include
~/minilibx $ sudo cp libmlx.a /usr/X11/lib
~/minilibx $ sudo cp libmlx_intel-mac.a /usr/X11/lib
```

## Exemple
### Hello World

Comme d’habitude, le fameux Hello World (cf titre de la fenêtre) pour commencer:
```C
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <mlx.h>

typedef struct    data_s
{
    void          *mlx_ptr;
    void          *mlx_win;
}                 data_t;

int main(void)
{
    data_t        data;

    if ((data.mlx_ptr = mlx_init()) == NULL)
        return (EXIT_FAILURE);
    if ((data.mlx_win = mlx_new_window(data.mlx_ptr, 640, 480, "Hello world")) == NULL)
        return (EXIT_FAILURE);
    mlx_loop(data.mlx_ptr);
    return (EXIT_SUCCESS);
}
```

Attention, pour la compilation, il ya deux cas.

- Soit vous utilisez les « vieilles » versions utilisant X11 et il faut lancer ceci (attention à ne pas oublier les flags et attention à leur ordre):
```bash
~ $ gcc -I /usr/X11/include -g -L /usr/X11/lib -lX11 -lmlx -lXext <fichier(s) .c>
~ $ ./a.out
```
- Soit vous utilisez les nouvelles lib natives MacOS X et il faut ajouter les flags `-framework OpenGL -framework AppKit`  en plus. Vous remarquerez que même si cette libraire n’utilise plus X.org/X11, j’utilise toujours les chemins vers X11. C’est parce que lors de l’installation, nous avons copié la libmlx dans les dossiers X11 (souvenez-vous des sudo `cp ... /usr/X11/...`). Il faut donc ajouter les chemins de X11 pour la compilation. Si votre libmlx est ailleurs, pas besoin d’ajouter les chemins d’include vers X11.:
```bash
~ $ gcc -I /usr/X11/include -g -L /usr/X11/lib -l mlx -framework OpenGL -framework AppKit <fichier(s) .c>
~ $ ./a.out
```

Dans les deux cas, une fenêtre noir titre "Hello world" devrait s'ouvrir.

## Quelques petits pièges à éviter

Si le sujet ne précise pas la possibilité d’utiliser la fonction système `exit()`, c’est une erreur de sujet. En effet, il est impossible de quitter « proprement » la miniLibX, il faut forcément faire un `exit()` ou quitter *sauvagement* via le bouton « Fermer » de la fenêtre. D’anciennes légendes parlent d’une fonction `mlx_loop_stop()` mais elle n’est pas présente dans les sources fournies ici.

Lorsque vous initialisez la mlx, avec `mlx_init()`, attention aux environnements shell vides, c’est à dire, n’oubliez pas de protéger votre `mlx_init()` car si votre appel de fonction n’est pas correctement protégé, vous aurez un joli plantage en règle, allez tout droit en prison, sans passer par la case départ, ne touchez pas 20 000€. La commande pour tester cela vous même: `$ env -i <votre-executable>` S’il plante, c’est que vous ne l’avez pas bien protégé.

Si vous avez utilisé un paquet Debian ou une autre méthode d’installation, il est possible de devoir rajouter un dossier d’include pour la compilation: `-I /opt/X11/include/X11`




Pour plus d'info : <https://achedeuzot.me/2014/12/20/installer-la-minilibx/>