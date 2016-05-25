### Outils:
- Flex: `flex flex-doc`
- Bison: `bison bison-doc`
- Makefile: `make-doc`
- LLVM: `llvm-dev libncurses5-dev`
- Git: `git`
- Man: `manpages-dev`

- Ubuntu 14.04.4 i386 virtual image for VirtualBox and VMware
- proxy-free wifi

### Resources Utile:
    - Bison manual: `$ info bison`
    - Flex manual: `$ info flex`
    - C99 draf
    - Gnu C Manual
    - bc manual: `$ info bc -n "Basic Expressions"`
    - GCC internals manual et autres manuals en: https://gcc.gnu.org/onlinedocs/
    - Tutorials and docs on GCC Resource Center at IIT Bombay: http://www.cse.iitb.ac.in/grc/

Compilateurs & GCC introduction:
================================
Un compilateur est un outil  qui transforme un code source écrit dans un langage de programmation (le langage source) en un autre langage informatique (le langage cible).
C compilers:
    - top: MVCC, GCC (mingw, cygwin), Clang/LLVM
    - autres: Turbo/Borland, intel compiler, wactom compiler, TCC, PCC
### GCC overview:
    - multi archs: ~ 70 archs including (ARM, x86, ...) :
      de supercomputer -> pc -> phones -> embed systems -> firmware/bios
    - multi langs:
        - Principal: C, C++, obj-C, obj-C++, Fortan, Ada, Java, Go
        - autres : Pascal, Mercury, Modula-{2,3}, PL/1, D, VHDL
        - extensions : OpenMP, OpenACC
    - mutli platforms and OSs: tres grand nombre.
```
    .........................Front-End....................              .....Middle-End....              .......Back-End....
    +-------------+  +------------+  +-------------------+              +-------------------+            +-----------------+
    |  lexer /    |  | parser     |  | semantic          |              |  optimizer        |            |    obj code     |
    | scanner     |  |            |  | analyzer          |    ==>       |                   |    ===>    |    gen          |
    | (tokens)    |  | (AST)      |  | (check errors)    |              |                   |            |                 |
    +-------------+  +------------+  +-------------------+              +-------------------+            +-----------------+
        ( C )   ----> ( AST : GENERIC )                   ========>   ( GIMPLE ) ---> ( RTL )  ====>   ( ASM ) ---> ( obj code )

    |....Flex.....|  |...Bison....|  |...................................LLVM..............................................|
```
- autre IR output:
    - Standard Portable Intermediate Representation SPIR/SPIR-V: utilisé en OpenCl, OpenGL et Vulkan
    - LLVM Intermediate Representation: utilisé en LLVM
    - HSA Intermediate Layer: Utilisé en HSA specification
- version ancienne (4.3.1): contienne 2,029,115 lines in the main source et 1,546,826 lines in libraries. 57,825 fichiers + 52 configuration scripts + 163 Makefiles.
### lexical analyser overview:
- instruction:   i = 5 + 2 ;
        token    | token type
        -------- | -------------------
        foo      | VAR
        =        | assign operator
        5        | number
        +        | plus operator
        2        | number
        ;        | end of instruction

##### syntactic analyser:
- génère AST depuis definitions du grammaire
- erreurs de syntax détecté la
- `gcc -E file.c` to get pre-processed c code
##### semantic analyser:
- détecté les erreurs semantic:
    - erreurs de types
    - var non déclaré ou initialisé
    -
- different lexicer, syntactic (et la plus tard different semantic) analyser pour chaque lang

##### optimization:
- généralement maximiser la vitesse d'exécution et minimiser le taille objet code generé:
    - pre-calculer la valeur d'une équation constante
    - éliminer les block mort (condition d'entrer est toujours false
    - éliminer function et variable non utilisé
    - expressions et fonctions sans side-effect déplacé au dehors des boucles.
    - `y * 8` => `y << 3`
    - `y % 32` => `y & 31`
    - `for(i = 0; i < 10; i++) printf("%u\n", i*10);` => `for(i = 0; i < 100; i+= 10) printf("%u\n", i);`
- partie commun pour les plus tard des lang implementer
- optimizations des floats et doubles expressions dangerous because of
    precision limitation:
    - `x / 5.0` != `x * 0.2`
    - `(x + y) - y` != `x`
      ```c
      double x = 123454.034684121687525678523745234845234874;
      double y = 45234845234.874454563246798654679865467659848655489765;
      (x + y) - y != y;
      ```

- more:
    - Wikipedia: Program_optimization
    - http://www.pobox.com/~qed/optimize.html
##### generation du code:
- du presentation intermédiaire (AST ou autre implementation: SSA, IR, RTL) à code assembleur puis à code machine (obj code)
- different implementation pour chaque type processus

### Flex (lexical analyzer) & Bison (syntactic analyser)
- why:
    - très utilisés en implementation des lang: Ruby (YARV), PHP (Zend Parser), GCC, Go, Bash, PostgreSQL, MySQL, ...)
    - bien utilisés aussi en implementation des fichier configuration et mini-scripts
    - écrits en C (et utilisable en C, C++ et peut être Java)
    - free et open source (we love open source)
    - standardisés
    - facile à intégrer avec GCC et LLVM
    - très utilisé en materiel d'éducation en USA et Europe
