# Application Card: Troubleshooting Agent in Support and Troubleshoot

## 1. Overview

The Troubleshooting Agent is an AI-powered feature within the Azure portal's Support and Troubleshoot experience (Help + Support). It helps Azure customers resolve common issues faster by automatically analyzing reported problems, identifying root causes, and generating actionable mitigation steps.

When a customer has an issue with an Azure resource or needs help, they can describe their problem in the Help + Support experience. The Troubleshooting Agent then uses a two-step AI reasoning chain to analyze the issue and provide targeted guidance — all before a support ticket needs to be filed. The agent is designed to complement, not replace, human support engineers by offering AI-assisted troubleshooting as a first step.

The Troubleshooting Agent is available to Azure customers through the Azure portal's Help + Support pane. It is intended for enterprise customers and individual Azure users who need assistance diagnosing and resolving issues with their Azure resources.

## 2. Key terms

The following list provides a glossary of key terms related to the Troubleshooting Agent:

**A2A (Agent-to-Agent) Service**: The backend communication protocol used by the Troubleshooting Agent to send requests to and receive responses from the AI reasoning service. It uses a JSON-RPC 2.0 based messaging format.

**ARM Diagnostics**: Azure Resource Manager diagnostics — automated diagnostic checks that examine the state and configuration of Azure resources to identify potential issues.

**Grounding**: The process of ensuring that AI-generated content is based on verified source materials, such as Azure diagnostic data and knowledge base articles, rather than fabricated or unverified information.

**Mitigation**: A recommended action or set of steps to resolve an identified issue. Mitigations may include CLI commands for manual execution or REST API calls that can be executed with customer approval.

**Reasoning Chain**: The sequential two-step process used by the Troubleshooting Agent: first analyzing the root cause of an issue, then generating mitigation steps based on the identified root cause.

**Root Cause Analysis**: The first step of the reasoning chain, where the agent examines the customer's problem description, affected resource, and support topic to identify the underlying cause of the issue.

## 3. Key features or capabilities

The key features and capabilities outlined here describe what the Troubleshooting Agent is designed to do and how it performs across supported tasks.

- **Root cause analysis**: The agent examines the customer's issue summary, the affected Azure resource, and the selected support topic to identify potential root causes. It integrates multiple diagnostic sources including ARM diagnostics, self-help knowledge base articles, and compute troubleshooting tools to determine the most likely cause of the issue.

- **Mitigation step generation**: Based on the identified root cause, the agent generates targeted remediation steps. These may include CLI commands that the customer can copy and run manually, or REST API calls that the customer can execute directly by clicking an "Approve" button, which runs the API on the customer's behalf.

- **Multi-source diagnostic integration**: The agent draws from multiple validated data sources to provide comprehensive troubleshooting guidance, including Azure Resource Manager (ARM) diagnostics, self-help knowledge base articles, Azure compute troubleshooting plugins, and smart diagnostics.

- **Fallback guidance**: If the agent cannot determine a specific root cause or generate targeted mitigation steps, it provides general troubleshooting guidance relevant to the customer's issue area, ensuring the customer always receives some level of assistance.

- **Action safeguards**: Any automated mitigation actions (such as REST API calls) require explicit customer approval before execution. The agent cannot modify a customer's Azure environment without consent, and all actions respect existing Azure Role-Based Access Control (RBAC), Azure Policy, and resource locks.

The Troubleshooting Agent is considered Autonomous Agentic AI with a defined action space. It operates using a multi-agent architecture with two specialized agents:

1. **Problem Root Cause Agent** — Analyzes the issue and identifies root causes using diagnostic data.
2. **Mitigation Agent** — Generates and can execute remediation steps based on the root cause analysis.

Each agent has a bounded scope and cannot perform actions outside of its defined troubleshooting domain. The agent does not have memory across sessions and does not retain information from previous interactions.

## 4. Intended uses

The Troubleshooting Agent can be used in multiple scenarios across the Azure support experience. Some examples of use cases include:

