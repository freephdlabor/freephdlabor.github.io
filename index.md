# `freephdlabor`: customizing your own research lab to do scientific research in your field 24/7

<p align="center">
  <a href="https://github.com/ltjed/freephdlabor">
    <img src="https://img.shields.io/badge/GitHub-Repository-black?style=for-the-badge&logo=github" alt="GitHub Repository">
  </a>
  <a href="https://github.com/ltjed/freephdlabor">
    <img src="https://img.shields.io/github/stars/ltjed/freephdlabor?style=for-the-badge" alt="GitHub Stars">
  </a>
  <a href="https://github.com/ltjed/freephdlabor/blob/main/TR/technical_report/paper.pdf">
    <img src="https://img.shields.io/badge/Paper-Technical_Report-blue?style=for-the-badge&logo=arxiv" alt="Technical Report">
  </a>
</p>

**TLDR:** An "one-size-fits-all" agentic system for all scientific domains/use cases is unfortunately not yet possible. However, `freephdlabor` is the ***first fully customizable*** open-source multiagent framework with *dynamic workflow* to continually automate *all* stages of research (idea conception to latex-formatted papers with figures and citations). The modular design, support features, and empirical guidelines together allow you to **build, test, and ship your multiagent system tailored to your own domain/use case within hours**.

You can customize your own `freephdlabor`, allowing you to give it half-baked research hypotheses (e.g., *"can we predict neural network training phases?"*) in the evening and wake up to a full paper with **real** experiments and citations:

<embed src="papers/example_paper.pdf" type="application/pdf" width="100%" height="600px" />

**We can also see it in action:**

<iframe width="100%" height="600" src="https://www.youtube.com/embed/xwG7dpUtues" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

A great weekend project would be to **customize our system for yourself**—once running, you'll have extra brains (or entire research teams) **working alongside you 24/7** on the same problems you care about: [github.com/ltjed/freephdlabor](https://github.com/ltjed/freephdlabor)

---
## Generating useful scientific slop

The most exciting goal in AI today is arguably to **accelerate scientific progress** (perhaps aside from direct self-improvement). A good first step towards such acceleration would be agentic systems that work 24/7 on your research problems—testing hypotheses while you sleep, exploring dead-ends so you don't have to, and delivering publication-grade reports that you can provide feedback on, much like directing your own research lab full of cheap PhD labor.

Despite recent work has shown exciting promise, such a dream assistant remains out of reach. No existing system can flexibly adapt to any scientific subdomain while autonomously managing its context for sustained, long-term research programs.

To get closer to this vision, we need to address two fundamental challenges:

**Modularity.** A pipeline designed for ML experiments with a mandatory round of ablation studies can't be easily adapted to, say, materials science. To democratize AI research assistant systems to more people, we need **customizable, modular systems** where you can swap in domain-specific tools and agents without rebuilding from scratch.

**Context Engineering.** Long-running autonomous research generates massive amounts of information—experiment results, literature notes, failed attempts, insights. As agents work over hours or days, they need the right information at the right time in their context window. Without proper context management, agents either suffer from context bloat or miss critical details.

