Here's a condensed summary of the key points from the webinar:

# 1. Agentic AI systems are different from traditional CRUD applications, being conversation-centered rather than transaction-centered.

# 2. There's growing interest in agentic AI, with search terms increasing 15,000% recently.

# 3. Main challenges include:
   - High latency compared to traditional databases
   - Cost (10-10,000x more expensive per call than database operations)
   - Managing conversational history and context
   - Integrating with existing SAS systems

# 4. AA (the company hosting the webinar) offers a platform with:
   - Streaming endpoints and multimodal capabilities
   - Connectors for various databases and LLMs
   - Agent orchestration and workflow management
   - In-memory context database
   - Governance and lifecycle management tools

# 5. They claim to offer efficiencies through:
   - Non-blocking, asynchronous LLM adapters
   - Shared compute for API and agentic services
   - Multi-region support with automated replication

# 6. The company aims to help customers go from concept to production in 8 weeks.

# 7. For those just starting, they offer a proof-to-value program to create prototypes in 48 hours.

# 8. Deployment options include cloud, virtual private cloud, and self-hosted solutions.

# 9. Key considerations for implementation include latency management, cost optimization, and explainability of AI decisions. 

The webinar emphasized that while there are challenges, they're working to provide solutions to make agentic AI more accessible and efficient for enterprises.
# 1. The Agentic Opportunity and the move to a-tier Architecture
**1.1. AI Assistant**
A user app that understands natural language commans and uses a conversational AI interface to complete tasks on-demand.
**1.2. AI Agent**
A system that can autonomously fulfill goals by interacting with other systems and agents.
**1.3. AI Agency**
Capacity to make meaning from your environment
- **Low Agency**
   - static
   - reactive
   - tasks
   - supervised
- **RPA** 
   - text extract
   - open docs
   - consolidate data
- **Agents**
   - chatbots
   - lead qualification
   - task automation
   - suggestions
- **Agentic**
   - diagnostic
   - surveillance
   - IT support
   - cust. service
- **High Agency (Humans)**
   - adaptive
   - proactive
   - goals
   - autonomous
> A big gap exists between current LLM-based assistant and full-fledged AI agents, but this gap will close as we learn how to build, govern and trust agentic AI solutions.
> -Gartner
# 2. A Blueprint for Agentic Services
**A paradigm shift to AI-fueled app ecosystem**
**AI agents and apps become part of a symbiotic existence**
> By 2028, 33% of enterprise software applications will include agentic AI, up from less than 1% in 2024.

**App Ecosystem (SaaS Applications + Agentic AI Services)**: ->
- Enhanced user experience: AI agents personalize interactions to increase satisfaction
- Operational efficiency: AI agents automate routine tasks to allow humans to focus on strategic initiatives
- Scalability: AI-driven SaaS adapt to business needs without proportional increases in cost

**Agentic is the 5the wave of compute (100x trillions in users, 1,000,000 TPS)**.
Every human and device with dozens of sleepless assistants

**Transactional apps -> Conversational agents**. A fundamental shift from request-response to contextual iterations.
   - SaaS Applications: Stateless
     - Transactional: Database (Immediate, Structured, Concurrent, Durable)
   - Agentic Services: Orchestrated | Stateful
     - Historical: Event Store (Replayable, Traceable, Auditable, Event sources)
   - Agentic AI Services: Orchestrated | Stateful
     - Historical: Event Store
     - Conversational: LLMs (Stateless)
    
**Agents are orchestrated services**
   - Workflows:
      - traceable
      - auditable
      - debuggable
      - with point-in-time recovery
   - Agents are workflows
      - reliable execution of AI tasks with visibility into request / response data, built-in retries, and error compensation
      - trigger -> agent workflow: monitor, timeout, storage, policies, domain logic, plug-ins -> sequential, parallel, event-driven, state machine (rules: fave activities, budget, itinerary), human-in-the-loop (proposed itinerary, adjust dates, budget, plan) 
   - Task chaining
      - AI agents break comples workflows into smaller composable steps
      - trigger -> agent workflow: sequence, choice of agent, validation -> sub-task agent 1, sub-task agent 2, sub-task agent 3

**Agent types orchestrate levels of agency**. De-coupled, event-driven patterns and control loops.
   - Retrieve - augment: agent that combine external knowledge with reasoning and action
      - trigger -> agent workflow: knowledge retrieval, crawling fan-out, augment promt, reasoning -> repeat unti desired outcome -> output + action   
   - Environment controllers: control environments in real-time robotics, edge, and automation
      - streaming sensors -> agent workflow: perception, reasoning, action, feedback -> reapeat until goal achieved -> output + changed state
   - Self learning: agents that improve themselves over time through self-reflection and evnironment adaption
      - input query -> agent workflow: reasoning, action feedback, execution, learning, reasoning -> continuous iteration -> evolved agent
 **Conversations are stateful**. Context and conversation database now a part of the agentic stack
      - each iteration adds context
      - journaled sequences for context and recovery
      - in-memory, durable journals for speed + resilience

**Agentic AI augmentation cycle**
# 3. Agentic stories and AI in practice
## Real-time video augmentation, model-driven personalization, Google Earth AI inference 


