---
ID: 96
post_title: 'Au coeur du JDK : TimeUnit'
author: ocroisier
post_date: 2011-01-04 20:48:00
post_excerpt: ""
layout: post
permalink: http://blog.zenika-offres.com/?p=96
published: true
---
<p>Aujourd'hui, je vais attirer votre attention sur une petite classe très discrète, pourtant présente depuis Java 5&nbsp;: <code>java.util.concurrent.TimeUnit</code>.</p> <p>Comme son nom l'indique, cet enum permet de représenter les unités temporelles usuelles. Rien de bien révolutionnaire, certes, mais ça peut éviter de  réinventer la roue dans chacun de nos projets.</p> <pre> NANOSECONDS MICROSECONDS MILLISECONDS SECONDS MINUTES HOURS DAYS </pre> <p>Là où ça devient plus intéressant, c'est que <code>TimeUnit</code> propose des méthodes de conversion entre les unités&nbsp;:</p> <pre class="java code java" style="font-family:inherit">convert<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">long</span> duration, TimeUnit sourceUnit<span style="color: #000000;">&#41;</span> toMillis<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">long</span> duration<span style="color: #000000;">&#41;</span> toSeconds<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">long</span> duration<span style="color: #000000;">&#41;</span> toHours<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">long</span> duration<span style="color: #000000;">&#41;</span> ...</pre> <p>En particulier, vu le nombre de méthodes qui prennent des millisecondes en paramètre (<code>Thread.sleep()</code>, <code>Object.wait()</code>, etc.), la méthode <code>toMillis()</code> pourra rendre de grands services et clarifier le code&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #808080; font-style: italic;">// Avant : long oneHourInMillis = 1000 * 60 * 60;</span> <span style="color: #7F0055; font-weight: bold;">long</span> oneHourInMillis = TimeUnit.<span style="color: #000000;">HOURS</span>.<span style="color: #000000;">toMillis</span><span style="color: #000000;">&#40;</span><span style="color: #cc66cc;">1</span><span style="color: #000000;">&#41;</span>;</pre> <p>Enfin, cerise sur le gâteau (ou plutôt nain sur la bûche, en cette période de fêtes), <code>TimeUnit</code> facilite l'appel des principales méthodes utilisant des millisecondes&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #808080; font-style: italic;">// Avant : Thread.sleep(1000 * 60)</span> TimeUnit.<span style="color: #000000;">MINUTES</span>.<span style="color: #000000;">sleep</span><span style="color: #000000;">&#40;</span><span style="color: #cc66cc;">1</span><span style="color: #000000;">&#41;</span>; &nbsp; <span style="color: #808080; font-style: italic;">// Avant : Object.wait(1000 * 60 * 2)</span> TimeUnit.<span style="color: #000000;">MINUTES</span>.<span style="color: #000000;">timedWait</span><span style="color: #000000;">&#40;</span>object, <span style="color: #cc66cc;">2</span><span style="color: #000000;">&#41;</span> &nbsp; <span style="color: #808080; font-style: italic;">// Avant : anotherThread.join(1000 * 60 * 3)</span> TimeUnit.<span style="color: #000000;">MINUTES</span>.<span style="color: #000000;">timedJoin</span><span style="color: #000000;">&#40;</span>anotherThread, <span style="color: #cc66cc;">3</span><span style="color: #000000;">&#41;</span></pre> <p>En conclusion, maintenant que vous connaissez la discrète mais très utile classe <code>TimeUnit</code>, je pense que vous ne voudrez plus jamais calculer <code>1000 * 60 * 60 * 24</code> pour représenter... une simple journée&nbsp;!</p>