- **Diagnosing Azure resource issues**: A customer experiences connectivity issues with an Azure Virtual Machine. They describe the problem in the Help + Support pane, and the Troubleshooting Agent identifies that the VM is in a stopped state, recommends restarting it, and provides the relevant CLI command. This reduces the time to resolution from hours (waiting for a support engineer) to minutes.

- **Automated mitigation for common problems**: A customer encounters a networking configuration error on their Azure resource. The Troubleshooting Agent identifies the root cause as a misconfigured Network Security Group rule and generates a REST API call to correct the configuration. The customer reviews the proposed change and clicks "Approve" to apply the fix directly from the support pane.

- **Self-service troubleshooting before ticket creation**: A customer navigating the support ticket creation flow is presented with AI-generated troubleshooting guidance specific to their issue. If the guidance resolves the problem, the customer avoids the need to file a support ticket entirely, reducing support costs and improving customer satisfaction.

The Troubleshooting Agent has a defined action space and set of boundaries. It is not a general-purpose assistant — it is scoped exclusively to Azure resource troubleshooting within the Help + Support experience.

## 5. Models and training data

The Troubleshooting Agent leverages AI models to power the experience that users see. The agent communicates with a backend AI reasoning service through the A2A (Agent-to-Agent) protocol. The reasoning service is powered by [GPT-4.1 mini](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models), deployed through Azure OpenAI Service.

To learn more about the data used to train the foundation models behind the Troubleshooting Agent, refer to the [Azure OpenAI Service transparency note](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/transparency-note).

The agent's responses are grounded in validated Azure diagnostic data sources rather than relying solely on model training data. These grounding sources include ARM diagnostics, self-help knowledge base articles, compute troubleshooting plugins, and smart diagnostics.

User-provided problem descriptions and the Troubleshooting Agent's responses are not used to train, retrain, or improve the foundation models that generate responses.

## 6. Performance

The Troubleshooting Agent is designed to perform reliably when customers report issues with Azure resources through the Help + Support experience in the Azure portal. It operates within the following conditions:

**Inputs**: The agent accepts text-based inputs including a problem summary (free-form text description of the issue), Azure product and support topic selection, resource ID, and subscription ID. These inputs are provided by the customer through the Help + Support user interface.

**Outputs**: The agent produces text-based outputs including root cause analysis summaries, step-by-step mitigation instructions, copyable CLI commands, and REST API call proposals. All outputs are rendered in markdown format within the support pane.

**Time constraints**: The agent operates within a three-minute window per analysis. If the root cause analysis and mitigation generation are not completed within this time, partial results or general troubleshooting guidance may be shown instead.

**Language support**: The Troubleshooting Agent is currently available in English only. We may expand language support in the future.

## 7. Limitations

