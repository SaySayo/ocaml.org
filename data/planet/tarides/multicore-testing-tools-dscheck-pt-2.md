---
title: 'Multicore Testing Tools: DSCheck Pt 2'
description: Explore how DSCheck improves multithreaded program testing by efficiently
  catching elusive bugs and validating lock-free properties in OCaml's Saturn library.
url: https://tarides.com/blog/2024-04-10-multicore-testing-tools-dscheck-pt-2
date: 2024-04-10T00:00:00-00:00
preview_image: https://tarides.com/blog/images/DSCheck2-1360w.webp
featured:
authors:
- Tarides
source:
---

<p>Welcome to part two! If you haven't already, check out <a href="https://tarides.com/blog/2024-02-14-multicore-testing-tools-dscheck-pt-1/">part one</a>, where we introduce DSCheck and share one of its uses in a naive counter implementation. This post will give you a behind-the-scenes look at how DSCheck works its magic, including the theory behind it and how to write a test for our naive counter implementation example. We&rsquo;ll conclude by going a bit further, showing you how DSCheck can be used to check otherwise hard-to-prove properties in the <a href="https://github.com/ocaml-multicore/saturn">Saturn</a> library.</p>
<h2>How Does DSCheck Work?</h2>
<p>Developers use DSCheck to catch non-deterministic, hard-to-reproduce bugs in their multithreaded programs. DSCheck does so by ensuring that all the executions possible on the multiple cores (called interleavings) are valid and do not result in faults. Doing this without a designated tool like DSCheck would be incredibly resource-intensive.</p>
<h3>In Theory</h3>
<p>DSCheck operates by simulating parallelism on a single core, which is possible thanks to <a href="https://overreacted.io/algebraic-effects-for-the-rest-of-us/">algebraic effects</a> and a custom scheduler. DSCheck doesn't actually exhaustively 'check' all interleavings but examines a select number of relevant ones that allow it to ensure that all terminal states are valid and that no edge cases have been missed.</p>
<p>You may reasonably be asking yourself how this works. Well, the emergence of <a href="https://users.soe.ucsc.edu/~cormac/papers/popl05.pdf">Dynamic Partial-Order Reduction</a> (DPOR) methods have made DSCheck-like model checkers possible. The DPOR approach to model-checking stems from observations of real-world programs, where many interleavings are equivalent. If at least one of them is covered, so is its entire equivalent class &ndash; which is called a trace. DSCheck, therefore, checks one interleaving per trace, which lets it ensure that the whole equivalent trace is without faults.</p>
<p>Let's illustrate this with a straightforward example:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">a</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">make</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">make</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> Domain A </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">set</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> A1 </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">set</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> A2 </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">|&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">ignore</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> Domain B </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">set</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> B </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">|&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">ignore</span><span class="ocaml-source">
</span></code></pre>
<p>There are three possible interleavings: A1.A2.B, A1.B.A2, and B.A1.A2. The ordering between B and the second step of the first domain, A2, does not matter as it does not affect the same variable. Thus, the execution sequences A1.A2.B and A1.B.A2 are different interleavings of the same trace, which means that if at least one is covered, so is the other.</p>
<p>DPOR skips the redundant execution sequences and provides an exponential performance improvement over an exhaustive (also called naive) search. Since naive model checkers try to explore every single interleaving, and since interleavings grow exponentially with the size of code, there quickly comes a point where the number of interleavings far exceeds what the model checker can cover in a reasonable amount of time. This approach is inefficient to the degree that the only programs a naive model checker can check are so simple that it's almost useless to do so.</p>
<p>By reducing the amount of interleavings that need to be checked, we have significantly expanded the universe of checkable programs. DSCheck has reached a point where its performance is strong enough to test relatively long code, and most significantly, we can use it for data structure implementation.</p>
<p>In addition to the DPOR approach, some conditions must be met for the validations that DSCheck performs to be sound. These conditions include:</p>
<ul>
<li><strong>Determinism:</strong> DSCheck runs the same program multiple times (once per explored interleaving). There should be no randomness in between these executions, meaning that they should all run with the same seed, since otherwise DSCheck may miss some traces and thus miss bugs.</li>
<li><strong>Data-Races:</strong> The program being tested cannot have data races between non-atomic variables, as DSCheck does not see such different behaviours. You should use <a href="https://github.com/ocaml-multicore/ocaml-tsan">TSan</a> before running DSCheck to remove data races.</li>
<li><strong>Atomics:</strong> Domains can only communicate through atomic variables. Validation, including higher-level synchronisation primitives (like mutexes), has not yet been implemented.</li>
<li><strong>Lock-Free:</strong> Programs being tested have to be at least lock-free. Lock-free programs are programs that have multiple threads that share access memory, where none of the threads can block each other. If a program has a thread that cannot finish independently, DSCheck will explore its transitions ad infinitum. To partially mitigate this problem, we can force tests to be lock-free. For example, we can modify a spinlock to explicitly fail once it reaches an artificial limit.</li>
</ul>
<h3>In Practice</h3>
<p>Let's look at how a DSCheck test can catch a bug in a naive counter implementation. To see how we set up the naive counter implementation in this example, have a look <a href="https://tarides.com/blog/2024-02-14-multicore-testing-tools-dscheck-pt-1/">at part one</a> of our two-part DSCheck series.</p>
<p>This is how to write a test for the previous naive counter module we introduced in part one:</p>
<pre><code><span class="ocaml-keyword-other">module</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Dscheck</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-constant-language-capital-identifier">TracedAtomic</span><span class="ocaml-source">
</span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> The test needs to use DSCheck's atomic module </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">
</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">test_counter</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">trace</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">create</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source">)</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> [Atomic.spawn] is the DSCheck function to simulate [Domain.spawn] </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source">)</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-comment-block">(*</span><span class="ocaml-comment-block"> There is no need to join domains as DSCheck does not actually spawn domains. </span><span class="ocaml-comment-block">*)</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">final</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">check</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-other">fun</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">get</span><span class="ocaml-source"> </span><span class="ocaml-source">counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">==</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-source">)</span><span class="ocaml-source">)</span><span class="ocaml-source">)</span><span class="ocaml-source">
</span></code></pre>
<p>As you can tell, the test is very similar to the <a href="https://tarides.com/blog/2024-02-14-multicore-testing-tools-dscheck-pt-1/"><code>main</code> function we wrote previously</a> to check our counter manually, but now it uses DSCheck's interface. This includes:</p>
<ul>
<li>Shadowing the atomic module with DSCheck's <code>TracedAtomic</code>, which adds the algebraic effects we need to compute the interleavings</li>
<li><code>Atomic.trace</code> takes the code for which we want to test its interleavings as an input.</li>
<li><code>Atomic.spawn</code> simulates <code>Domain.spawn</code>.</li>
</ul>
<p>In this case, DSCheck will return the following output:</p>
<pre><code>Found assertion violation at run 2:
sequence 2
----------------------------------------
P0 P1
----------------------------------------
start
get a
                        start
                        get a
