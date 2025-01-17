---
title: 'Dune Developer Preview: Installing The OCaml Compiler With Dune Package Management'
description: Dune's Package Management now supports direct installation of OCaml compilers,
  streamlining setup and enabling shared usage across projects for faster builds.
url: https://tarides.com/blog/2024-10-16-dune-developer-preview-installing-the-ocaml-compiler-with-dune-package-management
date: 2024-10-16T00:00:00-00:00
preview_image: https://tarides.com/blog/images/enhancing_dune-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>We&rsquo;re excited to share a significant update to Dune's Package Management system, particularly one that will be of great interest to OCaml developers. For those who have been exploring Dune&rsquo;s experimental package management capabilities over the past six months, you&rsquo;ll be pleased to know that we've recently merged a feature allowing OCaml compiler packages to be installed directly through Dune.</p>
<p>Until now, Dune&rsquo;s Package Management has supported the installation of various packages from the opam repository. However, it lacked the ability to install a functioning OCaml compiler &mdash; a crucial component for most Dune projects. This limitation meant that early adopters of Dune&rsquo;s Package Management had to rely on external tools to manage their OCaml compiler installations, which wasn&rsquo;t ideal. The good news is that this obstacle has been removed, making Dune Package Management far more robust and ready for testing by early adopters.</p>
<p>The challenge we faced in integrating the OCaml compiler installation stemmed from a conflict between the compiler&rsquo;s build system and the way Dune handles package builds. Dune builds packages in what&rsquo;s known as a &ldquo;sandbox&rdquo; &mdash; a temporary directory where a package is initially constructed and installed. Once the build is complete, the package's installed components are moved to their final destination within Dune&rsquo;s build directory. However, the OCaml compiler assumes that its installation location will be its permanent home. Moving the installed files after installation caused the compiler to malfunction, making this an intractable problem for Dune&rsquo;s package management.</p>
<p>While work is underway to make the OCaml compiler more flexible in terms of its installation location, we didn&rsquo;t want to delay Dune&rsquo;s package management features until this work was completed. Instead, we&rsquo;ve introduced a workaround that allows compiler packages to be installed in a way that maintains their functionality.</p>
<p>The solution involves installing OCaml compiler packages to a global, per-user directory, rather than within the project&rsquo;s sandbox. By default, the compiler is installed in a directory such as <code>~/.cache/dune/toolchains/ocaml-base-compiler.&lt;version&gt;-&lt;hash&gt;</code>. This ensures that the compiler remains in its expected location and operates correctly without the need for relocation.</p>
<p>Moreover, this approach offers an added benefit: compilers installed through Dune can be shared across multiple projects. If two projects use the same version of the compiler, the installation step can be skipped in the second project, significantly speeding up the build process. Given that installing an OCaml compiler can take several minutes, this optimisation will save developers considerable time.</p>
<p>In summary, this new feature represents a substantial improvement to Dune&rsquo;s Package Management system, making it easier and faster for developers to set up and manage their OCaml projects. By enabling direct installation of OCaml compilers through Dune, we&rsquo;re removing a major barrier to adoption and enhancing the overall development experience.</p>
<p>Since it works transparently whenever you build a project with the Developer Preview, we'd love for you to test it out! Try out the new <a href="https://tarides.com/blog/2024-10-03-introducing-the-dune-developer-preview-a-new-era-for-ocaml-development/">Dune Developer Preview</a> today, and let us know how it goes on <a href="https://discuss.ocaml.org/">Discuss</a>. We&rsquo;re eager to see how the community leverages this new capability and look forward to your feedback as we continue to refine and expand Dune&rsquo;s Package Management features.</p>