In this blog, we will dive into how `freephdlabor` tackles these two challenges, enabling you to build a customized AI research system for own research needs. We focus on key design principles and intuitions; for complete implementation and the full set of features, please refer to our [technical report](https://github.com/ltjed/freephdlabor/blob/main/TR/technical_report/paper.pdf).

## Challenge 1: Modularity

Over the past year, systems like `AI Scientist`[^1], `AI Scientist-v2`[^2], `Agent Laboratory`[^3], and `Zochi`[^4] have demonstrated automated research in specific domains. However, these systems employ **fixed workflows**—operating like assembly lines that impose the same sequence of steps on all research topics (one exception is Google's `AI co-scientist`[^5], which allocate resources to different tasks/agents a priori, but it was never made open-source). While fixed workflows reduce variability and make systems less prone to errors, they prevent customization: a pipeline designed for ML experiments can't easily adapt to your specific research area without significant re-engineering.

Building a customizable multiagent system requires clean interfaces and modular design. `freephdlabor` addresses this through these key mechanisms:

### An Example Architecture

<img src="figures/architecture.png" alt="freephdlabor Architecture" width="800">

<!-- *Figure 1: **Multi-Agent System Architecture**. The ManagerAgent serves as the central coordinator, delegating tasks to specialized agents (IdeationAgent, ExperimentationAgent, WriteupAgent, ReviewerAgent) and managing communication through a shared workspace.* -->

### ManagerAgent - PI of a Research Lab

From an individual agent's perspective, it needs to keep the following information in its context to decide which agent to communicate with at any given time: **(a)** the entire research history (ideas attempted, experiment results, etc.) accumulated thus far, and **(b)** complete description of the environment, which includes the description of every other agent. The total context this information requires grows **quadratically** with the number of agents.

For this reason, in our example system we designate a **ManagerAgent** to handle the delegation. This is the only agent that keeps track of both (a) and (b) mentioned above. Mechanistically, delegation to other agents is integrated much like a tool: both tools and agents have a description detailing their purpose, capabilities, etc., which are included as part of the ManagerAgent's system prompt.

<img src="figures/decision.png" alt="Dynamic Decision Making" width="600">

<!-- *Figure 3: **Dynamic Agent Decision-Making in freephdlabor**. When encountering a limitation in the current research context, the system's ManagerAgent autonomously reasons about the appropriate response and decides whether to delegate to specialized agents, interact with the workspace, or call other tools. This dynamic decision-making enables adaptive research workflows that respond to real-time progress.* -->

Thus, delegating to an agent is as simple as calling a tool with instructions as a parameter. The delegated agent will start a run of its own, call a variable number of tools to achieve the goal specified in its system prompt plus instructions from ManagerAgent, and call the `final_answer` tool when ready to report back to ManagerAgent, with the report passed as an argument to the `final_answer` tool.

This *hub-and-spoke* design also makes the design more *modular*: the central ManagerAgent functions as an intelligent 'adapter' that requires user to readjust any agent twice (i.e., it can receives enough info from ManagerAgent to perform its job AND reports results to the same agent effectively) rather than having to repeat so for all other agents. This vastly decreases the amount of trial-and-error to, say, integrate a new agent into freephdlabor.

### Auto Agent Optimization

Another goal of freephdlabor is to enable everyone to easily customize their own multiagent system for bespoke use cases. Customization typically involves:

1. **Defining agent(s)**:
   - (a) Writing a system prompt
   - (b) Defining the tools the agent has access to

2. **Ensuring integration**:
   - (a) Agent receives necessary information from other agents
   - (b) Agent faithfully and effectively communicates its work

We want to allow users to zoom in on (1), especially (1)(b). To make (2) easier, **freephdlabor automatically tracks all LLM calls** made by all agents, organized in temporal order, in `agent_llm_calls.jsonl`. As recent research indicates[^6][^7], systematically analyzing `agent_llm_calls.jsonl` (especially across different runs) can enable a coding assistant, specialized agent, or fine-tuned LLM like AgentTracer-8B[^7] to identify points for improvement.

E.g., users can use these **Claude Code slash commands**:
- `/analyze_agent_context` - Helps ensure agents receive necessary information 
- `/refine_agent_prompt` - Helps improve agent communication effectiveness

At the moment, suggested improvements center around system prompts, but in the future, with better context engineering and coding assistants, we plan to support more general improvements involving code changes.

### Workspace - File-Based Communication Channel/External Memory

Allowing agents to communicate through a single `string` at a time is problematic for many reasons, the biggest of which is that it introduces the **"game of telephone"** effect, where an agent needs to transcribe information one or more times before another agent can access it.

<img src="figures/game_of_telephone.png" alt="Game of Telephone Problem" width="650">

A much better alternative is to **write important information as files** inside a shared workspace folder, communicating only the file path (or even better, with a brief summary of its content) to another agent. As an added bonus, these files can serve as references to return to as needed in the future. It is paramount to give files descriptive names—lengthy names are perfectly acceptable if they enhance clarity.

## Challenge 2: Context Engineering

LLMs are pure functions—without tuning hyperparameters like temperature, their outputs depend entirely on what's in the context window. Effective long-term autonomous operation requires managing both context length and ensuring that the right information is available at the right time.

### Agent Memory

When an agent in freephdlabor runs, it reviews all past memories (the full conversation context), generates an action (like calling a tool or writing code), observes the results, and saves this entire cycle as another step in memory for future reference.

<img src="figures/memory.png" alt="Agent Memory Cycle" width="700">

*Agents maintain a growing memory of reasoning, actions, and observations from each step*

This means the agent's context includes not just the current task, but the complete history of reasoning, actions, and observations from previous steps. The framework handles memory persistence, step replay, and secure code execution environments automatically[^8]. While this memory-based approach enables sophisticated multi-step reasoning, it also means context windows can grow large over time—which is why we implement multiple context management strategies.

### Context Compaction

Context compaction handles growing conversations *within* a single session. When tokens exceed **75% of the model's limit**, the `AutomaticContextCompactor` kicks in: it intelligently summarizes the context—tool usage patterns, key observations, recent reasoning, errors encountered—and reconstructs the agent's memory with this compact summary **plus the last 3 ActionSteps**.

### Towards continual collaboration

An important goal of `freephdlabor` is to enable **continual research** of the same topic while allowing plenty of interaction between you and your own 'research lab.'

To this end, the system automatically saves the complete memory of all agents—every execution step with detailed reasoning traces, tool usage history, and inter-agent interactions. When combined with workspace files, this creates a comprehensive record of the entire research trajectory. **Resuming from checkpoints** is simple: specify the workspace you wish to continue from, and the system reconstructs the entire multiagent environment from the saved state.

**Real-time interruption** enables interactive context injection when needed. The system listens continuously for user input signals. After each step completes, if an interrupt is present, the agent pauses and prompts you for new instructions—either to refine the current task or inject critical context the agent may have missed. These instructions are stored in agent memory and incorporated into subsequent steps.

This creates a **collaborative loop** where agents remain self-directed most of the time, yet always receptive to human guidance when needed.

### Workspace as External Memory

The workspace folder also addresses context limitations by serving as **external memory that doesn't consume tokens**. Important information—experiment results, intermediate analyses, literature reviews—lives in files rather than taking up precious context space. Agents can reference these files when needed, dramatically extending their effective memory capacity beyond the token limit when workspace in organized appropriately.

With context compaction, memory persistence, and workspace-based external memory working together, you finally have free PhD labor that works 24/7 on topics of your interest—running experiments, generating reports, and most importantly, **building on previous lessons learned**.

## Future Directions

**Agent Deception**: Agents sometimes engage in deceptive behavior when faced with difficult requirements they cannot satisfy. For example, when asked to generate a paper with specific length requirements, agents may create "placeholder" content consisting mostly of gibberish rather than admitting inability to meet requirements. Moreover, as seen in the demo video run for ResourcePreparationAgent in the first run, sometimes agents may just make a mistake and don't realize it. Regardless, improving reliability via dedicated agents, prompting, or post-training are all viable options.

**Adapting to Your Domain**: The most direct extension of freephdlabor is modifying existing agents for your specific use case. For instance, if you're a materials scientist, you could substitute the `RunExperimentTool` (designed for AI/ML experiments) with a tool that takes in a hypothesis and outputs lab experiment results. Resources like `ToolUniverse`[^9] provide curated collections of validated tools that can be seamlessly integrated into agent definitions for domain-specific customization.

**Context Engineering Benefits**: A commonly stated advantage of multi-agent systems is specialization via system prompts. Through building freephdlabor, we've discovered that **delegation of certain tasks to other agents can significantly reduce the burden on individual context windows**, enabling more sophisticated reasoning chains. As context engineering capabilities improve, this architectural benefit becomes increasingly valuable.

**In-Context Learning for Improvement**: Currently, the most straightforward way for agents in freephdlabor to "learn" between different runs is through in-context learning—incorporating information into system prompts or as files in the workspace upon initialization. While effective, this approach has drawbacks: the information takes up precious context window space and can distract agents when tasks are unrelated to saved information.

**Specialization via Fine-Tuning**: We believe an underappreciated advantage of the multiagent approach is **specialization via fine-tuning**. The major bottleneck in traditional fine-tuning lies in the amount of data/capability we can post-train into each LLM without interfering with other capabilities.

Since `agent_llm_calls.jsonl` contains the LLM calls (i.e., state-action pairs) of different agents across runs, it would be fascinating to **fine-tune individual agents using curated versions of those trajectories**. This approach could enable deep domain-specific expertise while maintaining general capabilities—each agent becomes a specialist through targeted fine-tuning on its own behavioral data.

### Flexibility v. Stability

The primary trade-off in designing freephdlabor is between **flexibility** and **stability**. Fully agentic workflows enable adaptive research processes but can sometimes lead to unpredictable behavior. We address this through:

1. **Structured workspace conventions** that guide agent behavior
2. **Context compaction** to prevent information overload
3. **Explicit quality gates** in the ManagerAgent's delegation logic
4. **Human-in-the-loop interruption mechanisms** for course correction

As foundation models continue to improve, we expect the stability-flexibility trade-off to become increasingly favorable for agentic systems.

<img src="figures/example_run_mermaid.svg" alt="Example Research Run" width="100%">

*Sequence diagram demonstrating how information flows in an example `freephdlabor` run*

---

## Taking stock...

*AI-accelerated innovation shouldn't belong only to Google*. `freephdlabor` is a step toward this vision: a multiagent framework where **modularity enables customization** and **context engineering enables persistence**, further *closing the gap between compute hours and scientific innovation*.

Ready to build your own AI research team? Check out:
- **GitHub Repository**: [https://github.com/ltjed/freephdlabor](https://github.com/ltjed/freephdlabor)
- **Full Technical Report**: [PDF](https://github.com/ltjed/freephdlabor/blob/main/TR/technical_report/paper.pdf)

We welcome contributions, feedback, and discussions. Join us in democratizing AI-powered scientific discovery!

---

## References

[^1]: Lu, C., et al. (2024). *AI Scientist: Fully Automated Scientific Discovery*. arXiv preprint arXiv:2408.06292. [https://arxiv.org/abs/2408.06292](https://arxiv.org/abs/2408.06292)

[^2]: Yamada, Y., Lange, R. T., Lu, C., Hu, S., Lu, C., Foerster, J., Clune, J., & Ha, D. (2025). *The AI Scientist-v2: Workshop-Level Automated Scientific Discovery via Agentic Tree Search*. arXiv preprint arXiv:2504.08066. [https://arxiv.org/abs/2504.08066](https://arxiv.org/abs/2504.08066)

[^3]: Schmidgall, S., Su, Y., Wang, Z., Sun, X., Wu, J., Yu, X., Liu, J., Moor, M., Liu, Z., & Barsoum, E. (2025). *Agent Laboratory: Using LLM Agents as Research Assistants*. arXiv preprint arXiv:2501.04227. [https://arxiv.org/abs/2501.04227](https://arxiv.org/abs/2501.04227)

[^4]: Zhou, Y., et al. (2025). *Zochi: Technical Report on Automated Scientific Research*. [https://github.com/IntologyAI/Zochi/blob/main/Zochi_Technical_Report.pdf](https://github.com/IntologyAI/Zochi/blob/main/Zochi_Technical_Report.pdf)

[^5]: Gottweis, J., Weng, W.-H., Daryin, A., Tu, T., Palepu, A., Sirkovic, P., et al. (2025). *Towards an AI co-scientist*. arXiv preprint arXiv:2502.18864. [https://arxiv.org/abs/2502.18864](https://arxiv.org/abs/2502.18864)

[^6]: Agrawal, P., et al. (2025). *GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning*. arXiv preprint arXiv:2507.19457. [https://arxiv.org/abs/2507.19457](https://arxiv.org/abs/2507.19457)

[^7]: Zhang, Y., et al. (2025). *AgenTracer: Who Is Inducing Failure in the LLM Agentic Systems?* arXiv preprint arXiv:2509.03312. [https://arxiv.org/abs/2509.03312](https://arxiv.org/abs/2509.03312)

[^8]: HuggingFace. (2024). *smolagents Documentation*. [https://huggingface.co/docs/smolagents](https://huggingface.co/docs/smolagents)

[^9]: Gao, J., et al. (2025). *Democratizing AI scientists using ToolUniverse*. arXiv preprint arXiv:2509.23426. [https://arxiv.org/abs/2509.23426](https://arxiv.org/abs/2509.23426)
