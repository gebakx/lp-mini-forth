# Pràctica LP: mini Forth

Aquesta pàgina descriu la pràctica de GEI-LP (edició 2025-2026 Q1). En aquesta pràctica has d'escriure un intèrpret d'una versió simplificada de Forth utilitzant Python i ANTLR. Utilitzareu el `doctest` de python per executar els jocs de probes.

Forth va ser desenvolupat per Charles H. Moore als anys 70. És un llenguatge funcional basat en pila. En aquesta pràctica haureu de treballar amb un subconjunt de Forth.

## mini Forth

A continuació tens un breu resum dels conceptes bàsics de Forth. Pots provar-lo amb l'intèrpret [Gforth](https://gforth.org/) i llegir més sobre el llenguatge a [Starting Forth](https://www.forth.com/starting-forth/1-forth-stacks-dictionary/). A més, aquesta secció també afita les construccions de Forth que caldrà implementar en aquesta pràctica. Has de dissenyar la gramàtica en ANTR per a que reconegui les diferents construccions que esmenta aquest document i el visitor per interpretar el codi. Els jocs de proves els gestionareu amb el *doctest* de python.

### Sintaxi bàsica

Forth utilitza una pila per avaluar tot:

```forth
1 2 3 .s  ( Comentari: el .s mostra la pila )
👉  [1, 2, 3]
```

En trobar números els anirà ficant en la pila d'avaluació; `.s` en mostra el seu contingut. Els comentaris es delimiten amb parèntesis.

`.` desempila un element i el mostra:

```forth
1 2 3 .
👉  3
```

Si la pila està buida donarà l'error pertinent:

```forth
1 . .
👉  
1
Error: pila buida!
```

#### Operacions aritmètiques

En aquesta pràctica treballarem només amb nombres enters. Els operadors aritmètics requerits són `+`, `-`, `*`, `/` i `mod`. 

```forth
5 2 .s - .s .
👉  
[5, 2]
[3]
3
```

```forth
7 2 mod .
👉  1
```

#### Manipulació de la pila

`swap`: intercanvia els dos elements superiors de la pila.

```forth
1 2 3 .s swap .s
👉  
[1, 2, 3]
[1, 3, 2]
```

`2swap`: intercanvia els dos parells d'elements superiors de la pila.

```forth
1 2 3 4 .s 2swap .s
👉  
[1, 2, 3, 4]
[3, 4, 1, 2]
```

`dup`: duplica el cim de la pila.

```forth
1 2 .s dup .s
👉  
[1, 2]
[1, 2, 2]
```

`2dup`: duplica els dos elements superiors de la pila.

```forth
1 2 3 .s 2dup .s
👉  
[1, 2, 3]
[1, 2, 3, 2, 3]
```

`over`: fa un *push* de l'element just per sota del cim.

```forth
1 2 .s over .s
👉  
[1, 2]
[1, 2, 1]
```

`2over`: fa un *push* de la parella just per sota de la del cim.

```forth
1 2 3 4 .s 2over .s
👉  
[1, 2, 3, 4]
[1, 2, 3, 4, 1, 2]
```

`rot`: mou el 2on element per sota del cim al cim.

```forth
1 2 3 .s rot .s
👉  
[1, 2, 3]
[2, 3, 1]
```

`drop`: esborra el cim de la pila.

```forth
1 2 .s drop .s
👉  
[1, 2]
[1]
```

`2drop`: esborra el parell del cim de la pila.

```forth
1 2 3 .s 2drop .s
👉  
[1, 2, 3]
[1]
```

#### Relacionals i booleans

Els operador relacionals són els habituals, except el diferent `<>`.
Els booleans es codifiquen com `0` (fals) i `-1` (cert) i conté els operadors booleans habituals: `and`, `or` i `not`.

```forth
2 3 <> -1 and .
👉  -1
```

#### Funcions

Les funcions utilitzen també la pila per la tranferència d'informació. Es delimiten per `:` i `;`:

```forth
: doble 2 * ;
3 doble .
👉  6
: f doble 1 + ;
3 f .
👉  7
```

#### Condicionals

Els condicionals es fan amb la sintaxi `condició if codi_cert else codi_fals endif` (on la part del *else* és opcional) i han de ser dins una funció:

```forth
: abs dup 0 < if 0 swap - endif ;
-2 abs .  👉  2

: min 2dup < if drop else swap drop endif ; 
3 2 min .  👉  2

: signe dup 0 < if drop -1 else 0 > if 1 else 0 endif endif ;
-2 signe .s  👉  [-1]
0 signe .s  👉  [0]
```

#### Recursivitat

La recursivitat es fa afegint la comanda `recurse`. A continuació teniu un exemple del seu ús:

```forth
: faux dup rot * swap 1 - dup 2 < if drop else recurse endif ;
: factorial dup 2 < if drop 1 else dup 1 - faux endif ;
4 factorial .  👉  24
0 factorial .  👉  1
```

### Jocs de proves

El *doctest* és un component python que ens permet gestionar jocs de proves. És un component molt útil per comprovar el funcionament de funcions. A continuació teniu un exemple que codifica un parell de probes:


```
>>> from forth import interpret

>>> interpret('2 3 + .')
5

>>> data = '\n'.join([
... ': doble 2 * ;',
... '3 doble .'
... ])

>>> interpret(data)
6
```

Si l'arxiu amb el contingut anterior té com a nom 'test.txt', haureu de cridar al doctest:

```bash
python3 -m doctest test.txt
```

Podeu afegir un parell de paràmetres que us seran especialment útils en el desenvolupament de la pràctica:
- `-v`: *verbose*.
- `-f`: que pari en trobar el primer error.