- alternative:
    - Ragel: alternative du flex avec plus mieux support du windows, multi output lang, and graph visualization
    - Quex: alternative du Flex avec syntax similaire à Flex et plus mieux support à C++ et Unicode
    - Lemon: Bison alternative used on SQLite
    - JavaCC: pour implémentation en Java, exemple d'utilisation: Apache Derby, Apache Lucene, Vaadin
    - ANTLR: en Java aussi, exemple d'utilisation: Groovy, Jython, Hibernate, Twitter search engine, Apache Cassandra
    - Irony: pour C# et .net (IronyPython, Script.NET, ....)
- backend-end (et middle-end) à supporter:
    - on peut developer notre specific implementation (avec generation direct d'assembleur d'une platform specific ou d'une platform virtuelle)
    - sinon, une platform indépendante (avec un code prés d'assembleur, connu comme byte code ou intermédiaire representation) qui génère lui meme le code assembleur pour des platform reels et peut être aussi implement le middle-end (optimization). Cette platform est nommé low level virtuel machine:
      - LLVM: qui génère un code machine optimisé pour des dizaines d'arch (Nvidia et AMD pour leurs GPU, ...)
      - CLR: génère un code pour la platform .Net, (un peu specific pour windows et x86)
      - JVM: génère un code pour la platform Java, multi platform.

Flex:
=====
- intro: syntax and file struct:
    - %{ CODE C %}
    - REGEX DEFS et OPTIONS
    - %% patterns et actions qui match et return le type du token %%
    - CODE C
- output: yy.lex.c
- rules:
    - longest match
    - premier rule valid
- RegExp intro:  (*, +, ?, [], (), |, ., [[:digit:]], {N,M}, \s, \S, \w, \W, \d, \D, ^, $, [-], [^])
- Flex Global Variables(yytext, yyleng, .........)
- demos:
    - ID              [_a-zA-Z][_a-zA-Z0-9]*
    - number          [0-9]+
    - email
    - Decimal
    - Hex
    - opérateur relationnel (+, -, *, /, %)
Bison:
======
- file syntax et structure:
    - %{ CODE C %}        %code (top|requires|provide|) ( CODE C )
    - définir les token, les types, precedence et autres règles et options
    - %% définition du grammaire en syntax comme BNF %%
    - CODE C
- Bison Global Variables(yylval, yylloc, .........)
- output: <app>.tab.c <app>.tab.h
- BNF intro et syntax:
    - ???
- LALR parser intro
- example: syntax du lua et python
- demo:
    - simple calc: NUM ( OP NUM ) *
- reduce/reduce & shift/reduce conflicts (exemples et solutions)

**Workshop**: implementer bc basic, relational et Boolean Expression et precedence
**Workshop**: implementer p

AST:
===
- Implementation simple en C++
- `friend cout& operator<<(...)`
- pretty affiche de l'arbre

**Workshop**: implementer des parties de C99:
- control flow: if, for, while, do..while, switch
- functions

LLVM:
=====
- LLVM et Clang intro:
    - A l'origine, un remplacement pour le midlle-end (et partie du back-end) du GCC
    - a utilise le front-end du GCC
    - un front-end (Clang) a remplacé GCC front-end
    - architecture modulaire, moderne et intégrable
    - en train de developer sont assembleur et linker
    - le meilleur choie pour le développement du nouveau lang et compilateurs
- LLVM IR:
    - IR sont utilisé pour facilité l'analyse et l'optimisation du code et programs
    - chaque instruction a une simple et seul action bien définit (shift, add, move)
    - abstraction des registers, et nombre illimité
    - C lui meme est une IR pour les plus haut level lang
    - ByteCode (dans java et .Net) sont des IR
    - type SSA (Static Single Assignment): type safety, low-level operations
    - 2 type de variables: global (functs & global vars) `@` et local `%`
    - `;` pour les commentaires
    - types: iN (int 1 à 2^23-1 bits), iN*, [i x iN], [i x iN]*
	     fN (float), vN (vector), SN (stack), void, ...
	     halt (16bit float), float (32bit), double (64bit)
	     vector: `<i x iN>`
	     struct: {iN, iN,...}
    - functs: define TYPE @NAME (args...) { ret .. }
	      declare TYPE @NAME (args...)
    - instructions: high lavel assambly
      - add, div, or, and, ret, br, ...
      - icmp, switch, select, call
      - llvm.{memcpy,memmove,memset,sqrt,powi,ctlz,
    - pour avoir le code IR, utilise: `clang -S -emit-llvm file.c`
    - lli & llc
- program Hello World sample en C (utilisé `puts` au lieu de `printf`) et son IR output ( `clang -S -emit-llvm app.c` )
  ```c
  #include <stdio.h>
  int main() {
    puts("Hello Wolrd!");
  }
  ```
- étudier le syntax du IR (app.ll) et le modifier et l' interpreter (lli) et le compiler (llc)
- le réécrire en llvm-c
- le réécrire en llvm cpp
- ajoute LLVM output à notre AST `::Codegen()`
- Optimization passes

JIT:
===
- intro
- implementation avec LLVM
??????


### Related:
- language theory (+ new diff langs: scala, python, ada, lisp/clojure, prolog)
- programming patterns
- OS arch and dev
- system archi