# Project Specification: Legislative Counterfactual Simulation Engine

Author: Gabriel Castilho
Role: Backend & System Architecture
Status: Final Architecture Phase (Post-Final Review)
Core Direction: Utility-Driven Strategic Multi-Agent System

## 1. Executive Summary

The Legislative Counterfactual Simulation Engine is a research-grade Multi-Agent System (MAS). It transcends standard graph diffusion models by embedding true Agent Agency.

Rather than passively broadcasting states, agents in this system possess a "Decision Layer." At each timestep, agents dynamically calculate the utility of persuading their peers, allocating limited political capital to target specific neighbors. The system marries the generative strategy of LLMs (which draft the targeted arguments) with a mathematically bounded, deterministic graph update equation, allowing for rigorous counterfactual analysis of legislative behavior.

## 2. The Agent Decision Layer (True Agency)

The core differentiator of this system is that interaction is not hardcoded; it is an optimization problem solved by the agent at every timestep.

### 2.1. Political Capital & The Activation Function

Each agent $i$ has a limited budget of Political Capital, $C_i$, representing how many messages they can send per iteration (e.g., $C_i = 2$).
Agent $i$ must choose a subset of active neighbors $A_i(t)$ from its total neighbors $N(i)$ to maximize influence.

### 2.2. The Influence Utility Function

To select targets, Agent $i$ evaluates the Expected Utility $U_i(j, t)$ of messaging neighbor $j$:

$$U_i(j, t) = \text{Centrality}(j) \times |P_i(t) - P_j(t)| \times \alpha_j$$

Intuition:

$\text{Centrality}(j)$: How structurally important is this neighbor? (e.g., Eigenvector centrality).

$|P_i(t) - P_j(t)|$: How much do we currently disagree? (Alignment gain potential).

$\alpha_j$: How persuadable is this neighbor? (Avoid wasting capital on stubborn ideologues).

Agent $i$ calculates this for all $j \in N(i)$ and activates the top $C_i$ targets. This is true strategic agency.

## 3. Explicit Role Differentiation

Agent heterogeneity is codified into specific architectural roles that govern their parameters ($\alpha_i$) and strategy:

The Leader: * Low $\alpha$ (Highly stable/stubborn).

High Political Capital ($C_i = 5$).

Strategy: Maximizes Utility function prioritizing high-centrality targets (Brokers).

The Broker: * Medium $\alpha$.

High betweenness centrality in the graph $W_{ij}$.

Strategy: Connects disparate clusters, prioritizing targets with high ideological divergence.

The Follower: * High $\alpha$ (Highly sensitive to influence).

Low Political Capital ($C_i = 1$).

Strategy: Seeks alignment, typically targeting Leaders for cues.

The Ideologue: * Near-zero $\alpha$ (Unchanging).

Strategy: Ignores utility. Either refuses to interact or blindly targets ideological enemies with zero expectation of return.

## 4. The Dynamic Semantic Update Rule

The bounded update equation ensures stability, but the semantic component ($S_{ij}$) is explicitly dynamic per iteration, not a static feature of the bill.

### 4.1. Generative Interaction

Once Agent $i$ selects target $j$ via the Utility function, the LLM generates a targeted argument.

$E(M_{ij}(t))$ = The embedding of the message sent from $i$ to $j$ at time $t$.

$S_{ij}(t) = \text{CosineSimilarity}(E(M_{ij}(t)), E(Profile_j))$

### 4.2. Bounded Update Equation

At time $t$, Agent $i$'s probability of voting "Yes" ($P_i \in [0, 1]$) updates via:

$$P_i(t+1) = (1 - \alpha_i) P_i(t) + \alpha_i \sum_{j \in A_i(t)} \left( \tilde{D}_{ij}(t) \cdot P_j(t) \right)$$

Where $\tilde{D}_{ij}(t)$ is the normalized combination of structural trust and dynamic semantic resonance:

$$D_{ij}(t) = W_{ij} \times \max(0, S_{ij}(t))$$

## 5. Evaluation, Baselines & Ablation

To prove the measurable value of the added complexity, the system will undergo rigorous evaluation against a hidden historical vote:

### 5.1. Baselines

Logistic Regression: Prediction using only static Ideology and Party affiliation.

Graph-Only Diffusion: Update rule using only $W_{ij}$ (no LLM, no $S_{ij}$, no strategic utility targeting).

### 5.2. Ablation Studies

We will systematically disable components to measure their impact on prediction accuracy:

Remove $S_{ij}$: Replace dynamic semantic messaging with a constant $1.0$.

Remove Utility Targeting: Force agents to message neighbors randomly instead of maximizing $U_i(j, t)$.

Remove Heterogeneity: Set all $\alpha_i$ to a constant $0.5$ (ignoring Roles).

### 5.3. Counterfactual Experiments

The ultimate showcase of the MAS:

The Decapitation Scenario: Remove the top 5 "Leaders" from the graph. Does the party consensus collapse?

The Echo Chamber: Force all agents to only target neighbors with $|P_i(t) - P_j(t)| < 0.2$. Does polarization increase?

## 6. Observations on Final Senior Feedback

The final review successfully identified the threshold between a "controlled diffusion model" and a true "Multi-Agent System."

### 6.1. The "Agent Agency" Solution

Feedback: Agents must choose actions, plan influence, and adapt strategy.
Resolution: Addressed in Section 2. We introduced the explicit Influence Utility Function. Agents now perform an $\text{argmax}$ over expected influence gain, adding the critical decision-making layer without bloating the architecture with unnecessary sub-systems.

### 6.2. Roles and Interaction

Feedback: Push roles further and define an explicit activation function $A_i(j, t)$.
Resolution: Addressed in Section 3 and 2.2. Roles now strictly define the parameters ($\alpha$ and Capital $C$) that feed into the activation function, directly dictating how an agent behaves strategically.

### 6.3. The Semantic Component ($S_{ij}$)

Feedback: If $S_{ij}$ is static per bill, it behaves like a feature, not communication. It must depend on message content per iteration.
Resolution: Addressed in Section 4.1. We explicitly state that $S_{ij}$ is calculated at time $t$ based on the newly generated message embedding $E(M_{ij}(t))$, creating a true dynamic conversational loop.

### 6.4. Complexity Control

Feedback: Do not add more components. Add one clear agent decision mechanism and validate it.
Resolution: We resisted the urge to add moving edges or complex argumentation trees. We added exactly one mechanism: The Utility Targeting Function. It is elegant, mathematically sound, and profoundly changes the system's behavior from passive to strategic.