El vostre projecte ha d'incloure un arxiu en format *doctest* que demostri que el vostre intèrpret funciona correctament. La qualitat (però no la quantitat) dels jocs de proves serà un factor important en l'avaluació de la pràctica. 


## La vostra feina

Heu d'escriure un intèrpret de Forth utilitzant ANTLR i Python. Aquest intèrpret ha de ser capaç de llegir i avaluar expressions de Forth, així com definir i cridar funcions. Heu d'utilitzar ANTLR per escriure la gramàtica i els visitadors necessaris. Cal que la vostra implementació sigui compatible amb la sintaxi i les característiques de Forth descrites anteriorment.

El vostre programa s'ha de preparar amb un cop de `make`. Llavors, el vostre script principal ha de tenir una funció `interpret` que rebi com a paràmetre un string amb el codi a interpretar. Per exemple:

```bash
make antlr
python3 -i forth.py
>>> interpret('1 2 + .')
3
>>> quit()
make test
```

El vostre projecte ha de venir acompanyat d'un arxiu 'test.txt' en format doctest amb el vostre joc de proves.

### Llibreries

Utilitzeu `ANTLR` per escriure la gramàtica i l'intèrpret. Podeu utilitzar lliurament qualsevol llibreria **estàndard** de Python. No podeu usar cap altra llibreria no estàndard.

### Errors

Si el programa en Forth conté errors sintàctics, cal reportar-ho. 

En canvi, per senzilla, en aquesta pràctica, suposarem que no es dónen mai errors semàntics ni errors de tipus. En cas de donar-se, l'efecte del programa és indefinit.

Només s'han de tractar els errors d'execució de divisió per zero i de pila buida. En cas de donar-se algun altre, l'efecte del programa és indefinit.

Però la vostra pràctica no ha de petar en cap cas, totes les excepcions que es puguin produïr han de ser capturades.

## Lliurament

Heu de lliurar la vostra pràctica al Racó. Només heu de lliurar un fitxer ZIP que, al descomprimir-se generi:

- Un fitxer `README.md` que documenti el vostre projecte.
  
  - vegeu, per exemple, https://www.makeareadme.com/.

- Un fitxer `Makefile` tal que, quan s'executi `make`, es crein els fitxers necessaris per executar el vostre projecte.

- Un fitxer `forth.g4` amb la gramàtica del LP.

- Un fitxer `forth.py` amb el programa principal de l'intèrpret.

- Més fitxers `.py` amb les classes, visitadors i funcions auxiliars.

- Joc de proves en format doctest en el fitxer `test.txt`.

- Res més. 

Observacions:

- Els vostres fitxers de codi en Python han de seguir les regles d'estı́l PEP8, tot i que podeu oblidar les restriccions sobre la llargada màxima de les lı́nies. L'ús de tabuladors en el codi queda prohibit (zero directe).

- El termini de lliurament és el **dilluns 7 de gener a les 08:00**.

- Per evitar problemes de còpies, no pengeu el vostre projecte en repositoris públics.

- El vostre lliurament no ha d'incloure els fitxers que genera ANTLR, aquests s'han de crear via `make`.

- Si no heu realitzat alguna part de la pràctica, o sabeu que aquest té algun error en alguna part, deixeu-ho escrit al `README.md`.

## Avaluació

L'avaluació de la vostra pràctica tindrà en compte diversos aspectes clau, entre els quals es destaquen els següents:

1. **Qualitat de la gramàtica**: Es valorarà la qualitat de la gramàtica ANTLR, incloent la seva _completitud_, _precisió_, _concisió_ i _robustesa_. La gramàtica ha de ser capaç de reconèixer correctament els programes i les expressions de Mini Scheme, així com de manegar els diferents literals i operadors que es poden realitzar en aquest llenguatge. 

2. **Qualitat del codi**: S'examinarà el codi font tenint en compte diversos factors, com ara la _correctesa_, la _completitud_, la _llegibilitat_, l'_eficiència_, el _bon ús dels identificadors_, ètc. També es tindrà en compte la _bona estructuració_ del codi, és a dir, l'ús adequat de funcions, classes, mòduls i altres elements que afavoreixin la redacció, la comprensió i el manteniment del codi a llarg termini. Es valorarà negativament l'ús de funcions llargues o poc clares, funcions incomprensibles sense especificació, la presència de codi duplicat o innecessari, acoblament fort, variables globals o atributs de classe erronis, comentaris excessius, i altres pràctiques nocives. Per aquesta pràctica, l'eficiència és secundària, però no s'admetran disbarats.

3. **Qualitat de la documentació**: S'analitzarà la documentació del projecte, amb especial atenció a la seva _claredat_, _precisió_ i _completesa_, alhora que la seva _concisió_. La documentació hauria de descriure adequadament el funcionament del codi, les seves funcions i característiques principals, així com qualsevol altre aspecte rellevant que faciliti la comprensió i l'ús del projecte. La documentació també ha de deixar clares les decisions de disseny preses.

4. **Qualitat del joc de prova**: Es valorarà l'existència, la _cobertura_ i la _fiabilitat_ dels jocs de proves dissenyats per verificar el correcte funcionament del codi. Els jocs de proves han de ser suficientment amplis i variats per garantir que el codi respon de manera adequada a diferents situacions i casos d'ús. Tanmateix, els jocs de proves han de ser _limitats, _concisos_ i _eficaços_, evitant la redundància i la repetició innecessària. Alhora, el propòsit dels jocs de proves ha de ser _fàcil de comprendre_. Han de ser fàcils d'_executar_, i han de proporcionar una _sortida clara_ que permeti identificar ràpidament qualsevol problema o error.

En definitiva, és important recordar que es tracta d'un projecte de programació i, per tant, s'espera que el codi segueixi _bones pràctiques de programació_. 
