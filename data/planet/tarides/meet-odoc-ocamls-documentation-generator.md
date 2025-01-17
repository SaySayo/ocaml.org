---
title: Meet odoc, OCaml's Documentation Generator
description: Contributed to `odoc` development by rewriting its model for 2.0, adding
  source code linking, and implementing new Dune rules, enhancing OCaml documentation.
url: https://tarides.com/blog/2024-01-10-meet-odoc-ocaml-s-documentation-generator
date: 2024-01-10T00:00:00-00:00
preview_image: https://tarides.com/blog/images/odoc-1360w.webp
featured:
authors:
- Tarides
source:
---


<p>Effective documentation is a cornerstone of software development. It helps developers understand how to use a language, its libraries, and its tooling, which leads to more robust and maintainable code. When it comes to OCaml, <code>odoc</code> is the wizard behind the scenes, ensuring developers not only understand OCaml's quirks but also become familiar with its libraries and tools. <code>odoc</code> powers the OCaml.org package documentation, so it's used widely by the entire OCaml community.</p>
<p><a href="https://ocaml.github.io/odoc/"><code>odoc</code></a> is a documentation generator specifically designed for OCaml. It takes comments in the source code and generates documentation in HTML, man pages, and LaTex for PDF generation. Think of it like <a href="https://www.sphinx-doc.org/en/master/">Sphinx</a> or <a href="https://en.wikipedia.org/wiki/Javadoc">Javadoc</a> specifically made for OCaml. In other words, what Sphinx is to Python, <code>odoc</code> is to OCaml. With <code>odoc</code>, you can automatically create documentation for your libraries.</p>
<p>See it in action! <a href="https://ocaml.github.io/odoc/#overview">The <code>odoc</code> documentation</a> pages, the <a href="https://aantron.github.io/dream/">Dream docs</a>, and the <a href="https://ocaml.org/p/cmdliner/latest">online OCaml Packages docs (e.g., for <code>cmdliner</code>)</a> were rendered using <code>odoc</code>.</p>
<h2>History of <code>odoc</code></h2>
<p><a href="https://v2.ocaml.org/manual/ocamldoc.html">OCamldoc</a>, developed by Maxence Guesdon, was the earliest documentation generator for OCaml. It created docs in HTML by using comments in the source code tagged with <code>(**</code> and <code>*)</code>. This allowed developers to produce documentation that closely followed the code and made it easy to keep the docs up to date. It provided a simple and efficient way to generate documentation for OCaml projects.</p>
<p>In recent years, there has been a transition to <code>odoc</code>, an open-source project that OCaml Labs and Tarides developed. This more modern and expandable documentation generator is built on the OCaml compiler's infrastructure. This makes <code>odoc</code> more tightly integrated with the language, so it supports all aspects of the OCaml language.</p>
<p><code>odoc</code> offers several advantages over OCamldoc. It provides support for features like cross-referencing, modular documentation, and custom HTML theming, making it easier for developers to generate comprehensive and visually appealing documentation for their OCaml projects. Other recent features added to <code>odoc</code> include linking to source code and support for searching through the documentation.</p>
<p>The adoption of <code>odoc</code> marks a significant step forward to improve the quality and accessibility of OCaml documentation. It reflects the OCaml community's continuous effort to build an ecosystem of well-documented packages to improve OCaml developer experience and support the community's growth.</p>
<h2>Generate Docs With <code>odoc</code></h2>
<p><code>odoc</code> produces documentation by reading special comments embedded into the source code. <code>odoc</code>'s rich markup language allows standard formatting elements such as bold, italic, lists, and code sections, as well as section headings or even tags (<a href="https://ocaml.github.io/odoc/odoc_for_authors.html">like @param</a>) for adding custom data to specific aspects.</p>
<p>It's quite simple to use <code>odoc</code> because it reads the comments delineated with <code>(** ... *)</code>. For example:</p>
<pre><code>(** This is an OCaml docstring. It supports {b bold}, {e italic}, [code], and much more!
    Here is a list:
    - Item 1
    - Item 2
    And even a table:
    {t
    Table | support
    ------|--------
    is    | cool!
    } *)
