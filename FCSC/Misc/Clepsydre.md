# Clepsydre - 200

## Description

`À l'origine, la clepsydre est un instrument à eau qui permet de définir la durée d'un évènement, la durée d'un discours par exemple. On contraint la durée de l’évènement au temps de vidage d'une cuve contenant de l'eau qui s'écoule par un petit orifice. Dans l'exemple du discours, l'orateur doit s'arrêter quand le récipient est vide. La durée visualisée par ce moyen est indépendante d'un débit régulier du liquide ; le récipient peut avoir n'importe quelle forme. L'instrument n'est donc pas une horloge hydraulique (Wikipedia).`

## Solution

On se connecte à l'adresse fournie. On nous demande alors de rentrer un mot de passe.

On y voit également une citation de Rabelais. Avec le nom du challenge, et sa description en plus, la thématique du temps est récurente.

![cl1](https://github.com/sklaer/sklaer.github.io/blob/master/Images/clepsydre_1.PNG)

Aucune autre indication n'est donnée. On essaie donc de rentrer un mot de passe pour voir :

![cl2](https://github.com/sklaer/sklaer.github.io/blob/master/Images/clepsydre_2.PNG)

Après plusieures tentatives infructueuses, on remarque un petit détail : si le mot de passe commence par le caractère **T**, la réponse ne vient pas aussi rapidement.

On vérifie :

![cl3](https://github.com/sklaer/sklaer.github.io/blob/master/Images/clepsydre_3.PNG)

Puisque c'est la piste la plus probante (pour ne pas dire la seule), autant la suivre. Après plusieurs essais, il s'avère que si l'on adjoint un **3** à notre **T** initial, alors la réponse met ~2s. Cela semble prometreur !

![cl4](https://github.com/sklaer/sklaer.github.io/blob/master/Images/clepsydre_4.PNG)

Visiblement, à chaque caractère, le temps de réponse augmente d'une seconde, il n'y a donc plus qu'a scripter la recherche (script naïf largement améliorable).

```python
def main():
    res = ""
    no_diff = False
    while not no_diff:
        no_diff = True
        for l in range(33, 127):
            cp = time.time()
            subprocess.run(["bash", "-c", "echo '{}{}' | nc -vvv challenges2.france-cybersecurity-challenge.fr 6006A".format(res, chr(l))])
            t = time.time() - cp
            if t >= len(res) + 1:
                print("--------BINGO-------\nGood for {} with {} !".format(chr(l), t))
                res += chr(l)
                no_diff = False
                break
    print("----FINI----\nres = {}".format(res))
```

On finit par obtenir le mot de passe : **T3mp#!**

![cl5](https://github.com/sklaer/sklaer.github.io/blob/master/Images/clepsydre_5.PNG)

[Retour à l'accueil.](https://sklaer.github.io/)
