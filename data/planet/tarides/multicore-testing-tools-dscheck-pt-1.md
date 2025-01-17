---
title: 'Multicore Testing Tools: DSCheck Pt 1'
description: Exploring DSCheck's role in enhancing parallel programming, this post
  delves into its use for testing multi-threaded code and its application in OCaml
  tools.
url: https://tarides.com/blog/2024-02-14-multicore-testing-tools-dscheck-pt-1
date: 2024-02-14T00:00:00-00:00
preview_image: https://tarides.com/blog/images/verifying-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>Reaping the plentiful benefits of parallel programming requires the careful management of the intricacies that come with it. Tarides played a significant part in making <a href="https://github.com/ocaml-multicore/ocaml-multicore">OCaml Multicore</a> a reality, and we have continued to work on supporting tools that make parallel programming in OCaml as seamless as possible.</p>
<p>To that end, the OCaml <a href="https://kcsrk.info/webman/manual/memorymodel.html">memory model</a> is carefully designed to help developers reason about their programs, and OCaml 5 introduced several guarantees to make multi-threaded programming safer and more predictable. Tarides also recently brought <a href="https://tarides.com/blog/2023-10-18-off-to-the-races-using-threadsanitizer-in-ocaml/">ThreadSanitizer</a> support to OCaml, which lets users check their code for possible data races.</p>
<p>This post introduces the <a href="https://github.com/ocaml-multicore/dscheck#motivation">DSCheck</a> library, a model checker written in OCaml. It helps developers catch non-deterministic, hard-to-reproduce bugs in parallel programs. Read on to discover why and how we use DSCheck to thoroughly test multi-threaded code before deploying it!</p>
<h2>Why Use DSCheck in the First Place?</h2>
<p>Formally, concurrent programming means that an <a href="https://medium.com/@itIsMadhavan/concurrency-vs-parallelism-a-brief-review-b337c8dac350#:~:text=A%20system%20is%20said%20to,the%20phrase%20%E2%80%9Cin%20progress.%E2%80%9D">application is making progress on more than one task at the same time</a>. In addition, parallel programming allows for more than one concurrent process to happen simultaneously. When several concurrent processes share a resource in parallel, the complexity and the possibility of bugs increases by several degrees.</p>
<p>This is why developers hear the terms deadlock, starvation, and data races so often when they learn about multicore programming or concurrence. These terms are all different ways of saying the same thing: multicore programming is hard!</p>
<p>Enter DSCheck and other <a href="https://tarides.com/blog/2022-12-22-ocaml-5-multicore-testing-tools/">multicore testing tools</a>! DSCheck is a testing tool with a particular use case: it is used for algorithms that do not spawn domains themselves, are lock-free, and use atomics. Developers can use lock-free multicore programming to significantly boost performance, but they need to check that all the executions possible on the multiple cores (called interleavings) are valid to assert that their program is without faults. Manually, this would be a gargantuan task but with DSCheck, it's made easy!</p>
<h2>When To Use DSCheck: A Naive Counter Implementation</h2>
<p>Let&rsquo;s look at a practical example of when to use DSCheck. For instance, if we choose to implement a naive counter in OCaml Multicore, it might look something like this:</p>
<pre><code><span class="ocaml-keyword-other">module</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">struct</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">t</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-support-type">int</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">t</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">create</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">make</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">		</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">old_value</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">get</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">		</span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">set</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">old_value</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">+</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-source">)</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">get</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">get</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">end</span><span class="ocaml-source">
</span></code></pre>
<p>Now, say we increment the counter on 2 domains in parallel. If our implementation is correct, the counter should be at 2 at the end.</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">main</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">create</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">domainA</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">domainB</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">join</span><span class="ocaml-source"> </span><span class="ocaml-source">domainA</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">join</span><span class="ocaml-source"> </span><span class="ocaml-source">domainB</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">	</span><span class="ocaml-keyword-other">assert</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">get</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-source">)</span><span class="ocaml-source">
</span></code></pre>
<p>However, there are several ways in which the incrementation can go wrong<sup><a href="https://tarides.com/feed.xml#fn-1" role="doc-noteref" class="fn-label">[1]</a></sup> and cause the counter to actually hold the value <code>1</code> by the end of the execution. To figure out what has gone wrong, we need to unfold all the possible interleavings in which the domains can execute their accesses to their shared values &ndash; here only <code>counter</code>. Both domains perform two accesses to the counter: first a read (<code>get</code>) then a write (<code>set</code>).</p>
<p>Since OCaml&rsquo;s <code>atomics</code> guarantee that program order is kept between atomic operations on a single domain, we can not reorder the operations made by the same domain.</p>
<p>However, there are still a few possible interleavings. Domain B could for example only begin working after A is done. This gives us interleaving 1:</p>
<div role="region"><table>
<tbody><tr>
<th>Step</th>
<th>Domain A</th>
<th>Domain B</th>
<th>Counter</th>
</tr>
<tr>
<td>1</td>
<td><code>Atomic.get counter</code></td>
<td></td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td><code>Atomic.set counter (0+1)</code></td>
<td></td>
<td>1</td>
</tr>
<tr>
<td>3</td>
<td></td>
<td><code>Atomic.get counter</code></td>
<td>1</td>
</tr>
<tr>
<td>4</td>
<td></td>
<td><code>Atomic.set counter (1+1)</code></td>
<td>2</td>
</tr>
</tbody></table></div><p>Or they could do the same in reverse order (interleaving 2):</p>
<div role="region"><table>
<tbody><tr>
<th>Step</th>
<th>Domain A</th>
<th>Domain B</th>
<th>Counter</th>
</tr>
<tr>
<td>1</td>
<td></td>
<td><code>Atomic.get counter</code></td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td></td>
<td><code>Atomic.set counter (0+1)</code></td>
<td>1</td>
</tr>
<tr>
<td>3</td>
<td><code>Atomic.get counter</code></td>
<td></td>
<td>1</td>
</tr>
<tr>
<td>4</td>
<td><code>Atomic.set counter (1+1)</code></td>
<td></td>
<td>2</td>
</tr>
</tbody></table></div><p>Or they could actually <em>interleave</em> their actions, which results in interleaving 3 (and interleaving 4 by permuting A and B)</p>
<div role="region"><table>
<tbody><tr>
<th>Step</th>
<th>Domain A</th>
<th>Domain B</th>
<th>Counter</th>
</tr>
<tr>
<td>1</td>
<td><code>Atomic.get counter</code></td>
<td></td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td></td>
<td><code>Atomic.get counter</code></td>
<td>0</td>
</tr>
<tr>
<td>3</td>
<td><code>Atomic.set counter (0+1)</code></td>
<td></td>
<td>1</td>
</tr>
<tr>
<td>4</td>
<td></td>
<td><code>Atomic.set counter (0+1)</code></td>
<td>1</td>
</tr>
</tbody></table></div><p>Interleavings 1 and 2 work fine as the counter ends up with a value of 2. A problem arises in interleavings 3 and 4, since both A and B witness the counter with a 0 value thus causing the counter to end up with a value of 1.</p>
<p>This is obviously quite a simple example (and yes, you can totally avoid this bug by using <code>Atomic.incr</code>), but what it shows is that even with just two domains doing the same thing, composed of only two lines of code, we end up with four different interleavings. This is why we need DSCheck!</p>
<p>DSCheck is a model checker. Its job is to find all possible interleavings (what we just did manually) and test that every single one returns the expected result. If not, DSCheck returns the first interleaving it finds that is not working, and lets you do the work of debugging the code now that you know where and what the problem is. We will show you how we write a test for a naive counter implementation using DSCheck in part 2, so keep a look out for that.</p>
<h2>What is DSCheck Currently Used For?</h2>
<p>We have used DSCheck in several of the libraries and tools we maintain to verify certain aspects of them. For example, in the <a href="https://github.com/ocaml-multicore/saturn">Saturn</a> library of parallelism-safe data structures for OCaml, DSCheck is used to verify both the lock-freedom and safety properties of the structures. The effects-based IO stack <a href="https://github.com/ocaml-multicore/eio">Eio</a> also has its internal lock-free data structures verified with DSCheck. In short, DSCheck lets you build complex Multicore data structures with confidence.</p>
<h2>Part Two</h2>
<p>We'll end this part here now that you have an idea of what DSCheck is and when it is used. Next time, we will look at how DSCheck works in greater detail, including how we use it in Saturn. Check out our blog for part two, and connect with us on <a href="https://twitter.com/tarides_">X</a> and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> to stay up-to-date with what we're up to. See you soon!</p>
<section role="doc-endnotes"><ol>
<li>
<p>Note that the probability of this bug happening is low and may be hard to witness. You will most likely need to add a barrier to synchronise the domains.</p>
<span><a href="https://tarides.com/feed.xml#ref-1-fn-1" role="doc-backlink" class="fn-label">&#8617;&#65038;&#65038;</a></span></li></ol></section>

