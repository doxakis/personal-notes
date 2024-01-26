# Falsehood programmers believe in - General

On découvre le monde d'une manière différente. Que cela soit par la langue, la culture, les règles qui régissent la société où l'on vit, la scolarité, l'entourage, les politiques mise en place par les politiciens, les voyages, etc. À force de retrouver certains patterns communs à répétition avec le temps, nous généralisons parfois à tord certains concepts. Cela peut être source de bugs en programmation. Voyons quelques exemples..

## L'identité d'un individu
- Le nom d'un usager peut changer suite à un mariage.
- Pour le prénom, il peut y avoir un changement légal.
- Un email peut être présent plusieurs fois si l'on a une table multi-tenant.

## Le temps
Un timestamp ne peut pas être considéré unique, comme par exemple dans des logs (i.e. souvent des problèmes d'ordonnancement quand il y a de nombreux logs quasi-simultanés)

Représenter un timestamp (unix time) dans un entier signé (32bit) va poser problème au courant de l'année 2038. Le débordement sur le bit de signe va causer l'affichage d'une date dans le passé et peut causer des problèmes variés selon l'app et ce qui est fait avec la date. (ref: https://en.wikipedia.org/wiki/Year_2038_problem)

Il existe de nombreux timezones. Certain pays ont une heure d'été. La différence par rapport au temps UTC n'est pas toujours exprimé en heures. Il peut y avoir aussi un décalage de minutes (ex: 30 ou 45 minutes).

## Adresse

Le numéro civique d'une adresse peut contenir plus qu'un chiffre. C'est possible d'avoir des lettres et même des fractions.
(ref: https://www.canadapost-postescanada.ca/scp/fr/soutien/sujet/directives-adressage/adresse-municipale.page#suffixe-du-numero-municipal)

## Internationalization

- La longueur d'un texte traduit varie d'une langue à l'autre et peut même doubler par rapport à l'anglais pour certaines langues
- Il existe différentes règles pour le pluriel et même plusieurs variantes selon le nombre et la langue.

### Internationalisation vs localisation

Adapter un logiciel pour d'autres pays se n'est pas seulement de traduire les textes, d'adapter le format des dates et la monnaie utilisée.

> L'internationalisation est le processus de conception d'une application logicielle afin de l'adapter à différentes langues et régions sans modifications techniques. La localisation est le processus d'adaptation d'un logiciel internationalisé pour une région ou une langue spécifique en traduisant le texte et en ajoutant des composants spécifiques aux paramètres régionaux.
(ref: https://fr.wikipedia.org/wiki/Internationalisation_et_localisation)

Prenons l'exemple de l'hébreu. Tenir compte de la localisation en développement logiciel, ça serait non seulement de traduire les textes mais également d'adapter l'interface pour que l'alignement général du texte et des boutons soit à droite. D'ailleurs, même déplacer le curseur via les touches gauche/droite du clavier dans un texte en hébreu se fait dans le sens inverse. Si l'on aurait des boutons avec un icone et du texte à droite, il faudrait les inverser.

Aperçu de Windows en hébreu :

- L'heure se trouve à gauche.
- Le menu démarrer est à droite.
- Les boutons fermer/agrandir/réduire d'une fenêtre sont inversé.

![Windows en hébreu](https://ia802905.us.archive.org/33/items/WinVistaSP2AIOHebrew/Windows_Vista_Desktop.png)

Le site web Wikipedia est un autre bon exemple. ref: https://he.wikipedia.org/

## Networking
- Il n'est pas impossible qu'il y ait plus d'une machine avec la même adresse IP.
- Un appel TCP vers un API ne va pas toujours répondre avec un succès ou une erreur (Il peut y avoir des timeouts ou un proxy qui drop la connexion)

### Distributed computing

Voici plusieurs énoncés qui ne sont pas forcément vrai:

- The network is reliable;
- Latency is zero;
- Bandwidth is infinite;
- The network is secure;
- Topology doesn't change;
- There is one administrator;
- Transport cost is zero;
- The network is homogeneous.

(ref: https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)

Si vous désirez en savoir plus, je suggère de consulter https://github.com/kdeldycke/awesome-falsehood et https://github.com/minimaxir/big-list-of-naughty-strings qui rassemble une liste de chaîne de caractères pouvant généralement poser problème.
