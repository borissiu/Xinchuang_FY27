<div class="section" id="inline-implementation">
<h1>Inline implementation<a class="headerlink currentURL" href="#inline-implementation" title="Permalink to this heading">¶</a></h1>
<p>The first implementation method we are going to explore is <strong>inline</strong>.</p>
<p><strong>F5 AI Guardrails</strong> sits in the inference path and enforces policies in real time, forwarding only clean prompts/responses.</p>
<p>The HTTPS connection will be terminated by <strong>F5 AI Guardrails</strong> and recreated toward the backend inference endpoint.</p>
<p>The traffic reaching the <strong>F5 AI Guardrails</strong> endpoint needs to conform to one of the following specs:</p>
<ol class="arabic simple">
<li>The <strong>F5 AI Guardrails</strong> API spec detailed here <a href="https://docs.calypsoai.com/operations/post_prompts.html" target="_blank" class="externalLink">https://docs.calypsoai.com/operations/post_prompts.html</a> or by using the F5 AI Guardrails Python SDK <a href="https://docs.calypsoai.com/api-docs/sending-prompt-specific-provider.html" target="_blank" class="externalLink">https://docs.calypsoai.com/api-docs/sending-prompt-specific-provider.html</a></li>
<li><strong>OpenAI chat completions</strong> spec, more info can be found here <a href="https://docs.calypsoai.com/api-docs/openai-compatibility.html" target="_blank" class="externalLink">https://docs.calypsoai.com/api-docs/openai-compatibility.html</a></li>
</ol>
<p>Once the request is received, <strong>F5 AI Guardrails</strong> can transform the request to any inference spec from OpenAI, Ollama, Hugging Face, and more.</p>
<p><strong>Traffic Flow</strong></p>
<ol class="arabic simple">
<li>The user sends a prompt to the orchestrator.</li>
<li>The orchestrator builds the full context and sends the API call to <strong>F5 AI Guardrails</strong>.</li>
<li><strong>F5 AI Guardrails</strong> scans the prompt and, if all is good, forwards it to the LLM inference endpoint.
The request will also be transformed when forwarded to the configured inference spec.</li>
<li>The LLM responds and sends the response to <strong>F5 AI Guardrails</strong>.</li>
<li><strong>F5 AI Guardrails</strong> scans the response and, if all is good, forwards it to the orchestrator.
The response will be transformed to the client-side API spec.</li>
<li>The orchestrator replies to the user.</li>
</ol>
<img alt="../../../_images/inline.PNG" class="align-center" src="../../../_images/inline.PNG">
</div>
