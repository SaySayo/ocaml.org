---
title: 'Creating the SyntaxDocumentation Command - Part 2: OCaml LSP'
description: Discover how the `SyntaxDocumentation` command from Merlin was integrated
  into the OCaml LSP, enhancing hover requests with detailed syntax info for developers.
url: https://tarides.com/blog/2024-06-12-creating-the-syntaxdocumentation-command-part-2-ocaml-lsp
date: 2024-06-12T00:00:00-00:00
preview_image: https://tarides.com/blog/images/ufo_hover-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>In the first part of this series, <a href="https://tarides.com/blog/2024-04-17-creating-the-syntaxdocumentation-command-part-1-merlin/">Creating the <code>SyntaxDocumentation</code> Command - Part 1: Merlin</a>, we explored how to create a new command in Merlin, particularly the <code>SyntaxDocumentation</code> command. In this continuation, we will be looking at the amazing OCaml LSP project and how we have integrated our <code>SyntaxDocumentation</code> command into it. OCaml LSP is a broad and complex project, so we will be limiting the scope of this article just to what's relevant for the <code>SyntaxDocumentation</code> command.</p>
<h2>Language Server Protocol</h2>
<p>The <a href="https://microsoft.github.io/language-server-protocol/">Language Server Protocol (LSP)</a> defines the protocol used between an editor or IDE and a language server that provides language features like auto complete, go to definition, find all references, etc. In turn, the protocol defines the format of the messages sent using <a href="https://www.jsonrpc.org/">JSON-RPC</a> between the development tool and the language server. With LSP, a single language server can be used with multiple development tools, such as:</p>
<ul>
<li>Integrated Development Environments (IDEs): Visual Studio Code, Atom, or IntelliJ IDEA</li>
<li>Code editors: Sublime Text, Vim, or Emacs</li>
<li>Text editors with code-related features</li>
<li>Command-line tools for code management, building, or testing</li>
</ul>
<h3>How LSP Works</h3>
<p>Here's a typical interaction between a development tool and a language server:</p>
<ol>
<li><strong>Document Opened:</strong> When the user opens a document, this notifies the language server that a document is open <code>(textDocument/didOpen)</code>.</li>
<li><strong>Editing:</strong> When the user edits the document, this notifies the server about the changes <code>(textDocument/didChange)</code>. The server analyses the changes and notifies the tool of any detected errors and warnings <code>(textDocument/publishDiagnostics)</code>.</li>
<li><strong>Go to Definition:</strong> The user executes &quot;Go to Definition&quot; on a symbol. The tool sends a <code>textDocument/definition</code> request to the server, which responds with the location of the symbol's definition.</li>
<li><strong>Document Closed:</strong> The user closes the document. A <code>textDocument/didClose</code> notification is sent to the server.</li>
</ol>
<h2>OCaml LSP</h2>
<p><a href="https://github.com/ocaml/ocaml-lsp">ocaml-lsp</a> is an implementation of the Language Server Protocol for OCaml in OCaml. It provides language features like code completion, go to definition, find references, type information on hover, and more, to editors and IDEs that support the Language Server Protocol. OCaml LSP is built on top of Merlin, which provides the actual analysis and type information.</p>
<p>Currently, OCaml LSP supports several LSP requests such as <code>textDocument/completion</code>, <code>textDocument/hover</code>, <code>textDocument/codelens</code>, etc. For the purposes of this article, we will limit the scope to <code>textDocument\hover</code> requests because this is where our command is implemented. You can find out more about supported OCaml LSP requests at <a href="https://github.com/ocaml/ocaml-lsp/tree/master?tab=readme-ov-file#features">Features | OCaml LSP</a>.</p>
<h2>Hover Requests</h2>
<p>When a user hovers over a symbol or some syntax, their development tool sends a <code>textDocument/hover</code> request to the language server. To better understand this process, let us consider some sample code:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">get_children</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">position</span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-constant-language-capital-identifier">Lexing</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">position</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">root</span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source">node</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">some</span><span class="ocaml-source"> </span><span class="ocaml-source">code</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">
</span></code></pre>
<p>When the user hovers over the function name, <code>get_children</code>, the hover request (taken from the server logs) is as follows:</p>
<pre><code class="language-json">[Trace - 4:07:21 AM] Sending request 'textDocument/hover - (13)'.
Params: {
    &quot;textDocument&quot;: {
        &quot;uri&quot;: &quot;file:///home/../../merlin/src/kernel/mbrowse.ml&quot;
    },
    &quot;position&quot;: {
        &quot;line&quot;: 279,
        &quot;character&quot;: 10
    }
}
</code></pre>
<p>This request includes the following information:</p>
<ul>
<li>The URI of the document where the user is hovering</li>
<li>The position (line and character) within the document where the hover event occurred</li>
</ul>
<p>The language server then responds with information corresponding to what its hover query should do. This could be type information, documentation information, etc.</p>
<pre><code class="language-json">[Trace - 4:07:21 AM] Received response 'textDocument/hover - (13)' in 2ms.
Result: {
    &quot;contents&quot;: {
        &quot;kind&quot;: &quot;markdown&quot;,
        &quot;value&quot;: &quot;```ocaml\nLexing.position -&gt; ('a * node) list -&gt; node\n```&quot;
    },
    &quot;range&quot;: {
        &quot;end&quot;: {
            &quot;character&quot;: 16,
            &quot;line&quot;: 279
        },
        &quot;start&quot;: {
            &quot;character&quot;: 4,
            &quot;line&quot;: 279
        }
    }
}
</code></pre>
<p>The response received indicates that at this position the type signature is <code>Lexing.position -&gt; ('a * node) list -&gt; node</code>, and it's formatted with Markdown, since it was done in VSCode. For development tools that don't support Markdown, this response will simply be plaintext. The <code>range</code> is used by the editor to highlight the relevant line(s) for the user.</p>
<h2><code>SyntaxDocumentation</code> Implementation</h2>
<p>With OCaml LSP, type information displayed from a hover request is taken from Merlin using the <code>type_enclosing</code> command, and the information returned is passed onto the hover functionality to be displayed as a response. With this, we can attach the result from querying Merlin about the <code>SyntaxDocumentation</code> command and add the results to the <code>type_enclosing</code> response.</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">type_enclosing</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">{</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">loc</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Loc</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">t</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">typ</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">string</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">doc</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">string</span><span class="ocaml-source"> </span><span class="ocaml-source">option</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">syntax_doc</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Query_protocol</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">syntax_doc_result</span><span class="ocaml-source"> </span><span class="ocaml-source">option</span><span class="ocaml-source">
</span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p>To query Merlin for something, we use <code>Query_protocol</code> and <code>Query_command</code>. You can read more about what these do from <a href="https://tarides.com/blog/2024-04-17-creating-the-syntaxdocumentation-command-part-1-merlin/">Part 1</a> of this article series.</p>
<pre><code><span class="ocaml-source"> </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">syntax_doc</span><span class="ocaml-source"> </span><span class="ocaml-source">pipeline</span><span class="ocaml-source"> </span><span class="ocaml-source">pos</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">res</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">command</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Query_protocol</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-constant-language-capital-identifier">Syntax_document</span><span class="ocaml-source"> </span><span class="ocaml-source">pos</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-constant-language-capital-identifier">Query_commands</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">dispatch</span><span class="ocaml-source"> </span><span class="ocaml-source">pipeline</span><span class="ocaml-source"> </span><span class="ocaml-source">command</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">match</span><span class="ocaml-source"> </span><span class="ocaml-source">res</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">with</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-polymorphic-variant">`Found</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Some</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-polymorphic-variant">`No_documentation</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">None</span><span class="ocaml-source">
</span></code></pre>
<h3>Making <code>SyntaxDocumentation</code> Configurable</h3>
<p>Sometimes, too much information can be problematic, which is the case with the hover functionality. Most times, users just want a specific kind of information, and presenting a lot of unrelated information can have a negative effect on their productivity. For this reason, <code>SyntaxDocumentation</code> is made to be configurable, so users can toggle it on or off. This is made possible by passing configuration settings to the server.</p>
<pre><code><span class="ocaml-source">syntaxDocumentation</span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-source">{</span><span class="ocaml-source"> </span><span class="ocaml-source">enable</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-source">boolean</span><span class="ocaml-source"> </span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p>For a piece of code such as:</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">color</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Red</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Blue</span><span class="ocaml-source">
</span></code></pre>
<p>When SyntaxDoc is turned off, we receive the following response:</p>
<pre><code class="language-json">{
      &quot;contents&quot;: { &quot;kind&quot;: &quot;plaintext&quot;, &quot;value&quot;: &quot;type color = Red | Blue&quot; },
      &quot;range&quot;: {
        &quot;end&quot;: { &quot;character&quot;: 21, &quot;line&quot;: 1 },
        &quot;start&quot;: { &quot;character&quot;: 0, &quot;line&quot;: 1 }
      }
    }
</code></pre>
<p>When SyntaxDoc is turned on, we receive the following response:</p>
<pre><code class="language-json">{
      &quot;contents&quot;: {
        &quot;kind&quot;: &quot;plaintext&quot;,
        &quot;value&quot;: &quot;type color = Red | Blue. `syntax` Variant Type: Represent's data that may take on multiple different forms..See [Manual](https://v2.ocaml.org/releases/4.14/htmlman/typedecl.html#ss:typedefs)&quot;
      },
      &quot;range&quot;: {
        &quot;end&quot;: { &quot;character&quot;: 21, &quot;line&quot;: 1 },
        &quot;start&quot;: { &quot;character&quot;: 0, &quot;line&quot;: 1 }
      }
    }
</code></pre>
<h2>Conclusion</h2>
<p>In this article, we looked at the LSP protocol and a few examples of how it is implemented in OCaml. With OCaml LSP, the <code>SyntaxDocumentation</code> command becomes a very handy tool, empowering developers to get documentation information by just hovering over the syntax. If you wish to support the OCaml LSP project, you are welcome to submit issues and code constibutions to the repository at <a href="https://github.com/ocaml/ocaml-lsp/issues">Issues | OCaml LSP</a>. In the next and final part of this series, we will look at the VSCode Platform Extension for OCaml and how we can add a visual checkbox to the UI for toggling on/off <code>SyntaxDocumentation</code>.</p>