set a
                        set a
----------------------------------------
</code></pre>
<p>This output reveals the buggy interleaving with one column per domain (<code>P0</code> and <code>P1</code>). We need to infer ourselves that <code>a</code> means <code>counter</code> here, but once we know that this is pretty straightforward to read, isn't it?</p>
<h2>Case Study: Saturn</h2>
<p>Let's take a closer look at using DSCheck with <a href="https://github.com/ocaml-multicore/saturn">Saturn</a>. Offering industrial-strength, well-tested data structures for OCaml 5, the library makes it easier for Multicore users to find data structures that fit their needs. If you use a data structure from Saturn, you can be sure it has been tested to perform well with Multicore usage. In Saturn, DSCheck has two primary uses: firstly, the one demonstrated above, i.e. catching interleavings that return buggy results; secondly, we use it to detect blocking situations.</p>
<p>Most data structures available through Saturn need to be lock-free. As part of the lock-free property&rsquo;s definition, the structure also needs to be obstruction-free, which technically means that a domain running in isolation can always make progress or be free of blocking situations. So, if all domains bar one are paused partway through their execution, the one still working can finish without issue or being blocked. The most common blocking situation is due to a lock; if one domain acquires a lock, all the other domains must wait until the first one has released the lock to proceed.</p>
<p>Let's take a look at how DSCheck tests for blocking situations:</p>
<p>Here is an example of a code that is <strong>not</strong> obstruction-free. This is a straightforward implementation of a barrier for two domains. Both domains need to increment it to pass the whole loop.</p>
<pre><code><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">barrier</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">make</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">work</span><span class="ocaml-source"> </span><span class="ocaml-source">id</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">print_endline</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-string-quoted-double">Hello world, I'm </span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-keyword-operator">^</span><span class="ocaml-source"> </span><span class="ocaml-source">id</span><span class="ocaml-source">)</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">barrier</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">while</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Atomic</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">get</span><span class="ocaml-source"> </span><span class="ocaml-source">barrier</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&lt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">do</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-constant-language-unit">()</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">done</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">domainA</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">work</span><span class="ocaml-source"> </span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-string-quoted-double">A</span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">domainB</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">spawn</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">work</span><span class="ocaml-source"> </span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-string-quoted-double">B</span><span class="ocaml-string-quoted-double">&quot;</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">join</span><span class="ocaml-source"> </span><span class="ocaml-source">domainA</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-constant-language-capital-identifier">Domain</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">join</span><span class="ocaml-source"> </span><span class="ocaml-source">domain</span><span class="ocaml-source">
</span></code></pre>
<p>In this example, if B is paused by the operating system after printing <code>&quot;Hello world, I'm B&quot;</code>, then A can not progress past the barrier even though it is the only domain currently working. This code is thus not obstruction-free.</p>
<p>If we run this code through DSCheck, here is one interleaving it will explore.</p>
<div role="region"><table>
<tbody><tr>
<th>Step</th>
<th>Domain A</th>
<th>Domain B</th>
<th>Barrier</th>
</tr>
<tr>
<td>1</td>
<td></td>
<td>prints &quot;Hello world, I'm B!&quot;</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>prints &quot;Hello world, I'm A!&quot;</td>
<td></td>
<td>0</td>
</tr>
<tr>
<td>3</td>
<td>Increases <code>barrier</code></td>
<td></td>
<td>1</td>
</tr>
<tr>
<td>4</td>
<td>Reads <code>barrier</code> and loops</td>
<td></td>
<td>1</td>
</tr>
<tr>
<td>5</td>
<td></td>
<td>Increases <code>barrier</code></td>
<td>2</td>
</tr>
<tr>
<td>6</td>
<td></td>
<td>Reads <code>barrier</code> and passes the loop</td>
<td>2</td>
</tr>
<tr>
<td>7</td>
<td>Reads <code>barrier</code> and passes the loop</td>
<td></td>
<td>2</td>
</tr>
</tbody></table></div><p>In this interleaving, domain A only performs the loop once, waiting for domain B to increase the barrier. However, nothing prevents A from looping forever here if B never takes step 5. In other words, step 4 can be repeated one, two, three&hellip; an infinite number of times, creating a <em>new</em> trace (i.e. a new interleaving that is not equivalent to the previous one) each time. As DSCheck will try to explore every possible trace (i.e. each equivalent class of interleavings), the test will never finish. We can determine that a test is not going to finish by noting how the explored interleavings keep growing in size. In this case, they will look like B-A-A-A-B-B-A, then B-A-A-A-A-B-B-A, then B-A-A-A-A-A-B-B-A and so on. When this scenario occurs, we can conclude that our code is not obstruction-free.</p>
<p>To summarise, if DSCheck runs on some accidentally blocking code, it will not be able to terminate its execution as it will have infinite traces to explore. This is one way to determine if your tested code is obstruction-free, a property that is otherwise hard to prove, and a handy test for Saturn as it's a property that most of its data structures are supposed to have. It is important to note that we have simplified our example for this post, and in practice DSCheck also checks for lock-freedom.</p>
<h2>Want More Info?</h2>
<p>We invite you to discover more about DSCheck and the features that come with the multicore testing suite. You can read about the library on <a href="https://github.com/ocaml-multicore/dscheck#motivation">GitHub</a>, including more examples and performance optimisations since its initial release.</p>
<p>We also previously published a <a href="https://tarides.com/blog/2022-12-22-ocaml-5-multicore-testing-tools/">blog post about Multicore testing tools</a> that includes a section on DSCheck. It provides additional helpful context to the set of tools that surround DSCheck.</p>
<h2>Until Next Time</h2>
<p>We hope you enjoyed this sojourn into the realm of DSCheck! If you have any questions or comments, please reach out to us on <a href="https://twitter.com/tarides_">X</a> or <a href="https://www.linkedin.com/company/tarides">LinkedIN</a>.</p>
<p>You can also <a href="https://tarides.com/contact/">contact us</a> with questions or feedback and <a href="https://tarides.com/newsletter/">sign up for our newsletter</a> to stay up-to-date with what we're doing.</p>

