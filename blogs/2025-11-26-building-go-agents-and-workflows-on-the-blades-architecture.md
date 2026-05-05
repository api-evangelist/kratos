---
title: "Building Go Agents and Workflows on the Blades Architecture"
url: "https://go-kratos.dev/blog/blades/building-effective-agents/"
date: "Wed, 26 Nov 2025 00:00:00 GMT"
author: ""
feed_url: "https://go-kratos.dev/blog/rss.xml"
---
<div><h2 id="what-is-an-agent">What is an Agent?</h2></div>
<p>There is no unified, strict definition of <strong>Agent</strong> in the industry. Descriptions of Agents vary slightly among different vendors, open-source communities, and even academic papers. However, overall, an Agent typically refers to <strong>an intelligent system capable of long-term autonomous operation, possessing certain decision-making abilities, and able to call various external tools as needed by the task</strong>.</p>
<p>It can manifest as a program that “understands goals, makes judgments, and executes actions,” or it can be a traditional <strong>rule-based fixed-process robot</strong>.</p>
<p>In Blades, we collectively refer to these different forms as <strong>Agentic Systems</strong>. Although they all fall under the category of Agents, when designing the actual architecture, it is essential to distinguish between two core concepts: <strong>Workflow</strong> and <strong>Agent</strong>.</p>
<div><h3 id="workflow">Workflow</h3></div>
<p>Workflow leans more towards traditional software thinking: it relies on <strong>predefined execution steps</strong> to organize the calling relationships between LLMs and tools. All logical sequences are planned in advance and hardcoded into flowcharts or code.</p>
<p>Characteristics:</p>
<ul>
<li>The execution path is <strong>fixed</strong> and predictable</li>
<li>The inputs, outputs, and conditional branches for each step are predetermined by the developer</li>
<li>More suitable for tasks with standardized, stable, and decomposable business processes</li>
<li>Easy to control boundaries, test, and audit</li>
</ul>
<p>In a nutshell: <strong>The model follows the process, rather than the process adapting to the model.</strong></p>
<div><h3 id="agent">Agent</h3></div>
<p>The core value of an Agent lies in its “autonomy” and “adaptability.” It does not run according to a flowchart; instead, it uses the LLM as the “brain” to <strong>dynamically decide the next step</strong> based on the task objective and current state.</p>
<p>Typical capabilities of an Agent include:</p>
<ul>
<li>The LLM <strong>autonomously decides</strong> the execution steps, rather than relying on a preset process</li>
<li>Capable of autonomously selecting, combining, or making multiple calls to external tools</li>
<li>Possesses “memory” and “understanding” of the global task state during execution</li>
<li>Dynamically changes strategy based on real-time feedback, rather than following fixed branches</li>
<li>In some cases, it can run continuously, similar to a “service-type Agent”</li>
</ul>
<p>Therefore, compared to a Workflow, an Agent is closer to an <strong>active executor</strong> rather than a passive process node.</p>
<p>In a nutshell: <strong>A Workflow is “the process controlling the model,” whereas an Agent is “the model controlling the process.”</strong></p>
<div><h3 id="building-with-blades-agent">Building with Blades: Agent</h3></div>
<p>Leveraging Go’s concise syntax and high concurrency features, Blades provides a flexible and extensible Agent architecture. Its design philosophy is to enable developers to easily extend Agent capabilities while maintaining high performance through unified interfaces and pluggable components.
The overall architecture is as follows:</p>
<p><img alt="architecture" src="https://go-kratos.dev/images/architecture.png" /></p>
<p>In <strong>Blades</strong>, you can create an Agent using <code dir="auto">blades.NewAgent</code>. The Agent is the core component of the framework, primarily responsible for:</p>
<ul>
<li>Calling the LLM model</li>
<li>Managing tools</li>
<li>Executing prompts</li>
<li>Controlling the entire task execution flow</li>
</ul>
<p>Simultaneously, Blades provides an elegant and straightforward way to customize tools using <code dir="auto">tools.NewTool</code> or <code dir="auto">tools.NewFunc</code>, allowing the Agent to access external APIs, business logic, or computational capabilities.</p>
<p>Here is an example of a custom weather query tool:</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Define weather handling logic</span></div></div><div><div><span>func</span><span> </span><span>weatherHandle</span><span><span>(</span><span>ctx</span><span> context.Context, </span><span>req</span><span> WeatherReq) (WeatherRes, </span></span><span>error</span><span>) {</span></div></div><div><div><span>    </span><span>return</span><span> WeatherRes{</span><span>Forecast</span><span>: </span><span>"</span><span>Sunny, 25°C</span><span>"</span><span>}, </span><span>nil</span></div></div><div><div><span>}</span></div></div><div><div><span>// Create a weather tool</span></div></div><div><div><span>func</span><span> </span><span>createWeatherTool</span><span>() (tools.Tool, </span><span>error</span><span>) {</span></div></div><div><div><span>    </span><span>return</span><span> </span><span>tools</span><span>.</span><span>NewFunc</span><span>(</span></div></div><div><div><span>        </span><span>"</span><span>get_weather</span><span>"</span><span>,</span></div></div><div><div><span>        </span><span>"</span><span>Get the current weather for a given city</span><span>"</span><span>,</span></div></div><div><div><span>        </span><span>weatherHandle</span><span>,</span></div></div><div><div><span><span>    </span></span><span>)</span></div></div><div><div><span>}</span></div></div></code></pre></figure></div>
<p>Then build a smart assistant (Weather Agent) capable of calling the weather tool:</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Configure the model to call and its address</span></div></div><div><div><span>model</span><span> </span><span>:=</span><span> </span><span>openai</span><span>.</span><span>NewModel</span><span>(</span><span>"</span><span>deepseek-chat</span><span>"</span><span>, openai.Config{</span></div></div><div><div><span>    </span><span>BaseURL</span><span>: </span><span>"</span><span>https://api.deepseek.com</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>APIKey</span><span>: </span><span>os</span><span>.</span><span>Getenv</span><span>(</span><span>"</span><span>YOUR_API_KEY</span><span>"</span><span>),</span></div></div><div><div><span>})</span></div></div><div><div><span>// Create a Weather Agent</span></div></div><div><div><span>agent</span><span>, </span><span>err</span><span> </span><span>:=</span><span> </span><span>blades</span><span>.</span><span>NewAgent</span><span>(</span></div></div><div><div><span>    </span><span>"</span><span>Weather Agent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithModel</span><span>(</span><span>model</span><span>),</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithInstruction</span><span>(</span><span>"</span><span>You are a helpful assistant that provides weather information.</span><span>"</span><span>),</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithTools</span><span>(</span><span>createWeatherTool</span><span>()),</span></div></div><div><div><span>)</span></div></div><div><div><span>if</span><span> </span><span>err</span><span> </span><span>!=</span><span> </span><span>nil</span><span> {</span></div></div><div><div><span>    </span><span>log</span><span>.</span><span>Fatal</span><span>(</span><span>err</span><span>)</span></div></div><div><div><span>}</span></div></div><div><div><span>// Query the weather information for Shanghai</span></div></div><div><div><span>input</span><span> </span><span>:=</span><span> </span><span>blades</span><span>.</span><span>UserMessage</span><span>(</span><span>"</span><span>What is the weather in Shanghai City?</span><span>"</span><span>)</span></div></div><div><div><span>runner</span><span> </span><span>:=</span><span> </span><span>blades</span><span>.</span><span>NewRunner</span><span>(</span><span>agent</span><span>)</span></div></div><div><div><span>output</span><span>, </span><span>err</span><span> </span><span>:=</span><span> </span><span>runner</span><span>.</span><span>Run</span><span>(</span><span>context</span><span>.</span><span>Background</span><span>(), </span><span>input</span><span>)</span></div></div><div><div><span>if</span><span> </span><span>err</span><span> </span><span>!=</span><span> </span><span>nil</span><span> {</span></div></div><div><div><span>    </span><span>log</span><span>.</span><span>Fatal</span><span>(</span><span>err</span><span>)</span></div></div><div><div><span>}</span></div></div><div><div><span>log</span><span>.</span><span>Println</span><span>(</span><span>output</span><span>.</span><span>Text</span><span>())</span></div></div></code></pre></figure></div>
<p>With Blades’ framework design, you can quickly build an Agent capable of both calling tools and making autonomous decisions with just a small amount of code.</p>
<p>Next, we will introduce several typical workflow patterns based on the Blades framework. Each design pattern is suitable for different business scenarios, ranging from the simplest linear flows to highly autonomous Agents. You can choose the appropriate design method according to your actual business needs.</p>
<div><h2 id="pattern-1-chain-workflow">Pattern 1: Chain Workflow</h2></div>
<p>This pattern embodies the principle of “breaking down complex tasks into simple steps.”
Applicable scenarios:</p>
<ul>
<li>The task itself has clear, sequential steps.</li>
<li>Willing to sacrifice a little latency for higher accuracy.</li>
<li>Each step depends on the output of the previous step.</li>
</ul>
<p><img alt="" src="https://go-kratos.dev/images/blades/chain-workflow.png" /></p>
<p>Example code (examples/workflow-sequential):</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Sequential workflow, executing agents according to the arranged order</span></div></div><div><div><span>sequentialAgent</span><span> </span><span>:=</span><span> </span><span>flow</span><span>.</span><span>NewSequentialAgent</span><span>(flow.SequentialConfig{</span></div></div><div><div><span>    </span><span>Name</span><span>: </span><span>"</span><span>WritingReviewFlow</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>SubAgents</span><span>: []blades.Agent{</span></div></div><div><div><span>        </span><span>writerAgent</span><span>,</span></div></div><div><div><span>        </span><span>reviewerAgent</span><span>,</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>})</span></div></div></code></pre></figure></div>
<div><h2 id="pattern-2-parallelization-workflow">Pattern 2: Parallelization Workflow</h2></div>
<p>This pattern is used to have the LLM process multiple subtasks simultaneously and then aggregate the results.</p>
<p>Applicable scenarios:</p>
<ul>
<li>Need to process a large number of “similar but independent” items.</li>
<li>The task requires multiple different perspectives.</li>
<li>Time-sensitive and the task can be parallelized.</li>
</ul>
<p><img alt="" src="https://go-kratos.dev/images/blades/parallel-workflow.png" /></p>
<p>Example code (examples/workflow-parallel):</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Parallel workflow, storing generated results in the session state for reference by subsequent processes</span></div></div><div><div><span>parallelAgent</span><span> </span><span>:=</span><span> </span><span>flow</span><span>.</span><span>NewParallelAgent</span><span>(flow.ParallelConfig{</span></div></div><div><div><span>    </span><span>Name</span><span>:        </span><span>"</span><span>EditorParallelAgent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>Description</span><span>: </span><span>"</span><span>Edits the drafted paragraph in parallel for grammar and style.</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>SubAgents</span><span>: []blades.Agent{</span></div></div><div><div><span>        </span><span>editorAgent1</span><span>,</span></div></div><div><div><span>        </span><span>editorAgent2</span><span>,</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>})</span></div></div><div><div><span>// Define a sequential workflow, incorporating the parallel definition</span></div></div><div><div><span>sequentialAgent</span><span> </span><span>:=</span><span> </span><span>flow</span><span>.</span><span>NewSequentialAgent</span><span>(flow.SequentialConfig{</span></div></div><div><div><span>    </span><span>Name</span><span>:        </span><span>"</span><span>WritingSequenceAgent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>Description</span><span>: </span><span>"</span><span>Drafts, edits, and reviews a paragraph about climate change.</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>SubAgents</span><span>: []blades.Agent{</span></div></div><div><div><span>        </span><span>writerAgent</span><span>,</span></div></div><div><div><span>        </span><span>parallelAgent</span><span>,</span></div></div><div><div><span>        </span><span>reviewerAgent</span><span>,</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>})</span></div></div></code></pre></figure></div>
<p>This pattern can significantly improve throughput, but be mindful of the resource consumption and complexity introduced by parallelism.</p>
<div><h2 id="pattern-3-routing-workflow">Pattern 3: Routing Workflow</h2></div>
<p>This pattern uses the LLM to intelligently judge the input type and then dispatches it to different processing flows.
Applicable scenarios:</p>
<ul>
<li>Multiple input categories with significant structural differences.</li>
<li>Different input types require specialized processing flows.</li>
<li>High classification accuracy is achievable.</li>
</ul>
<p><img alt="" src="https://go-kratos.dev/images/blades/routing-workflow.png" /></p>
<p>Example code (examples/workflow-routing):</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Automatically selects the appropriate expert agent based on the Agent's description</span></div></div><div><div><span>agent</span><span>, </span><span>err</span><span> </span><span>:=</span><span> </span><span>flow</span><span>.</span><span>NewRoutingAgent</span><span>(flow.RoutingConfig{</span></div></div><div><div><span>    </span><span>Name</span><span>:        </span><span>"</span><span>TriageAgent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>Description</span><span>: </span><span>"</span><span>You determine which agent to use based on the user's homework question</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>Model</span><span>:       </span><span>model</span><span>,</span></div></div><div><div><span>    </span><span>SubAgents</span><span>: []blades.Agent{</span></div></div><div><div><span>        </span><span>mathTutorAgent</span><span>,</span></div></div><div><div><span>        </span><span>historyTutorAgent</span><span>,</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>})</span></div></div></code></pre></figure></div>
<div><h2 id="pattern-4-orchestrator-workers">Pattern 4: Orchestrator-Workers</h2></div>
<p>This pattern combines the “Agent” tendency: a central LLM acts as a task decomposer (orchestrator), and different “Workers” execute the subtasks.</p>
<p>Applicable scenarios:</p>
<ul>
<li>Cannot fully predict subtasks in advance.</li>
<li>The task requires multiple perspectives or processing methods.</li>
<li>Requires system adaptability and complex decision-making processes.</li>
</ul>
<p><img alt="" src="https://go-kratos.dev/images/blades/orchestrator-workers.png" /></p>
<p>Example code (examples/workflow-orchestrator):</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Define translators via tools (Agent as a Tool)</span></div></div><div><div><span>translatorWorkers</span><span> </span><span>:=</span><span> </span><span>createTranslatorWorkers</span><span>(</span><span>model</span><span>)</span></div></div><div><div><span>// The orchestrator selects and executes the required tools</span></div></div><div><div><span>orchestratorAgent</span><span>, </span><span>err</span><span> </span><span>:=</span><span> </span><span>blades</span><span>.</span><span>NewAgent</span><span>(</span></div></div><div><div><span>    </span><span>"</span><span>orchestrator_agent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithInstruction</span><span>(</span><span>`</span><span>You are a translation agent. You use the tools given to you to translate.</span></div></div><div><div><span><span>    </span></span><span>If asked for multiple translations, you call the relevant tools in order.</span></div></div><div><div><span><span>    </span></span><span>You never translate on your own, you always use the provided tools.</span><span>`</span><span>),</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithModel</span><span>(</span><span>model</span><span>),</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithTools</span><span>(</span><span>translatorWorkers</span><span>...</span><span>),</span></div></div><div><div><span>)</span></div></div><div><div><span>// Synthesize the multiple generated results</span></div></div><div><div><span>synthesizerAgent</span><span>, </span><span>err</span><span> </span><span>:=</span><span> </span><span>blades</span><span>.</span><span>NewAgent</span><span>(</span></div></div><div><div><span>    </span><span>"</span><span>synthesizer_agent</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithInstruction</span><span>(</span><span>"</span><span>You inspect translations, correct them if needed, and produce a final concatenated response.</span><span>"</span><span>),</span></div></div><div><div><span>    </span><span>blades</span><span>.</span><span>WithModel</span><span>(</span><span>model</span><span>),</span></div></div><div><div><span>)</span></div></div></code></pre></figure></div>
<div><h2 id="pattern-5-evaluator-optimizer">Pattern 5: Evaluator-Optimizer</h2></div>
<p>In this pattern, one model generates output, and another model evaluates that output and provides feedback, mimicking the human “write then revise” process.
Applicable scenarios:</p>
<ul>
<li>There are clear, quantifiable evaluation criteria.</li>
<li>Quality can be significantly improved through multiple rounds of “generate → evaluate → improve”.</li>
<li>The task is suitable for iterative refinement.</li>
</ul>
<p><img alt="" src="https://go-kratos.dev/images/blades/evaluator-optimizer.png" /></p>
<p>Example code (examples/workflow-loop):</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>// Iterates multiple times by generating content and then evaluating the effect</span></div></div><div><div><span>loopAgent</span><span> </span><span>:=</span><span> </span><span>flow</span><span>.</span><span>NewLoopAgent</span><span>(flow.LoopConfig{</span></div></div><div><div><span>    </span><span>Name</span><span>:          </span><span>"</span><span>WritingReviewFlow</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>Description</span><span>:   </span><span>"</span><span>An agent that loops between writing and reviewing until the draft is good.</span><span>"</span><span>,</span></div></div><div><div><span>    </span><span>MaxIterations</span><span>: </span><span>3</span><span>,</span></div></div><div><div><span>    </span><span>Condition</span><span>: </span><span>func</span><span><span>(</span><span>ctx</span><span> context.Context, </span><span>output</span><span> </span></span><span>*</span><span>blades.Message) (</span><span>bool</span><span>, </span><span>error</span><span>) {</span></div></div><div><div><span>        </span><span>// Evaluate content effectiveness to determine whether to end the iteration</span></div></div><div><div><span>        </span><span>return</span><span> </span><span>!</span><span>strings</span><span>.</span><span>Contains</span><span>(</span><span>output</span><span>.</span><span>Text</span><span>(), </span><span>"</span><span>The draft is good</span><span>"</span><span>), </span><span>nil</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>    </span><span>SubAgents</span><span>: []blades.Agent{</span></div></div><div><div><span>        </span><span>writerAgent</span><span>,</span></div></div><div><div><span>        </span><span>reviewerAgent</span><span>,</span></div></div><div><div><span><span>    </span></span><span>},</span></div></div><div><div><span>})</span></div></div></code></pre></figure></div>
<div><h2 id="best-practices-and-recommendations">Best Practices and Recommendations</h2></div>
<p>In business practice, whether for a single agent or a multi-agent architecture, engineering design often determines the final outcome more than model capability.</p>
<p>The following practical recommendations summarize the most critical principles from real projects and can serve as a reference when designing and implementing agents.</p>
<p>Start Simple</p>
<ul>
<li>Build a basic workflow first, then consider more complex agents.</li>
<li>Use the simplest pattern that meets the requirements.</li>
</ul>
<p>Design for Reliability</p>
<ul>
<li>Define clear error handling mechanisms.</li>
<li>Strive to use type-safe responses.</li>
<li>Add validation at every step.</li>
</ul>
<p>Trade-offs</p>
<ul>
<li>Balance latency against accuracy.</li>
<li>Evaluate the scenario before deciding on parallelism.</li>
</ul>
<div><h2 id="reference">Reference</h2></div>
<p><a href="https://github.com/go-kratos/blades/blob/main/examples">https://github.com/go-kratos/blades/blob/main/examples</a></p>