</code></pre>
<p>Here is what it looks like after rendering:</p>
<p><img src="https://tarides.com/blog/images/odoc_table-1360w~WPLQGtVyyPw4zLj9XjgHfg.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/odoc_table-170w~-2ofzG-FNbX2bMLbXSxRhw.webp 170w, /blog/images/odoc_table-340w~sdofbDRSJdd02mrcpT767w.webp 340w, /blog/images/odoc_table-680w~LaRkAyVuUVso9Qayk819Hw.webp 680w, /blog/images/odoc_table-1360w~WPLQGtVyyPw4zLj9XjgHfg.webp 1360w" alt="odoc table"/></p>
<h2><code>odoc</code> Features</h2>
<h3>Modules</h3>
<p>In <code>odoc</code>, the basic unit of organisation is the module. The <code>odoc</code> tool generates one page for each module, module type, class, and class type for HTML, LaTex, or man pages output.</p>
<h3>Extensivity</h3>
<p><code>odoc</code> will document all of the values, types, and classes, along with those specially-formatted comments, for each module.</p>
<h3>Cross-Reference</h3>
<p><code>odoc</code> has an accurate cross-referencer that can calculate links between types, modules, module types, and more. A simple click will take you to the type's definition. So if you've ever been baffled by exactly what the <code>t</code> was in <code>val f : t -&gt; unit</code>, <code>odoc</code> will link to it!</p>
<h3>Expander</h3>
<p>Figuring out a module's exact content from its signature is not always easy when using OCaml's expressive features such as <code>include</code> or <code>module type of</code>. <code>odoc</code> always expands such constructs to provide the reader the exact list of items available in the module!</p>
<h2>Tarides and <code>odoc</code></h2>
<p>Tarides wants to improve the OCaml developer experience and remove the blockers for new developers to adopt OCaml. An ecosystem of well-documented packages is critical to language adoption and creating a great developer experience. We aim to provide package authors with excellent tooling, so they can write rich documentation. Tarides' commitment to improving <code>odoc</code> directly addresses these goals.</p>
<p>Tarides contributes significantly to the development of <code>odoc</code>. Tarides engineer Jon Ludlam has led the project and has been rewriting <code>odoc</code>'s model for 2.0. We've added source code linking and support for search. Plus, we developed new <code>odoc</code> rules for Dune that can generate a link to the dependencies' documentation. We even created the <code>odoc</code> driver and CI pipeline that produce the package documentation for OCaml.org</p>
<h2>Conclusion</h2>
<p>Effective documentation guides developers through a language's intricacies, library functionalities, and tool utilisation. A documentation generator proves to be an indispensable conduit that translates complicated code into accessible, structured documentation. A <em>dedicated</em> documentation generator, like <code>odoc</code>, offers more than just documentation creation. It plays a crucial role in helping OCaml developers create well-documented libraries and applications, making it easier for programmers to work with the language and its rich ecosystem of libraries. It also encourages collaboration, accelerates learning curves, and crucially nurtures the growth of robust, well-documented, maintainable codebases. Tarides understands the huge benefit this gives developers, which is why we have a team dedicated to maintaining and improving <code>odoc</code>.</p>
<p>Join the <code>odoc</code> conversation on <a href="https://discuss.ocaml.org/c/eco/15">discuss.ocaml.org</a> under the Ecosystem category by using the <code>odoc</code> tag. Also, please don't hesitate to <a href="https://github.com/ocaml/odoc">open an issue</a> on GitHub, as we're always striving to improve our products. Get started by <a href="https://github.com/ocaml/odoc">installing <code>odoc</code> today</a>!</p>