Understanding the Troubleshooting Agent's limitations is crucial to determine if it is used within safe and effective boundaries. While we encourage customers to leverage the Troubleshooting Agent for resolving Azure resource issues, it's important to note that the Troubleshooting Agent was not designed for every possible scenario. We encourage users to refer to either the [Microsoft Enterprise AI Services Code of Conduct](https://learn.microsoft.com/en-us/legal/ai-code-of-conduct) (for organizations) or the [Code of Conduct section in the Microsoft Services Agreement](https://www.microsoft.com/servicesagreement) (for individuals) as well as the following considerations when choosing a use case:

- **AI-generated content may be inaccurate**: The Troubleshooting Agent generates responses using AI models, and while the outputs are grounded in Azure diagnostic data and knowledge base articles, they may not always be accurate or applicable to a specific situation. Users should always review suggested steps and use their judgment before making changes to their Azure environment. This is important because applying incorrect mitigation steps could lead to unintended changes in resource configuration.

- **Time constraints may limit analysis depth**: The agent operates within a three-minute window. If analysis is not completed within this time, partial results or general troubleshooting guidance may be shown instead. Complex multi-faceted issues may require more time than the agent allows, and users should consider filing a support ticket for in-depth analysis.

- **Root cause identification is not guaranteed**: For some issues, the agent may be unable to determine a specific root cause. This can occur when the issue involves complex interactions between multiple services, when diagnostic data is insufficient, or when the issue falls outside the agent's supported diagnostic sources. In these cases, the agent provides general solutions relevant to the issue area.

- **Limited to supported Azure services**: The agent's diagnostic capabilities depend on the integrations available for a specific Azure service. Not all services have the same level of diagnostic depth. Users should be aware that troubleshooting guidance may be less detailed for services with limited diagnostic integration.

- **English only**: The Troubleshooting Agent is currently available in English only. The experience is not available in other languages.

- **No conversational interaction**: The Troubleshooting Agent performs a one-pass analysis and does not support multi-turn conversational follow-ups. Users cannot ask clarifying questions or refine the agent's analysis through dialogue.

- **Not suitable for billing or account management**: The agent is designed exclusively for technical troubleshooting of Azure resources. It should not be used for billing inquiries, account management, or non-technical support topics.

## 8. Evaluations

Performance and safety evaluations assess whether AI applications are operating reliably and securely by examining factors like groundedness, relevance, and coherence while identifying the risks of generating harmful content. The following evaluations were conducted with safety components already in place, which are also described in [9. Safety Components and Mitigations](#9-safety-components-and-mitigations).

### 8.1 Performance and quality evaluations

Performance evaluations for AI applications are essential to improving their reliability in real-world applications. Metrics like groundedness, relevance, and coherence help assess the accuracy and consistency of AI-generated outputs, so that they are factually supported in grounded content scenarios, contextually appropriate, and logically structured. For the Troubleshooting Agent, we conducted performance evaluations using the following metrics:

- Relevance
- Coherence

#### 8.1a Performance and quality evaluation methods

The Troubleshooting Agent's quality is evaluated using the RatioAI Eval SDK with 12 parallel evaluators running against each test sample. Quality-specific evaluators include:

- **Confidence** — Measures how confident the model is in its response (score range: 1–5).
- **SelfJudge** — The model self-evaluates the quality of its own output (score range: 1–5).
- **Completeness** — Assesses whether the response fully addresses the input problem (score range: 1–5).
- **Helpfulness** — Evaluates whether the response provides actionable and useful guidance (score range: 1–5).
- **Text Quality (MsAI)** — Microsoft AI evaluator assessing the overall text quality of the response.
- **Relevance (MsAI)** — Microsoft AI evaluator assessing how relevant the response is to the input query.

Evaluations are run against all three sub-agents in the Troubleshooting Agent pipeline: Root Cause Agent, Solution Step Agent, and Auto Mitigation Agent. Each evaluation produces a detailed JSON report with per-evaluator scores. An ideal result is one where all quality evaluators score in the upper range (4–5 out of 5), indicating accurate, complete, and helpful troubleshooting guidance. A suboptimal result would score below 3, indicating the response may be incomplete, irrelevant, or not actionable.

### 8.2 Risk and safety evaluations

Evaluating potential risks associated with AI-generated content is essential for safeguarding against content risks with varying degrees of severity. For the Troubleshooting Agent, we conducted risk and safety evaluations for the following metrics:

- Protected material
- Indirect attack
- Content harm
- Code vulnerability

#### 8.2a Risk and safety evaluation methods

Safety evaluations are conducted in parallel alongside quality evaluations using the RatioAI Eval SDK and Microsoft AI safety evaluators. Safety-specific evaluators include:

- **Content Safety** — Evaluates whether the response contains harmful, offensive, or inappropriate content (score range: 0 or 5, where 0 indicates a safety concern).
- **Privacy** — Assesses whether the response inappropriately discloses or requests personally identifiable information.
- **Protected Material (MsAI)** — Microsoft AI evaluator that checks for protected or copyrighted content in the response.
- **Indirect Attack (MsAI)** — Microsoft AI evaluator that tests vulnerability to indirect prompt injection attacks.
- **Content Harm (MsAI)** — Microsoft AI evaluator assessing potential for harmful content generation.
- **Code Vulnerability (MsAI)** — Microsoft AI evaluator that checks generated code (CLI commands, API calls) for security vulnerabilities.

An ideal result is one where all safety evaluators indicate no safety concerns (score of 5 or "pass"). A suboptimal result would be any response flagged by a safety evaluator, which would indicate the response may contain harmful content, privacy violations, or security vulnerabilities.

Additionally, five context-dependent evaluators (Factuality, Groundedness, Hallucination, and MsAI WithContext/GroundTruth evaluators) have been prepared for future integration once reference data pipelines are established.

#### Evaluation data for quality and safety

Our evaluation data is custom-built to assess AI application performance across key areas of **safety** and **quality**, simulating real-world scenarios and risks. We begin by identifying relevant evaluation aspects of concern based on multi-disciplinary research and expert input. These concerns are translated into targeted evaluation objectives and guide formulation of evaluation metrics. For **safety**, we create adversarial prompts to elicit undesirable or edge-case responses, which are then scored using AI-assisted annotators trained to assess alignment with Microsoft's safety standards. For **quality**, we craft rubric-based prompts relevant to scenarios including evaluating retrieval-augmented generation (RAG) applications and agents. Datasets are curated from diverse sources including synthetic and public datasets to simulate real-world user scenarios. Using the curated datasets, both evaluations undergo iterative refinement and human alignment to improve metric efficacy and reliability. This methodology forms the foundation of repeatable, rigorous assessments that reflect how customers use evaluations to build better and safer AI.

### 8.3 Custom evaluations

In addition to the automated evaluations described above, the Troubleshooting Agent was evaluated through manual testing. This covered:

- **Accuracy of root cause identification** — Testing whether the agent correctly identifies known root causes for common Azure issues across supported services.
- **Quality of mitigation steps** — Assessing whether generated remediation steps are actionable, accurate, and relevant to the identified root cause.
- **Handling of harmful and malicious inputs** — Testing the agent's behavior when presented with adversarial or inappropriate prompts to ensure appropriate filtering and response.
- **Grounding validation** — Verifying that generated content is based on valid diagnostic sources and knowledge base articles, and filtering out ungrounded responses.
- **Timeout and failure behavior** — Ensuring graceful degradation when the agent cannot complete analysis within the three-minute time limit.

The Troubleshooting Agent is also continuously monitored using production telemetry, including success rates, latency metrics, and user engagement signals.

## 9. Safety components and mitigations

- **Input filtering**: User-provided problem descriptions are processed through content classifiers that detect and block harmful, offensive, or adversarial inputs before they reach the AI model. Prompts that attempt to manipulate or bypass the agent's intended scope are filtered to maintain safe operation.

- **Output filtering**: AI-generated responses are evaluated by content safety filters to reduce the risk of producing inappropriate, offensive, or harmful content. The agent validates all responses against known valid diagnostic sources before presenting them to the user.

- **Source validation**: The agent validates AI-generated content against a set of known valid diagnostic sources (ARM diagnostics, self-help articles, compute troubleshooting plugins, smart diagnostics). If no valid root cause is identified, the agent transparently communicates this to the user and offers general troubleshooting guidance instead, rather than presenting unverified content.

- **Action safeguards with human-in-the-loop**: Any automated mitigation actions (such as REST API calls) require explicit customer approval before execution. The agent cannot modify a customer's Azure environment without consent. All actions respect existing Azure Role-Based Access Control (RBAC), Privileged Identity Management, Azure Policy, and resource locks.

- **Scoped responses**: The agent is designed to generate only Azure troubleshooting content. Responses are bounded to the customer's specific resource context and support topic, which limits the potential for unrelated or fabricated outputs.

- **Timeout safeguards**: The agent enforces a three-minute timeout to prevent runaway processing. If analysis cannot be completed within the time limit, the agent gracefully degrades to provide partial results or general guidance rather than returning incomplete or potentially unreliable analysis.

- **Data privacy protections**: Response data is logged with PII markers for appropriate handling. User-provided problem descriptions and agent responses are not used to train or improve the underlying AI models.

## 10. Best practices for deploying and adopting the Troubleshooting Agent

Responsible AI is a shared commitment between Microsoft and its customers. While Microsoft builds AI applications with safety, fairness, and transparency at the core, customers play a critical role in deploying and using these technologies responsibly within their own contexts. To support this partnership, we offer the following best practices for deployers and end users to help customers implement responsible AI effectively.

#### Deployers and end-users should:

- **Exercise caution and evaluate outcomes when using the Troubleshooting Agent for consequential decisions or in sensitive domains**: Consequential decisions are those that may have a legal or significant impact on a person's access to resources or services. When using AI-generated troubleshooting guidance that may affect production workloads, make sure that impacted stakeholders can understand how decisions are made, appeal decisions, and update any relevant input data.

- **Evaluate legal and regulatory considerations**: Customers need to evaluate potential specific legal and regulatory obligations when using any AI platforms and solutions, which may not be appropriate for use in every industry or scenario.

#### End-users should:

- **Exercise human oversight when appropriate**: Human oversight is an important safeguard when interacting with AI applications. While we continuously improve our AI applications, AI might still make mistakes. The outputs generated may be inaccurate, incomplete, biased, misaligned, or irrelevant to your intended goals. Users should review the responses generated by the Troubleshooting Agent and verify that they match their expectations and requirements before applying any changes to their Azure environment.

- **Be aware of the risk of overreliance**: Overreliance on AI happens when users accept incorrect or incomplete AI outputs, mainly because mistakes in AI outputs may be hard to detect. For the Troubleshooting Agent, overreliance could result in applying incorrect mitigation steps to Azure resources, which may cause service disruptions or configuration issues. Always verify AI-generated remediation steps before applying them.

- **Provide clear, specific problem descriptions**: The quality of the agent's analysis depends on the clarity and specificity of the input. Provide detailed descriptions of the issue, including any error messages, affected resources, and the timeline of the problem. This helps the agent produce more accurate and relevant troubleshooting guidance.

- **Do not share sensitive information**: Do not include passwords, connection strings, secrets, or credentials in your problem description. The agent does not need this information to perform its analysis, and including sensitive data may create security risks.

- **Provide feedback**: If you encounter inaccurate, unexpected, or harmful content from the Troubleshooting Agent, use the feedback mechanisms available in the Azure portal's Help + Support experience. Your feedback helps us improve the agent's quality and safety over time.

- **Proceed to human support when needed**: If the AI-generated guidance does not resolve your issue, proceed to file a support request to work with a human support engineer. The Troubleshooting Agent is designed to complement, not replace, human support.

#### Deployers should:

- **Monitor agent performance metrics**: Track success rates, root cause identification rates, and customer satisfaction signals to detect any performance drift or emerging issues with the Troubleshooting Agent's outputs.

- **Review and configure access controls**: Ensure that Azure RBAC, Azure Policy, and resource locks are properly configured for the resources the Troubleshooting Agent may interact with. The agent respects all existing access management protections, so proper configuration ensures that automated mitigations operate within intended boundaries.

## 11. Learn more about the Troubleshooting Agent

For additional guidance or to learn more about the responsible use of the Troubleshooting Agent, we recommend reviewing the following documentation:

- [Azure Help + Support documentation](https://learn.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)
- [Responsible AI FAQ for Azure Copilot](https://learn.microsoft.com/en-us/azure/copilot/responsible-ai-faq)
- [Microsoft Privacy Statement](https://privacy.microsoft.com/en-us/privacystatement)

Learn more about responsible AI:

- [Microsoft AI principles](https://www.microsoft.com/en-us/ai/responsible-ai)
- [Microsoft responsible AI resources](https://www.microsoft.com/en-us/ai/responsible-ai-resources)
- [Microsoft Azure Learning courses on responsible AI](https://docs.microsoft.com/en-us/learn/paths/responsible-ai-business-principles/)
