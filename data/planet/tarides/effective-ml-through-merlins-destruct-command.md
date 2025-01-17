---
title: Effective ML Through Merlin's Destruct Command
description: Enhanced OCaml productivity with Merlin and OCaml LSP's refined `destruct`
  command, simplifying pattern matching by generating exhaustive cases efficiently.
url: https://tarides.com/blog/2024-05-29-effective-ml-through-merlin-s-destruct-command
date: 2024-05-29T00:00:00-00:00
preview_image: https://tarides.com/blog/images/destruct-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>The Merlin server and OCaml LSP server, two closely related OCaml language
servers, enhance productivity with features like autocompletion and type
inference. Their lesser known, yet highly useful <code>destruct</code> command simplifies
the use of pattern matching by generating exhaustive match statements, as we&rsquo;ll
illustrate in this article. The command has recently received a bit of love,
making it more usable, and we are taking advantage of this refresh to introduce
it and showcase some use cases.</p>
<p>A <em>good</em> IDE for a programming language ought to provide contextual information,
such as completion suggestions, details about expressions like types, and
real-time error feedback. However, in an ideal world, it should also serve as a
code-writing assistant, capable of generating code as needed. And even though
there are undeniably commonalities among a broad range of programming languages,
allowing for the &quot;generalisation&quot; of interactions with a code editor via a
protocol (such as <a href="https://github.com/ocaml/ocaml-lsp">LSP</a>), some languages
possess uncommon or even unique functionalities that require special
treatment. Fortunately, it is possible to develop functionalities tailored to
these particularities. These can be invoked within LSP through <strong>custom
requests</strong> to retrieve arbitrary information and <strong>code actions</strong> to transform a
document as needed. Splendid! However, such functionality can be more difficult
to discover, as it somewhat denormalises the IDE user experience. This is the
case with the <code>destruct</code> command, which is immensely useful and saves a great
deal of time.</p>
<p>In this article, we'll attempt to fathom of the command's usefulness and its
application using somewhat simplistic examples. Following that, we'll delve into
a few less artificial examples that I use in my day-to-day coding. I hope that
the article is useful and entertaining both for people who already know
<code>destruct</code> and for people who don't.</p>
<h2>Destruct in Broad Terms</h2>
<p>OCaml allows the expression of <a href="https://ocamlbook.org/algebraic-types/">algebraic data
types</a> that, coupled with <a href="https://ocaml.org/docs/basic-data-types">pattern
matching</a>, can be used to describe data
structures and perform case analysis. In the event that a pattern match falls
short of being exhaustive, <strong>warning 8</strong>, known as <code>partial-match</code>, will be
raised during the compilation phase. Hence, it is advisable to uphold exhaustive
match blocks.</p>
<p>The <code>destruct</code> command aids in achieving completeness. When applied to a pattern
(via <code>M-x merlin-destruct</code> in Emacs, <code>:MerlinDestruct</code> in Vim, and <code>Alt + d</code> in
Visual Studio Code), it generates patterns. The command behaves differently
depending on the cursor&rsquo;s context:</p>
<ul>
<li>
<p>When it is called on an expression, it replaces it by a pattern match over
its constructors.</p>
</li>
<li>
<p>When it is called on a pattern of a non-exhaustive matching, it will make the
pattern matching exhaustive by adding missing cases.</p>
</li>
<li>
<p>When it is called on a wildcard pattern, it will refine it if possible.</p>
</li>
</ul>
<blockquote>
<p>For those unfamiliar with the term <code>destruct</code>, pattern matching is case
analysis, and expressing the form (a collection of patterns) on which you
<em>match</em> is called <strong>destructuring</strong>, because you are unpacking values from
structured data. This is the same terminology <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment">used in
JavaScript</a>.</p>
</blockquote>
<p>Let's examine each of these scenarios using examples.</p>
<h3>Destruct on an Expression</h3>
<p>Destructing an expression works in a fairly obvious way. If the typechecker is
aware of the expression type (in our example, it knows this by
inference), the expression will be substituted by a matching on all enumerable
cases.</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-1~kHA8_iC67tU-2us0hsjbhQ.gif" alt="Destruct on expression"/>
</p>
<h3>Destruct on a Non-Exhaustive Matching</h3>
<p>The second behaviour is, in my opinion, the most practical. Although I rarely
need to substitute an expression with a pattern match, I often want to perform a
case analysis on all the constructors of a sum type. By implementing just a
single pattern, such as <code>Foo</code>, my match expression is non-exhaustive, and if I
<code>destruct</code> on this, it will generate all the missing cases.</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-2~h0Wv8gXWN0rskS6_w-ThXA.gif" alt="Destruct on non-exhaustive match"/>
</p>
<h3>Destruct on a Wildcard Pattern</h3>
<p>The final behaviour is very similar to the previous one; when you <code>destruct</code> a
wildcard pattern (or a pattern producing a wildcard, for example, a variable
declaration), the command will generate all the missing branches.</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-3~_VWYV_Exk4KJ46PdmxbGDA.gif" alt="Destruct on wildcard"/>
</p>
<h3>Dealing With Nesting</h3>
<p>When used interactively, it is possible to destruct nested patterns to quickly
achieve exhaustiveness. For example, let&rsquo;s imagine that our variable <code>x</code> is of
type <code>t option</code>:</p>
<ul>
<li>We start by destructing our wildcard (<code>_</code>), which will produce two branches,
<code>None</code> and <code>Some _</code>.</li>
<li>Then, we can destruct on the associated wildcard of <code>Some _</code>, which will
produce all conceivable cases for the type <code>t</code>.</li>
</ul>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-4~vTIN7T3JhO0yjcwShn0A4g.gif" alt="Destruct on nested patterns"/>
</p>
<h3>In the Case of Products (Instead of Sums)</h3>
<p>In the previous examples, we were always dealing with cases whose domains are
perfectly defined, only destructing cases of simple sum type branches. However,
the <code>destruct</code> command can also act on products. Let's consider a very ambitious
example where we will make exhaustive pattern matching on a value of type <code>t * t option</code>, generating all possible cases using <code>destruct</code> alone :</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-5~IAWhnKdaaVJhB3Iki-jBzQ.gif" alt="Destruct on nested tuples"/>
</p>
<p>It can be seen that when used interactively, the command saves a lot of time,
and coupled with Merlin's real-time feedback regarding errors, one can quickly
ascertain when our pattern matching is exhaustive. In a way, it's a bit like a
manual &quot;deriver.&quot;</p>
<p>The <code>destruct</code> command can act on any pattern, so it also works within function
arguments (although <a href="https://github.com/ocaml/ocaml/pull/12236">their representation has
changed</a> slightly for <code>5.2.0</code>), and
in addition to destructing tuples, it is also possible to destruct records,
which can be very useful for our quest for exhaustiveness!</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-6~uWJtPasoed3rVlH3jgbYPw.gif" alt="Destruct on nested records"/>
</p>
<h3>When the Set of Constructors is Non-Finite</h3>
<p>Sometimes types are not finitely enumerable. For example, how
are we to handle strings or even integers? In such situations, <code>destruct</code> will
attempt to find an example. For integers, it will be <code>0</code>, and for strings, it
will be the empty string.</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-7~qw12P2S9TTKCci78UQ749A.gif" alt="Destruct on non-enumerable values"/>
</p>
<p>Excellent! We have covered a large portion of the behaviors of the <code>destruct</code>
command, which are quite contextually relevant. There are others (such as cases
of destruction in the presence of GADTs that only generate subsets of patterns),
but it's time to move on to an example from the real world!</p>
<h2>The Quest for Exhaustiveness: Effective ML</h2>
<p>In 2010, <a href="https://x.com/yminsky">Yaron Minsky</a> gave an <a href="https://www.youtube.com/watch?v=-J8YyfrSwTk">excellent
presentation</a> on the reasons (and
advantages) for using OCaml at <a href="https://www.janestreet.com/">Jane Street</a>. In
addition to being highly inspiring, it provides specific insights and gotchas on
using OCaml effectively in an incredibly sensitive industrial context (hence the
name &quot;Effective ML&quot;.)! It was in this presentation that the maxim &quot;<em>Make
illegal states unrepresentable</em>&quot; was publicly mentioned for the first time, a
phrase that would later be frequently used to promote other technologies (such
as <a href="https://www.youtube.com/watch?v=IcgmSRJHu_8">Elm</a>). Moreover, the
presentation anticipates many discussions on domain modeling, which are dear to
the <a href="https://en.wikipedia.org/wiki/Software_craftsmanship">Software Craftsmanship
community</a>, by proposing
strategies for domain reduction (later extensively developed in
the book <a href="https://pragprog.com/titles/swdddf/domain-modeling-made-functional/"><em>Domain Modeling Made
Functional</em></a>).</p>
<p>Among the list of effective approaches to using an ML language, Yaron presents a
scenario where one might too hastily use the wildcard in a case analysis. The
example is closely tied to finance, but it's easy to transpose into a simpler
example. We will implement an <code>equal</code> function for a very basic type:</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">t</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Foo</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Bar</span><span class="ocaml-source">
</span></code></pre>
<p>The <code>equal</code> function can be trivially implemented as follows:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">equal</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">match</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">a</span><span class="ocaml-keyword-other-ocaml punctuation-comma punctuation-separator">,</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">with</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Foo</span><span class="ocaml-keyword-other-ocaml punctuation-comma punctuation-separator">,</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Foo</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">true</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Bar</span><span class="ocaml-keyword-other-ocaml punctuation-comma punctuation-separator">,</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Bar</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">true</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language">_</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">false</span><span class="ocaml-source">
</span></code></pre>
<p>Our function works perfectly and is exhaustive. However, what happens if we add
a constructor to our type <code>t</code>?</p>
<pre><code><span class="diff-source">  type t
</span><span class="diff-source">    | Foo
</span><span class="diff-source">    | Bar
</span><span class="diff-punctuation-definition-inserted">+</span><span class="diff-markup-inserted">   | Baz
</span></code></pre>
<p>Our function, in the case of <code>equal Baz Baz</code>, will return <code>false</code>, which is
obviously not the expected behavior. Since the wildcard makes our function
exhaustive, <strong>the compiler won't raise any errors</strong>. That's why Yaron Minsky
argues that in many cases with a wildcard clause, it's probably a mistake. If
our function had been exhaustive, adding a constructor would have raised a
<code>partial-match</code> warning, forcing us to explicitly decide how to behave in the
presence of the new constructor! Therefore, using a wildcard in this context
<strong>deprives us of the fearless refactoring</strong>, which is a strength of OCaml. This
is indeed an argument in favor of using a preprocessor to generate equality
functions, using, for example <a href="https://github.com/ocaml-ppx/ppx_deriving?tab=readme-ov-file#plugins-eq-and-ord">the <code>eq</code> standard
deriver</a>
or the more hygienic <a href="https://github.com/janestreet/ppx_compare"><code>Ppx_compare</code></a>.
But sometimes, using a preprocessor is not possible. Fortunately, the <code>destruct</code>
command can assist us in defining an exhaustive equality function!</p>
<p>We will proceed step by step, specifically separating the different cases and
using nested pattern matching to make the various cases easy to express in a
recurrent manner:</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-8~oY2PNq-cCp4GUov8aoDZ0Q.gif" alt="Destruct for equal on Foo and Bar"/>
</p>
<p>As we can see, <code>destruct</code> allows us to quickly implement an exhaustive <code>equal</code>
function without relying on wildcards. Now, we can add our <code>Baz</code> constructor to
see how the refactoring unfolds! By adding a constructor, we quickly detect a
recurring pattern where we try to give the <code>destruct</code> command <strong>as much leeway
as possible</strong> to generate the missing patterns!</p>
<p align="center">
<img src="https://tarides.com/blog/images/2024-05-21.merlin-destruct/merlin-destruct-9~6-SlJ_0fJKMCUmPd5GJscg.gif" alt="Destruct for equal on Foo, Bar and Baz"/>
</p>
<p>Fantastic! We were able to quickly implement an <code>equal</code> function. Adding a
new case is trivial, leaving <code>destruct</code> to handle all the work!</p>
<p>Coupled with modern text editing features (e.g., using multi-cursors),
it's possible to save a tremendous amount of time! Another example of the
immoderate use of <code>destruct</code> (but too long to be detailed in this article) was
the <a href="https://github.com/xhtmlboi/yocaml/blob/main/lib/yocaml/mime.ml">Mime</a> module
implementation in <a href="https://github.com/xhtmlboi/yocaml">YOCaml</a> for generating RSS feeds.</p>
<h2>In Conclusion</h2>
<p>Paired with a formatter like
<a href="https://github.com/ocaml-ppx/ocamlformat">OCamlFormat</a> (to neatly reformat
generated code fragments), <code>destruct</code> is an unconventional tool in the IDE
landscape. It aligns with algebraic types and pattern matching to simplify code
writing and move towards code that is easier to refactor and thus maintain!
Aware of the command's utility, the <a href="https://github.com/ocaml/merlin">Merlin</a>
team continues to maintain it, streamlining the latest features of OCaml to make
the command as usable as possible in as many contexts as possible!</p>
<p>I hope this collection of illustrated examples has motivated you to use the <code>destruct</code>
feature if you were not already aware of it. Please do not hesitate to send
us ideas for improvements,
fixes, and <strong>fun use cases</strong> via <a href="https://twitter.com/tarides_">X</a> or
<a href="https://www.linkedin.com/company/tarides">LinkedIn</a>!</p>
<p><em>Happy Hacking</em>.</p>

