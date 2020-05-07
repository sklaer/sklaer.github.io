# Chapardeur de mots de passe - 200

-   [Retour à l'accueil.](../../index.md)

## Description

```
Un ami vous demande de l'aide pour déterminer si l'email qu'il vient d'ouvrir au sujet du Covid-19 était malveillant et si c'était le cas, ce qu'il risque.
Il prétend avoir essayé d'ouvrir le fichier joint à cet mail sans y parvenir. Peu de temps après, une fenêtre liée à l'anti-virus a indiqué, entre autre, le mot KPOT v2.0 mais rien d'apparent n'est arrivé en dehors de cela.
Après une analyse préliminaire, votre ami vous informe qu'il est probable que ce malware ait été légèrement modifié, étant donné que le contenu potentiellement exfiltré (des parties du format de texte et de fichier avant chiffrement) ne semble plus prédictible. Il vous recommande donc de chercher d'autres éléments pour parvenir à l'aider.
Vous disposez d'une capture réseau de son trafic pour l'aider à déterminer si des données ont bien été volées et lui dire s'il doit rapidement changer ses mots de passe !
```

## Solution

On nous fournit le fichier de capture au format pcap.

Un détail intéressant à retenir dans l'énoncé : `KPOT v2.0`.

On trouve tous les renseignement nécéssaires [ici](https://www.proofpoint.com/us/threat-insight/post/new-kpot-v20-stealer-brings-zero-persistence-and-memory-features-silently-steal).

On y apprend notamment que :

-   Le malware utilise le protocole HTTP pour communiquer avec le serveur C2.
-   Il émet deux requêtes (GET puis POST) sur `/<random>/gates.php`.
-   La réponse du serveur C2 à la requête GET est encodé en base64 puis XORé.
-   Idem pour la requête POST. Le XOR est effectué à l'aide de la même clé.
-   La clé est probablement de taille 16.

On trouve même une réponse type à la requête GET :

```
1111111111111100__DELIMM__A.B.C.D__DELIMM__appdata__GRABBER__*.log,*.txt,__GRABBER__%appdata%__GRABBER__0__GRABBER__1024__DELIMM__desktop_txt__GRABBER__*.txt,__GRABBER__%userprofile%\Desktop__GRABBER__0__GRABBER__150__DELIMM____DELIMM____DELIMM__
```

Ce que nous avons à faire apparaît alors clairement :

-   Identifier les requêtes et récupérer leurs contenus.
-   Déterminer la clé permettant de décrypter lesdits contenus.

On trouve plusieurs résultats, mais la requête GET sur l'ip _203.0.113.42_ semble la plus prometteuse car elle est suivie d'une requête POST sur la même adresse ip.

![chap](https://github.com/sklaer/sklaer.github.io/blob/master/Images/chapardeur_1.PNG)

On exporte donc le contenu des deux requêtes.

On sait à quoi doit ressembler la requête GET une fois déchiffrée, et notamment qu'elle contiendra une suite de 16 _0_ ou _1_ suivi de la chaine `__DELIMM__`.

On utilise ces informations pour retrouver la clé :

```python
def xor(a, b):
    return ''.join(chr(ac ^ ord(bc)) for ac, bc in zip(a, b))

def findkey():
    with open("./gate_get.txt") as file:
        enc = b.b64decode(file.read())[16:27]
    motif = "__DELIMM__"
    key = ""
    while len(key) <= len(motif) - 1:
        for k in range (0, 256):
            if enc[len(key)] ^ k == ord(motif[len(key)]):
                key += chr(k)
                break
    key += "\x00"*(16-len(key))
    print(key)
    return key

def decrypt(key):
    with open("./gate_get.txt") as file:
        res = b.b64decode(file.read())
    print(xor(res, key*34))
```

Ce qui nous retourne :

```
$ python3 chapardeur.py
tDlsdL5dv2
0110101110RcYF__DELIMM__R       |YF8.149.373_j't!;M__appdataj<v)4BER__*.logI&,__GRABBERj<3data%__GRAw!t7)0__GRABBERj<bZB__DELIMM__QB9p_txt__GRAw!t7)*.txt,__GRt!s:)_%userprof\Tw42esktop__GRt!s:)_0__GRABBEg<nb7)DELIMM____q&}____DELIMMj<
```

C'est concluant, on retrouve bien notre chaine `__DELIMM__` comme prévue, mais on remarque également que les 10 premiers carcatères sont des _0_ ou des _1_, ainsi que la présence d'autres `__DELIMM__` et `__GRABBER`.

On peut en conclure que l'on dispose bien des 10 premiers caractères de la clé.

En analysant la chaine décryptée, on peut déjà retrouver les deux derniers caractères de notre clé :

`__GRABBERj<` doit devenir `__GRABBER__`

La clé devient donc `tDlsdL5dv25c\x00\x00\x00\x00` et on obtient cette fois :

```
$ python3 chapardeur.py
011010111011cYF__DELIMM__21     |YF8.149.373__Dt!;M__appdata__v)4BER__*.log,*&,__GRABBER__3data%__GRABBt7)0__GRABBER__bZB__DELIMM__deB9p_txt__GRABBt7)*.txt,__GRABs:)_%userprofilTw42esktop__GRABs:)_0__GRABBER_nb7)DELIMM____DE}____DELIMM__
```

On répète encore ce procédé jusqu'à finalement trouver la clé : `tDlsdL5dv25c1Rhv`

Avec celle-ci on obtient finalement :

```
0110101110111110__DELIMM__218.108.149.373__DELIMM__appdata__GRABBER__*.log,*.txt,__GRABBER__%appdata%__GRABBER__0__GRABBER__1024__DELIMM__desktop_txt__GRABBER__*.txt,__GRABBER__%userprofile%\Desktop__GRABBER__0__GRABBER__0__DELIMM____DELIMM____DELIMM__


_DRAPEAU_P|us2peurQue2M4l!  R4ssur3z-Votre-Am1-Et-vo1c1Votredr4peau_FCSC
{469e8168718996ec83a92acd6fe6b9c03c6ced2a3a7e7a2089b534baae97a7}
_DRAPEAU_
```

-   [Retour à l'accueil.](../../index.md)
