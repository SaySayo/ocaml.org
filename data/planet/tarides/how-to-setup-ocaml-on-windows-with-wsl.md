---
title: How to Setup OCaml on Windows with WSL
description: Learn how to set up OCaml on Windows using WSL2. This guide provides
  step-by-step instructions to use OCaml in a Linux-like environment on Windows.
url: https://tarides.com/blog/2024-05-08-how-to-setup-ocaml-on-windows-with-wsl
date: 2024-05-08T00:00:00-00:00
preview_image: https://tarides.com/blog/images/ocaml_wsl-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>The stable opam 2.2 and a fully Windows compatible ecosystem of OCaml libraries and tools are getting closer every month. That's extremely exciting! With opam 2.2, Windows users will be able to use OCaml directly and natively without extra set-up or workarounds. Everyone is excited about this future, so we often forget people who want to use OCaml on Windows <em>now</em> and without set-up problems.</p>
<p>In this guide, we demonstrate how to set up OCaml on Windows using WSL2. With WSL2, you can write OCaml programs on your Windows computer while utilising all the benefits of working in a Linux environment.</p>
<p>Note that as an alternative to WSL2, it's already possible to install OCaml natively on Windows. If you wish to install OCaml directly on your PC, follow <a href="https://ocaml.org/install">this guide on how to install OCaml natively on Windows using Diskuv</a> or you can use opam 2.2. We recommend native Windows in the following circumstances:</p>
<ul>
<li>For people who want to have Windows commands available (e.g., <code>dir</code>) instead of Unix commands (e.g., <code>ls</code>). Although it's a tougher set-up process.</li>
<li>For people who want to build OCaml native Windows binaries, e.g., to distribute OCaml apps on Windows</li>
<li>For people who are really into Windows from a perspective of technical curiosity</li>
<li>For advanced programmers who are happy to try native Windows support from the beginning</li>
</ul>
<p>For anyone else, like people who just want to try OCaml on their Windows machine, OCaml on Windows via WSL2 is a great solution, so we'll dive right in.</p>
<h2>Prerequisite</h2>
<ul>
<li>Windows 10 or 11 with WSL2 enabled (check <a href="https://learn.microsoft.com/en-us/windows/wsl/install">Mircrosoft: WSL setup</a> or <a href="https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/guides/install-ubuntu-wsl2/">Ubuntu: WSL setup</a> for a guide on how to setup WSL.)</li>
<li>Ubuntu installed on your WSL setup</li>
<li>An active internet connection</li>
</ul>
<h2>Procedure</h2>
<h3>Update Packages</h3>
<ul>
<li>Open your Ubuntu terminal (e.g., by searching &quot;Ubuntu&quot; in the Start menu).</li>
<li>Run the following command to update the package list and upgrade all packages:</li>
</ul>
<pre><code><span class="sh-source">$ sudo apt update </span><span class="sh-keyword-operator-list">&amp;&amp;</span><span class="sh-source"> sudo apt upgrade
</span></code></pre>
<h3>Install Required Packages</h3>
<ul>
<li>Run the following command to install packages required to succesfully install OCaml:</li>
</ul>
<pre><code><span class="sh-source">$ sudo apt install gcc build-essential curl unzip bubblewrap
</span></code></pre>
<h3>Download and Install opam</h3>
<p><a href="https://opam.ocaml.org/">Opam</a> is OCaml's package manager. It makes it easier to install additional libraries and tools relevant to various OCaml projects. It is similar to package managers like <a href="https://pip.pypa.io/en/stable/">pip</a> for Python or <a href="https://www.npmjs.com/">npm</a> for JavaScript. To download and install opam, run the following command:</p>
<pre><code><span class="sh-source">$ bash -c </span><span class="sh-punctuation-definition-string-begin">&quot;</span><span class="sh-string-quoted-double">sh &lt;(curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)</span><span class="sh-punctuation-definition-string-end">&quot;</span><span class="sh-source">
</span></code></pre>
<p>The error below (or similar) is due to DNS issues:
If you don't experience this error, you can skip directly to the
<a href="https://tarides.com/feed.xml#initialise-opam">Initialise opam section</a></p>
<pre><code><span class="sh-source">curl: </span><span class="sh-punctuation-definition-subshell">(</span><span class="sh-meta-scope-subshell">6</span><span class="sh-punctuation-definition-subshell">)</span><span class="sh-source"> Could not resolve host: raw.githubusercontent
</span></code></pre>
<p>To resolve:</p>
<ul>
<li>Use the nano editor to edit the <code>resolv.conf</code> file by running:</li>
</ul>
<pre><code><span class="sh-source">$ sudo nano /etc/resolv.conf
</span></code></pre>
<ul>
<li>Change the address <code>127.0.0.1</code> to <code>8.8.8.8</code>. The final file should look like this:</li>
</ul>
<pre><code><span class="sh-source">nameserver 8.8.8.8
</span></code></pre>
<ul>
<li>Now, we will use the nano editor to edit the <code>wsl.conf</code> file by running:</li>
</ul>
<pre><code><span class="sh-source">$ sudo nano /etc/wsl.conf
</span></code></pre>
<ul>
<li>Add this entry to the file:</li>
</ul>
<pre><code><span class="sh-source">generateResolvConf = </span><span class="sh-support-function-builtin">false</span><span class="sh-source">
</span></code></pre>
<ul>
<li>The final file should look like this:</li>
</ul>
<pre><code><span class="sh-source">[boot]
</span><span class="sh-source">systemd=true
</span><span class="sh-source">generateResolvConf = </span><span class="sh-support-function-builtin">false</span><span class="sh-source">
</span></code></pre>
<ul>
<li>At this point, we can re-run our script to install opam:</li>
</ul>
<pre><code><span class="sh-source">$ bash -c </span><span class="sh-punctuation-definition-string-begin">&quot;</span><span class="sh-string-quoted-double">sh &lt;(curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)</span><span class="sh-punctuation-definition-string-end">&quot;</span><span class="sh-source">
</span></code></pre>
<p>Our script should now run without any issues. If you are prompted for questions, such as where to install it, just press enter to accept the default locations and values.</p>
<h3>Initialise opam</h3>
<p>Now that we have installed opam, the next step is to initalise it. This step creates a new <a href="https://ocaml.org/docs/opam-switch-introduction">opam switch</a>, which acts as an isolated environment for your OCaml development. We can do this by running the command:</p>
<pre><code><span class="sh-source">$ opam init
</span></code></pre>
<p>When prompted, press <code>y</code> to modify the <code>profile</code> file so we can easily run <code>opam</code> commands. If you didn't press <code>y</code>, we can activate the switch by running:</p>
<pre><code><span class="sh-source">$ </span><span class="sh-support-function-builtin">eval</span><span class="sh-source"> </span><span class="sh-punctuation-definition-string-begin">$(</span><span class="sh-string-interpolated-dollar">opam env</span><span class="sh-punctuation-definition-string-end">)</span><span class="sh-source">
</span></code></pre>
<h3>Setup A Development Environment</h3>
<p>Now that we have OCaml setup. The next thing is to setup a development environment by installing packages that make programming in OCaml a much nicer experience. We can do this by using the <code>opam install &lt;package-name&gt;</code> command.
Below are the basic recommended packages and tools for OCaml development:</p>
<ul>
<li><code>ocaml-lsp-server</code> provides an LSP implementation for OCaml giving us a nice experience with editors and IDEs.</li>
<li><code>odoc</code> is a documentation tool that generates human-readable documentation from OCaml code, including comments, types, and function signatures.</li>
<li><code>ocamlformat</code> formats OCaml code according to a defined style guide, ensuring consistent formatting across different code files, which improves readability and maintainability.</li>
<li><code>utop</code> provides an interactive environment for OCaml development where you can directly type in OCaml expressions and see their results immediately. It's perfect for experimenting and testing.</li>
</ul>
<p>We can easily install all these packages at once and set them up by using the <a href="https://github.com/tarides/ocaml-platform-installer?tab=readme-ov-file#trying-the-platform">OCaml Platform Installer</a>. Alternatively, we can manually by type the command below, then follow the installation and set-up instructions:</p>
<pre><code><span class="sh-source">$ opam install ocaml-lsp-server odoc ocamlformat utop
</span></code></pre>
<p>The most supported editors include:</p>
<ul>
<li>VSCode: VSCode also has a WSL version.</li>
<li>Vim</li>
<li>Emacs</li>
</ul>
<p>Based on your preference, you can install any of the editors and begin programming in OCaml. Follow this helpful guide on how to setup your editor: <a href="https://ocaml.org/docs/set-up-editor">Setting up your editor for OCaml development</a>.</p>
<p>We have succesfully setup OCaml on our Windows PC using WSL. Now you can work on OCaml projects just like you would on a Linux computer.</p>

