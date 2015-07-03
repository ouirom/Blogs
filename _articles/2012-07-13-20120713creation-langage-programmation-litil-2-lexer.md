---
ID: 472
post_title: 'Le conte de Litil &#8211; Chapitre 2, Le dépeceur du texte, aka Lexer'
author: jmoussa
post_date: 2012-07-13 09:25:00
post_excerpt: "<p>Dans ce deuxième post du conte de Litil, je vais parler de la phase de lexing. C'est généralement la première étape dans la construction d'un compilateur (ou évaluateur) d'un langage donné. Cette phase sert à transformer le texte du code source (séquence de caractères) vers une séquence de <code>tokens</code>, qui seront consommés par le parseur à l'étape suivante.</p>"
layout: post
permalink: http://blog.zenika-offres.com/?p=472
published: true
slide_template:
  - ""
---
<p>Dans ce deuxième post du conte de Litil, je vais parler de la phase de lexing. C'est généralement la première étape dans la construction d'un compilateur (ou évaluateur) d'un langage donné. Cette phase sert à transformer le texte du code source (séquence de caractères) vers une séquence de <code>tokens</code>, qui seront consommés par le parseur à l'étape suivante.</p>
<!--more-->
<p>Les règles suivant lesquelles le texte est découpé en tokens varient d'un langage à un autre, mais en général (avec les langages conventionnels de la famille Algol par exemple), on peut définir les familles de tokens suivants:</p> <ul> <li>symboles: les opérateurs (<code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>, etc.), les signes de ponctuation (<code>,</code>, <code>;</code>, <code>(</code>, <code>[</code>, etc.).</li> <li>les littéraux: nombres (<code>42</code>, <code>3.14</code>, etc.), chaînes de caractères (<code>"kthxbye"</code>), booléens (<code>true</code>, <code>false</code>).</li> <li>les mots clés: <code>if</code>, <code>while</code>, etc.</li> <li>les identifiants: cela dépend du langage, mais généralement un identifiant commence par une lettre ou un tiret-bas (<code>_</code>), puis optionnellement une séquence de lettres, chiffres et quelques autres symboles.</li> </ul> <p>On peut donc voir un token comme un couple <strong>type</strong> (symbole, mot clé, littéral, etc.) et <strong>valeur</strong> (le contenu textuel de ce token). Optionnellement, on peut enrichir un token pour contenir aussi le numéro de la ligne et de la colonne où il apparait dans le texte source, ce qui peut s'avérer utile pour signaler l'emplacement d'une erreur.</p> <p>Enfin, le lexer se charge de cacher ou ignorer le contenu inutile dans le fichier source, comme les blancs, retours à la ligne et autres.</p> <h4>Gestion des blancs</h4> <p>Dans le langage que nous allons implémenter, et à l'encontre de la majorité des autres langages, les blancs sont importants car il servent à démarquer le début et la fin des blocs comme dans Python (et Haskell) par exemple. Le début d'un bloc est signalé par une augmentation du niveau de l'indentation tandis que sa fin est marquée par une dé-indentation.</p> <p>Exemple:</p> <pre class="ocaml code ocaml" style="font-family:inherit"><span style="color: #06c; font-weight: bold;">if</span> n <span style="color: #a52a2a;">&gt;</span> 0 <span style="color: #06c; font-weight: bold;">then</span>   <span style="color: #06c; font-weight: bold;">let</span> x <span style="color: #a52a2a;">=</span> <span style="color: #c6c;">10</span> <span style="color: #a52a2a;">/</span> n   print <span style="color: #3cb371;">&quot;Ohai&quot;</span> <span style="color: #06c; font-weight: bold;">else</span>   throw Error</pre> <p>L'équivalent Java serait:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055;font-weight: bold;">if</span> <span style="color: #000000;">&#40;</span>n <span style="color: #000000;">&gt;</span> 0<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span>   <span style="color: #7F0055; font-weight: bold;">int</span> x = <span style="color: #cc66cc;">10</span> / n;   <span style="color: #000000;">System</span>.<span style="color: #000000;">out</span>.<span style="color: #000000;">print</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Ohai&quot;</span><span style="color: #000000;">&#41;</span>; <span style="color: #000000;">&#125;</span> <span style="color: #7F0055;font-weight: bold;">else</span> <span style="color: #000000;">&#123;</span>   <span style="color: #7F0055; font-weight: bold;">throw</span> <span style="color: #7F0055; font-weight: bold;">new</span> <span style="color: #000000;">Exception</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; <span style="color: #000000;">&#125;</span></pre> <p>Bien que les corps des branches <code>then</code> et <code>else</code> du code java sont bien indentés, cette indentation est optionnelle et est carrément ignorée par le lexer. Ce sont les accolades ouvrantes et fermantes qui démarquent le début et la fin d'un bloc. On aurait pu obtenir le même résultat avec:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055;font-weight: bold;">if</span> <span style="color: #000000;">&#40;</span>n <span style="color: #000000;">&gt;</span> 0<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> <span style="color: #7F0055; font-weight: bold;">int</span> x = <span style="color: #cc66cc;">10</span> / n; <span style="color: #000000;">System</span>.<span style="color: #000000;">out</span>.<span style="color: #000000;">print</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Ohai&quot;</span><span style="color: #000000;">&#41;</span>; <span style="color: #000000;">&#125;</span> <span style="color: #7F0055;font-weight: bold;">else</span> <span style="color: #000000;">&#123;</span> <span style="color: #7F0055; font-weight: bold;">throw</span> <span style="color: #7F0055; font-weight: bold;">new</span> <span style="color: #000000;">Exception</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; <span style="color: #000000;">&#125;</span></pre> <p>ou encore:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055;font-weight: bold;">if</span> <span style="color: #000000;">&#40;</span>n <span style="color: #000000;">&gt;</span> 0<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span><span style="color: #7F0055; font-weight: bold;">int</span> x = <span style="color: #cc66cc;">10</span> / n;System.<span style="color: #000000;">out</span>.<span style="color: #000000;">print</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Ohai&quot;</span><span style="color: #000000;">&#41;</span>;<span style="color: #000000;">&#125;</span> <span style="color: #7F0055;font-weight: bold;">else</span> <span style="color: #000000;">&#123;</span><span style="color: #7F0055; font-weight: bold;">throw</span> <span style="color: #7F0055; font-weight: bold;">new</span> <span style="color: #000000;">Exception</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;<span style="color: #000000;">&#125;</span></pre> <p>La lisibilité du code en souffre, mais cela ne change rien du point de vue du compilateur. Ce n'est pas le cas avec Litil, où comme dit plus haut, l'indentation du code n'est pas optionnelle car elle sert à définir sa structure. De plus, là où dans Java on utilisait le <code>;</code> pour séparer les instructions d'un même bloc, Litil utilise plutôt les retours à la ligne. Les <code>;</code> ne sont pas optionnels, ils ne sont pas reconnues. Mon but était de s'inspirer des langages comme Haskell et Python pour créer une syntaxe épurée avec un minimum de bruit et de décorum autour du code utile. Je reviendrai là dessus dans le<em>(s)</em> post<em>(s)</em> à venir quand je vais détailler la syntaxe de Litil, mais pour vous donner quelques exemples:</p> <ul> <li>Pas d'accolades pour délimiter les blocs</li> <li>Pas de <code>;</code> pour séparer les instructions</li> <li>Pas de parenthèse pour les arguments d'une fonction: <code>sin 5</code></li> <li>Pas de parenthèse pour les conditions des <code>if</code></li> <li>etc.</li> </ul> <p>Donc, pour résumer, le lexer que nous allons développer ne va pas complètement ignorer les blancs. Plus précisément, le lexer devra produire des tokens pour signaler les retours à la ligne (pour indiquer la fin d'une instruction) et les espaces (ou leur absence) en début de lignes (pour indiquer le début ou la fin d'un bloc).</p> <h4>Implémentation</h4> <p>Pour commencer, voici la définition d'un Token:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">class</span> Token <span style="color: #000000;">&#123;</span>     <span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">enum</span> Type <span style="color: #000000;">&#123;</span>         NEWLINE, INDENT, DEINDENT, NAME, NUM, STRING, CHAR, SYM, BOOL, KEYWORD, EOF     <span style="color: #000000;">&#125;</span>     <span style="color: #7F0055; font-weight: bold;">publi
c</span> <span style="color: #7F0055; font-weight: bold;">final</span> Type type;     <span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">final</span> <span style="color: #000000;">String</span> text;     <span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">final</span> <span style="color: #7F0055; font-weight: bold;">int</span> row, col; &nbsp;     <span style="color: #7F0055; font-weight: bold;">public</span> Token<span style="color: #000000;">&#40;</span>Type type, <span style="color: #000000;">String</span> text, <span style="color: #7F0055; font-weight: bold;">int</span> row, <span style="color: #7F0055; font-weight: bold;">int</span> col<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span>         <span style="color: #7F0055; font-weight: bold;">this</span>.<span style="color: #000000;">type</span> = type;         <span style="color: #7F0055; font-weight: bold;">this</span>.<span style="color: #000000;">text</span> = text;         <span style="color: #7F0055; font-weight: bold;">this</span>.<span style="color: #000000;">row</span> = row;         <span style="color: #7F0055; font-weight: bold;">this</span>.<span style="color: #000000;">col</span> = col;     <span style="color: #000000;">&#125;</span> <span style="color: #000000;">&#125;</span></pre> <p>Un token est composé de:</p> <ul> <li><code>type</code>: le type du token: <ul> <li><code>NEWLINE</code>: pour indiquer un retour à la ligne</li> <li><code>INDENT</code>: pour indiquer que le niveau d'indentation a augmenté par rapport à la ligne précédente, et donc qu'on entre dans un nouveau bloc</li> <li><code>DEINDENT</code>: pour indiquer que le niveau d'indentation a diminué par rapport à a ligne précédente, et donc qu'on sort d'un bloc</li> <li><code>NAME</code>: une clé pour indiquer qu'il s'agit d'un identifiant</li> <li><code>NUM</code>, <code>STRING</code>, <code>CHAR</code>, <code>BOOL</code>: une clé pour indiquer qu'il s'agit d'un littéral</li> <li><code>KEYWORD</code>: une clé pour indiquer qu'il s'agit d'un mot clé</li> <li><code>EOF</code>: produit quand on a atteint la fin du texte source</li> </ul></li> <li><code>text</code>: une chaîne qui contient le texte correspondant à ce token</li> <li><code>row</code> et <code>col</code>: indique la position du token dans le texte source</li> </ul> <p>Voici maintenant l'interface qui décrit le lexer:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">interface</span> Lexer <span style="color: #000000;">&#123;</span>     Token pop<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span> <span style="color: #7F0055; font-weight: bold;">throws</span> LexingException; &nbsp;     <span style="color: #000000;">String</span> getCurrentLine<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; <span style="color: #000000;">&#125;</span></pre> <p>Cette interface définit les 2 méthodes suivantes:</p> <ul> <li><code>pop</code>: retourne le token suivant</li> <li><code>getCurrentLine</code>: retourne la ligne courante dans le texte source</li> </ul> <p>Notez l'absence d'une méthode qui indique la présence ou pas d'un token suivant. En effet, quand le lexer atteint la fin du fichier, il se place en un mode où tous les appels à <code>pop</code> retournent un token de type <code>EOF</code>. J'ai donc estimé inutile d'alourdir l'interface pour ajouter une méthode à la <code>hasNext</code> d'un itérateur par exemple.</p> <p>Notez aussi que si j'ai défini une interface pour le lexer, c'est parce qu'il y aurait plusieurs implémentations que nous allons voir par la suite.</p> <h5>Comment ça fonctionne</h5> <p>Voici une présentation rapide du fonctionnement de l'implémentation de base du lexer (<a href="https://github.com/jawher/litil/blob/master/src/main/java/litil/lexer/BaseLexer.java">Code source de BaseLexer.java sur Github</a>) pour ceux qui voudront suivre avec le code sous les yeux:</p> <ul> <li>Le constructeur du lexer prend en paramètre un <code>java.io.Reader</code> qui pointe vers le texte source</li> <li>Le lexer définit un champ <code>currentLine</code> qui contient la ligne courante et 2 autres champs <code>row</code> et <code>col</code> pour la position</li> <li>Quand <code>pop</code> est appelée, on teste si la ligne courante est renseignée ou pas. Si elle ne l'est pas, on essaie de lire une ligne complète du <code>Reader</code>. Si la méthode retourne <code>null</code>, c'est que la fin du texte source est atteinte, et dans ce cas le lexer se place en un mode où il retourne toujours un token de type <code>EOF</code>.</li> <li>Sinon, et après avoir traité les indentations au début de la ligne (je vais revenir là dessus plus tard), le lexer examine la première lettre de la ligne pour choisir la branche à suivre</li> <li>Si c'est une lettre, alors il continue à consommer la ligne un caractère à la fois jusqu'à ce qu'il trouve autre chose qu'une lettre ou un chiffre, en accumulant les caractères lus dans une chaîne <ul> <li>Si cette chaîne est égale à <code>true</code> ou <code>false</code>, il retourne un token de type <code>BOOL</code>.</li> <li>Si cette chaîne figure dans la liste des mots clés, il retourne un token de type <code>KEYWORD</code></li> <li>Sinon, c'est que c'est un identifiant. Il retourne alors un token de type <code>NAME</code></li> </ul></li> <li>Si le premier caractère est un chiffre, le lexer continue de consommer les caractères tant qu'il trouve des chiffres, puis il retourne un token de type <code>NUM</code></li> <li>Si c'est plutôt <code>"</code> ou <code>'</code>, le lexer lit la chaîne ou le caractère et retourne un token de type <code>STRING</code> ou <code>CHAR</code>. Ce n'est pas très compliqué comme traitement, si ce n'est pour gérer les échappements (<code>"</code> ou <code>n</code> par exemple)</li> <li>Le lexer tente ensuite de lire un symbole parmi la liste des symboles qu'il reconnait. Je vais revenir sur cette partie plus tard, mais l'idée est d'utiliser un automate en essayant de matcher le symbole le plus long (par exemple, matcher un seul token avec la valeur <code>-&gt;</code> plutôt que 2 tokens <code>-</code> et <code>&gt;</code>)</li> <li>Enfin, si on a atteint la fin de la ligne, on remet <code>currentLine</code> à <code>null</code>. De cette manière, le prochain appel à <code>pop</code> va passer à la ligne suivante. Sinon, on lance une erreur car on est face à une entrée non reconnue</li> </ul> <h5>Gestion des blancs</h5> <p>A lecture d'une nouvelle ligne, et avant d'exécuter l'algorithme décrit dans la section précédente, le lexer consomme les espaces en début de ligne en calculant leur nombre, ce qui définit le niveau d'indentation de la ligne. Il compare ensuite cette valeur au niveau d'indentation de la ligne précédente (qu'il maintient dans un champ initialisé à 0):</p> <ul> <li>Si les 2 valeurs sont égales, il retourne un token de type <code>NEWLINE</code></li> <li>Si le niveau d'indentation de la ligne courante est supérieur à celui de la ligne précédente, il retourne un token de type <code>INDENT</code> mais il se met aussi dans un mode où le prochain appel à <code>pop</code> retourne <code>NEWLINE</code>. Dans une première version du lexer de Litil, je générais uniquement <code>INDENT</code> ou <code>DEINDENT</code> quand le niveau d'indentation changeait, <code>NEWLINE</code> sinon. Mais cela posait plein de problèmes dans la phase suivante (le parseur) pour délimiter correctement les blocs, jusqu'à ce que je tombe sur <a href="http://stackoverflow.com/questions/232682/how-would-you-go-about-implementing-off-side-rule/946398#946398">cette réponse sur Stackoverflow</a>. En suivant la méthode décrite dans cette réponse, j'ai fini avec une implémentation beaucoup plus simple et surtout solide du parseur. Je reviendrai là-dessus dans le post consacré
 au parsing.</li> <li>Sinon, il retourne un token de type <code>DEINDENT</code> et se met en un mode pour retourner <code>NEWLINE</code> à l'appel suivant</li> </ul> <p>Un exemple pour clarifier un peu les choses. Etant donné ce texte en entrée:</p> <pre class="java code java" style="font-family:inherit">a   b   c d</pre> <p>Le lexer est censé générer les tokens suivants:</p> <ol> <li><code>NEWLINE</code></li> <li><code>NAME(a)</code></li> <li><code>INDENT</code>: le niveau d'indentation a augmenté à la 2ième ligne</li> <li><code>NEWLINE</code>: toujours produit pour une nouvelle ligne</li> <li><code>NAME(b)</code></li> <li><code>NEWLINE</code>: le niveau d'indentation n'a pas changé entre les 2ième et 3ième lignes</li> <li><code>NAME(c)</code></li> <li><code>DEINDENT</code>: le niveau d'indentation a diminué</li> <li><code>NEWLINE</code></li> <li><code>NAME(d)</code></li> <li><code>EOF</code>: fin de l'entrée</li> </ol> <p>Seulement, l'algorithme décrit jusqu'ici n'est pas suffisant pour que le parseur arrive à gérer proprement l'indentation. En effet, avec l'exemple suivant:</p> <pre class="java code java" style="font-family:inherit">a   b</pre> <p>Le lexer ne va pas produire un token de type <code>DEINDENT</code> après le <code>NAME(b)</code> mais plutôt un <code>EOF</code> car il n'y a pas de nouvelle ligne après le <code>b</code>. On pourrait imaginer une solution où le parseur utilise <code>EOF</code> en plus de <code>DEINDENT</code> pour détecter la fin d'un bloc, mais ce n'est pas suffisant. En effet, avec l'exemple suivant:</p> <pre class="java code java" style="font-family:inherit">a   b     c d</pre> <p>L'implémentation décrite ici va générer un seul token <code>DEINDENT</code>après <code>NAME(c)</code> alors que cette position dans le source marque la fin de 2 blocs et non pas un seul.</p> <p>Pour gérer ce type de situation, et ne voulant pas complexifier encore le code du lexer de base, j'ai décidé de gérer ça dans une autre implémentation de l'interface <code>Lexer</code>, <code>StructuredLexer</code>. Cette dernière implémente le pattern décorateur en délégant à <code>BaseLexer</code> pour générer les tokens du texte source, mais en l'enrichissant avec les traitements suivants:</p> <ul> <li>On maintient le niveau courant d'indentation dans un champ. Le niveau d'indentation est calculé en divisant le nombre d'espaces en début d'une ligne par une taille d'unité d'indentation, fixée à <strong>2 espaces</strong>.</li> <li>Dans <code>pop</code>, si le lexer de base retourne un <code>INDENT</code>: <ul> <li>Vérifier que le nombre d'espaces est un multiple de l'unité. Si ce n'est pas le cas, retourner une erreur</li> <li>S'assurer aussi que le niveau d'indentation n'augmente qu'avec des pas de <strong>1</strong></li> <li>Mettre à jour le champ niveau d'indentation</li> </ul></li> <li>Toujours dans <code>pop</code>, et quand le lexer de base retourne un <code>DEINDENT</code>: <ul> <li>Idem que pour <code>INDENT</code>, s'assurer que le nombre de blancs est un multiple de l'unité d'indentation</li> <li>Si le niveau d'indentation a diminué de plus d'une unité (comme dans l'exemple précédent), générer autant de tokens <code>DEINDENT</code> virtuels que nécessaires, tout en mettant à jour le champ niveau d'indentation</li> </ul></li> <li>Si dans <code>pop</code> le lexer de base retourne <code>EOF</code>, produire des <code>DEINDENT</code> virtuels jusqu'à ce que le niveau d'indentation atteigne 0, puis retourner <code>EOF</code></li> </ul> <p>Ainsi, avec l'exemple suivant:</p> <pre class="java code java" style="font-family:inherit">a   b     c d</pre> <p>Le lexer structuré génère 1 <code>DEINDENT</code> virtuel, en plus du <code>DEINDENT</code> généré par le lexer de base entre <code>c</code> et <code>d</code>. Comme ça, le parseur au dessus pourra détecter la fin de 2 blocs et détecter correctement que <code>d</code> a le même niveau que <code>a</code>.</p> <p>Enfin, <a href="https://github.com/jawher/litil/blob/master/src/main/java/litil/lexer/StructuredLexer.java">le code source</a> qui implémente cet algorithme est disponible dans le repo Github de Litil pour les intéressés.</p> <h5>Gestion des commentaires</h5> <p>Dans Litil, les commentaires sont préfixés par <code>--</code> (double <code>-</code>) et s'étendent sur une seule ligne uniquement. J'ai <em>(arbitrairement)</em> choisi de les gérer au niveau du lexer en les ignorant complètement. Mais j'aurais aussi pu produire des tokens de type <code>COMMENT</code> et les ignorer <em>(ou pas)</em> dans le parseur.</p> <p>Les commentaires sont gérés à 2 endroits dans le lexer:</p> <ul> <li>Dans le code qui lit une ligne du texte source. Si elle commence par <code>--</code>, on ignore la ligne entière et on passe à la ligne suivante (pour gérer les commentaires en début d'une ligne)</li> <li>Dans le code qui gère les symboles. Si le symbole <em>matché</em> correspond à <code>--</code>, on passe à la ligne suivante (pour gérer les commentaires à la fin d'une ligne)</li> </ul> <p>Exemples:</p> <pre class="ocaml code ocaml" style="font-family:inherit"><span style="color: #a52a2a;">--</span> compute <span style="color: #06c; font-weight: bold;">max</span> x y … <span style="color: #06c; font-weight: bold;">NOT</span> <span style="color: #a52a2a;">!</span> <span style="color: #06c; font-weight: bold;">let</span> <span style="color: #06c; font-weight: bold;">max</span> x y <span style="color: #a52a2a;">=</span> x</pre> <pre class="ocaml code ocaml" style="font-family:inherit"><span style="color: #06c; font-weight: bold;">let</span> <span style="color: #06c; font-weight: bold;">max</span> x y <span style="color: #a52a2a;">=</span> x <span style="color: #a52a2a;">--</span> It is a well known fact that x always wins <span style="color: #a52a2a;">!</span></pre> <h5>Gestion des symboles</h5> <p>Avec la gestion des indentations, c'est la partie la plus intéressante <em>(amha)</em> dans le code du lexer.</p> <p>Avant de rentrer dans les détails d'implémentation, je vais d'abord parler un peu d'<a href="http://fr.wikipedia.org/wiki/Automate_fini">automates finis</a>, qui sont une notion centrale dans la théorie des langages et la compilation.</p> <p>Un automate fini est un ensemble d'états et de transitions. On peut le voir comme un système de classification: étant donnée une séquence en entrée, il consomme ses éléments un à un en suivant les transitions adaptés (et donc en passant d'un état à un autre) jusqu'à ce qu'il ait consommé toute l'entrée ou encore qu'il arrive dans un état sans aucune transition  possible. Quelques états peuvent être marqués comme terminaux ou finals, une façon de dire que ça représente un succès. Donc étant donnée un automate et une entrée, si le traitement s'arrête dans un état terminal, on peut dire qu'on a prouvé une propriété donnée sur l'entrée. Cette propriété va dépendre de l'automate.</p> <p>Ok, j'explique comme un pied. Un exemple concrêt:</p> <p><img src="/wp-content/uploads/2015/07/litil-lexer-dfa0.png" alt="litil-lexer-dfa0.png" style="display:block; margin:0 auto;" /></p> <p>L'automate présenté dans la figure précédente se compose de:</p> <ul> <li>Un état initial (conventionnellement appelé <code>S0</code>). C'est l'unique point d'entrée de l'automate</li> <li>3 autres états <code>A</code>, <code>B</code> et <code>C</code>. Notez le double contour de ces états. Ca sert à indiquer que ce sont des états terminaux ou d'acceptation</li> <li>Des transitions entre ces états qui sont annotées par des caractères</li> </ul> <p>Maintenant, appliquons cet automate à la séquence de caractères <code>-&gt;a</code>. Comme dit plus haut, on se positionne initialement au point d'entrée <code>S0</code>. On examine le premier caractère de la séquence d'entrée (<code>-&gt;a</code>) et on cherche une transition qui part de cet état et qui est étiquetée avec ce caractère. Ca tombe bien, le premier caractère <code>-</code> correspond bien à une transition entre <code>S0</code> et <code>A</code>. On suit donc cette trans
