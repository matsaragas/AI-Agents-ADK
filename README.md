# AI-Agents-ADK
End-to-end Agentic AI Framework using Gemini ADK: models, tools, orchestration, multi-agents, memory and evaluation  



This is course provided by [Kaggle](https://www.kaggle.com/whitepaper-introduction-to-agents). The Key concepts of the Agentic
AI framework that we will be using is described in the [white paper](https://www.kaggle.com/whitepaper-introduction-to-agents).

The course covers the following:

### Day 1: Introduction 

Part 1: ADK Definition
- Define an agent using Google ADK, 
- specify the model available for the agent 
- connect the agent to the tools

Part 2: Multi-agent systems & workflows
- Multi-agent team: Define agents with specific output, Root coordinator to bring agents together as tools and instructions 
  on how to coordinate the agents,
  
- Sequential Workflows - Assembly Line: use `SequentialAgent` to connect and run sub-agents in a pre-specified order.
- Parallel Workflows - Independent RResearchers: Use `ParallelAgent` to execute sub-agents 
  concurrently and speed up the workflow. Aggregate the results using an aggregator agent. 
  
```python

aggregator_agent = Agent(
  name="AggregatorAgent",
  model=Gemini(
    model="gemini-2.5-flash-lite",
    retry_options=retry_config
  ),
  # It uses placeholders to inject the outputs from the parallel agents, which are now in the session state.
  instruction="""Combine these three research findings into a single executive summary:

    **Technology Trends:**
    {tech_research}
    
    **Health Breakthroughs:**
    {health_research}
    
    **Finance Innovations:**
    {finance_research}
    
    Your summary should highlight common themes, surprising connections, and the most important key takeaways from all three reports. The final summary should be around 200 words.""",
  output_key="executive_summary",  # This will be the final output of the entire system.
)

print("✅ aggregator_agent created.")

parallel_research_team = ParallelAgent(
    name="ParallelResearchTeam",
    sub_agents=[tech_researcher, health_researcher, finance_researcher],
)

# This SequentialAgent defines the high-level workflow: run the parallel team first, then run the aggregator.
root_agent = SequentialAgent(
    name="ResearchSystem",
    sub_agents=[parallel_research_team, aggregator_agent],
)

print("✅ Parallel and Sequential Agents created.")


runner = InMemoryRunner(agent=root_agent)
response = await runner.run_debug(
  "Run the daily executive briefing on Tech, Health, and Finance"
)
```


- Loops workflows: Use `LoopAgent` to run a set of sub-agents repeatedly until a condition is met 
  or a number of iterations is reached.

### Day 2: Agent Tools

Part 1: Agent Tools

- Define customized tools and make it available to Agents.
- Improve agents Reliability with Code. Use An `LlmAgent` that uses `BuiltInCodeExecutor` 
- Custom vs Built-in Tools:
  * Custom:
    * Advantage: Complete Control
    * Function Tools: Python functions -> Turn any python function into an agent 
    * Long Running Tools: Functions for operations that take significant time (Human in the loop approvals).
    * Agent Tools: other agents used as tools
    * MCP tools: Filesystem access, Google Maps
    * OpenAI tools: REST API endpoints become callale tools
  
  * Buit-in Tools:
    * Gemini Tools: google_search, BuiltInCodeExecutor
    * Google Cloud Tools
    * Third-party Tools: Hugging Face, Github
  









The Key Ideas are described below:

An AI agent is an autonomous system that uses a language model in a continuous loop to achieve goals. It is built from four core components:

---

## Core Architecture

### 1. **Model — The Brain**
The language or foundation model (LM) serves as the agent’s central reasoning engine.

**Responsibilities:**
- Interprets instructions and context
- Reasons about problems and options
- Makes decisions and plans next actions

**Key Notes:**
- Can be general-purpose, fine-tuned, or multimodal
- The agent carefully curates the model’s context window to maximize performance

---

### 2. **Tools — The Hands**
Tools allow the agent to interact with the real world beyond text generation.

**Examples:**
- API calls
- Code execution functions
- Databases, vector stores, or search systems

**How it works:**
1. The model decides which tool to use
2. The orchestration layer executes the tool
3. Results are injected back into the model’s context

This enables agents to retrieve real-time data, take actions, and validate outcomes.

---

### 3. **Orchestration Layer — The Nervous System**
This layer governs the agent’s behavior and operational loop.

**Responsibilities:**
- Planning and task decomposition
- Memory and state management
- Reasoning strategy selection (e.g., ReAct, Chain-of-Thought)
- Deciding *when to think* vs *when to act*

It is also responsible for giving agents the ability to **remember** across steps and sessions.

---

### 4. **Deployment — The Body**
Deployment transforms an agent from a prototype into a reliable service.

**Includes:**
- Secure, scalable hosting
- Monitoring, logging, and observability
- Versioning and lifecycle management

Once deployed, agents can be accessed:
- Through user interfaces (chat, dashboards)
- Programmatically via **Agent-to-Agent (A2A) APIs**




## The agentic Problem-Solving Process

#### 1. **Get the Mission**

This is provided by the user (e.g., "Organize my team's travel for the 
upcoming conference")

#### 2. **Scan the Scene**

The agent perceives its environment to gather context. This involves the orchestration layer accessing its available resources:
"What does the user wants?" "What information is in my memory?", "Did I already try to do this task?", "Did the user gave me feedback?", 
"What can I access from my tools, like calendars, DBs, APIs?"

#### 3. **Think it Through**

This is the agent's core "think" loop, driven by the reasoning model. The agent analyses the Mission (step 1) against the
scene and devices a plan. 

#### 4. **Take Action**
The orchestrator layer executes the first concrete step of the plan. It selects and invokes the appropriate tool.

#### 5. **Observe and iterate**
The agent observes the outcome of its action. The nea info is added to the agent's context or "memory". The tool then repeats to step 3. 


##  A taxonomy of Agentic Systems


## Agent Ops: A structured Approach to the Unpredictable

As we build agents, we usually manually test the behavior. Agentic systems require new ops philosophy, as we transition from traditional, deterministic software to stochastic.



## CORE Agent Architectures: Model, Tools, and Orchestration

#### Model: The brain of the agent 

The model is the agent’s brain: 

* It drives reasoning, tool use, cost, and speed—benchmark scores alone don’t predict real-world success.

* Optimize for the task, not the leaderboard: Evaluate models against your actual business problem (e.g., your codebase, documents, workflows), balancing quality, latency, and cost.

* Use multiple models when needed: Route complex reasoning to powerful models and simpler, high-volume tasks to faster, cheaper ones to improve efficiency.

* Choose the right approach for multimodality: Use multimodal models for simplicity or specialized tools (vision, speech) plus text models for flexibility—at the cost of added complexity.

* Plan for constant change: Models evolve rapidly, so build strong “Agent Ops” with continuous evaluation and CI/CD to 
  upgrade models without reworking your entire system.


#### Tools: The hands of your AI agent

* Tools available to retrieve real-time information and take action in the world. 
  A robust tool interface is a three-part loop: defining what a tool
  can do, invoking it, and observing the result


* Retrieving Information: Grounding in Reality: Tools such as RAG, retrieve info from Vector DB, Knowledge Graphs

* Executing Actions: The true power of agents is unleashed when they move from reading information to actively doing things. 
By wrapping existing APIs and code functions as tools, an agent can an email, schedule a meeting, or update a customer record in ServiceNow. For more dynamic tasks, an agent can 
  write and execute code on the fly. This also includes tools for human interaction. An Agent can use a Human in the loop 
  tool to pause its workflow and ask for confirmation or request specific info from a user interface. 
  
* Function Calling: Connecting Tools to your agent.
For an agent to reliably do "function calling" and use tools, it needs clear instructions, secure connections and orchestration. For simpler discovery and connections to 
  tools, open standards like the Model Context Protocol (MCP) have become popular because they are more convenient

#### The Orchestration Layer

If the model is the agent's brain and the tools are its hands, the orchestration is the central nervous system that
connects them. It is the engine that runs the "Think, Act, Observe" loop, the state machine that governs 
the agent's behavior, and the place where a developer's carefully crafted logic comes to life. 