ition pour arriver à l'état <code>A</code> et on passe au caractère suivant (<code>&gt;</code>). L'état <code>A</code> est terminal. On pourrait donc interrompre le traitement et dire et qu'on a réussi à matcher le caractère <code>-</code>. Cependant, dans Litil et presque tous les autres langages, le lexer essai plutôt de matcher la séquence la plus longue. L'alternative est qu'il ne faut jamais avoir plusieurs opérateurs qui commencent par le même caractère, ce qui serait trop contraignant.</p> <p>On continue donc le traitement. <code>A</code> dispose bien d'une transition étiquetée avec <code>&gt;</code>. On suit donc cette transition et on arrive à l'état <code>C</code> qui est aussi terminal. Mais par la même logique, on n'abandonne pas tout de suite et on tente de voir s'il est possible de matcher une séquence plus longue. Ce n'est pas le cas ici car l'état <code>A</code> n'a aucune transition étiquetée avec le caractère <code>a</code>. Le traitement s'arrête donc ici, et comme c'est un état terminal, on retourne un succès avec la chaîne <em>matchée</em> qui ici est <code>-&gt;</code>.</p> <p>Notez que l'algorithme que je viens de décrire (et qui est utilisé par l'implémentation actuelle du lexer de Litil) est plutôt simpliste et incomplet par rapport à l'état de l'art car il ne gère pas le <em>backtracking</em>. Par exemple, cet algorithme échoue avec l'automate suivante (censé reconnaitre les symboles <code>-</code> et <code>--&gt;</code>) avec la chaîne <code>--a</code> comme entrée alors qu'il devrait réussir à retourner deux fois le symbole <code>-</code> (le pourquoi est laissé comme exercice au lecteur):</p> <p><img src="/wp-content/uploads/2015/07/litil-lexer-dfa1.png" alt="litil-lexer-dfa1.png" style="display:block; margin:0 auto;" /></p> <p>Dans sa version actuelle, les symboles reconnus par Litil sont: <code>-&gt;</code>, <code>.</code>, <code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>, <code>(</code>, <code>)</code>, <code>=</code>, <code>%</code>, <code>&lt;</code>, <code>&gt;</code>, <code>&lt;=</code>, <code>&gt;=</code>, <code>:</code>, <code>,</code>, <code>[</code>, <code>]</code>, <code>|</code>, <code>_</code>, <code>=&gt;</code>, <code>@@, </code>--<code>, </code>::<code>, </code>{<code> et </code>}@@.</p> <p>Maintenant, juste pour la science, voici l'automate fini correspondant à ces symboles:</p> <p><img src="/wp-content/uploads/2015/07/litil-lexer-dfa-litil.png" alt="litil-lexer-dfa-litil.png" style="display:block; margin:0 auto;" /></p> <p>L'implémentation de cet automate dans Litil est dynamique, dans la mesure où l'automate est construit au <em>runtime</em> à partir d'une liste des symboles à reconnaitre. Aussi, cette implémentation ne gère pas le <em>backtracking</em>, qui est inutile pour le moment car, et à moins que je ne dise de bêtises, le problème décrit plus haut n'arrive que si on a des symboles à 3 caractères (et qui ont le même préfixe qu'un symbole à un caractère), ce qui n'est pas le cas dans Litil (ce n'est pas Scala tout de même. Enfin, pas encore). Par contre, l'implémentation tient en 50 lignes de code Java, et si on ignore le décorum de Java (les imports, les <em>getters</em>, le <code>toString</code>, le constructeur, etc.), l'essence de l'algorithme tient juste dans une douzaine de lignes. <a href="https://github.com/jawher/litil/blob/master/src/main/java/litil/lexer/LexerStage.java">Voici son code source sur Github</a> pour les intéressés.</p> <h5>Lookahead</h5> <p>Une fonctionnalité utile à avoir dans le lexer est le <code>lookahead</code>, i.e. la possibilité de voir les tokens suivants sans pour autant les consommer (comme c'est le cas avec <code>pop</code>). On va revenir la dessus dans le(s) post(s) à propos du parseur.</p> <p>Tout comme <code>StructuredLexer</code>, j'ai décidé d'implémenter cette fonctionnalité dans un décorateur autour d'un lexer concrêt, pour ne pas complexifier le code de ce dernier. Il s'agit de la classe <code>LookaheadLexerWrapper</code> (<a href="https://github.com/jawher/litil/blob/master/src/main/java/litil/lexer/LookaheadLexerWrapper.java">dont voici le code source</a>). L'implémentation derrière est plutôt simple. En effet, l'astuce est juste de maintenir une liste de tokens (en plus du lexer concrêt). Quand la méthode <code>lookahead</code> est appelé (avec un paramètre qui indique le niveau du lookahead: 1 pour le token suivant, etc.), on récupère autant de tokens que nécessaire du lexer et on les stocke dans cette liste. Quand <code>pop</code> est appelée, et avant de déléger au lexer concrêt, on vérifie si la liste de tokens n'est pas vide. Si c'est le cas, on retourne le premier élément de cette liste et en prenant soin de l'enlever, comme ça, l'appel suivant à <code>pop</code> va retourner le token suivant. Si cette liste est vide, alors on délègue au lexer.</p> <h4>Conclusion</h4> <p>Dans ce post, j'ai présenté rapidement et sans trop entrer dans les détails quelques techniques de lexing que j'ai utilisé dans Litil. Ce n'est en aucune façon la meilleure façon de faire ni la plus performante. C'était plus une implémentation relativement facile à coder et à étendre tout en restant raisonnable pour tout ce qui est consommation mémoire (le fait de ne pas lire la totalité du fichier dans une seule <code>String</code> d'un coup par exemple).</p> <p>Il faut aussi noter que c'est la partie la moins <em>marrante</em> à coder dans un langage. J'ai même été tenté de la faire générer par un outil comme AntLR ou SableCC, mais au final j'ai décidé de rester fidèle à ma vision de vraiment tout coder à la main <em>(sauf les parties que je ne coderai pas à la main, NDLR)</em>.</p> <p>Maintenant que cette partie est derrière nous, nous allons enfin pouvoir attaquer les sujets intéressants, e.g. le parseur et l'évaluateur, dans les posts suivants.</p> <p><a href="http://jawher.me/2012/06/27/creation-langage-programmation-litil-2-lexer/">Post original</a></p>