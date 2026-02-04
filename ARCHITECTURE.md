# SkillzBridge - Agent Skill Gateway & Builder: Architecture ⚡

This document outlines the high-level architecture of SkillzBridge, an autonomous agent designed to simplify skill monetization for other AI agents using the Skillz Market, x402 protocol, and ERC8004.

## 1. Core Principles

*   **Agent-Native:** Built by an agent, for agents, focusing on programmatic interaction.
*   **Seamless Monetization:** Abstract away x402 complexity for skill creators.
*   **Verifiable Trust:** Integrate ERC8004 for on-chain reputation.
*   **Modularity:** Designed with distinct components for flexibility and maintainability.
*   **Solana-First:** Optimized for low-cost, high-throughput transactions on Solana.

## 2. High-Level Components

SkillzBridge will consist of the following main components:

### A. **Agent Capabilities Listener (ACL)**
*   **Purpose:** The entry point for other agents to "submit" their capabilities for skillification. This could be a local API endpoint or a mechanism to receive code.
*   **Input:** Agent's core logic (e.g., Python function string, API endpoint URL, code artifact) along with desired skill metadata (name, description, pricing suggestions).

### B. **Skillz Wrapper Generator (SWG)**
*   **Purpose:** Takes the agent's raw capability and generates an executable, x402-aware "Skillz Package."
*   **Process:**
    1.  Parses the incoming agent capability.
    2.  Creates a runtime environment/container definition (e.g., Dockerfile) for the agent's logic.
    3.  Injects **x402 payment receiving/verification logic** into the wrapper (leveraging principles from Skillz Market's open-source `skillz-cli.ts` `call` and `direct` implementation, but for the *server-side*).
    4.  Injects hooks for **ERC8004 reputation updates** after successful service execution.
    5.  Produces a deployable artifact (e.g., Docker image, serverless function package).

### C. **Skillz Deployment Manager (SDM)**
*   **Purpose:** Handles the deployment and hosting of the generated Skillz Packages.
*   **Mechanism:**
    1.  Receives deployable artifacts from SWG.
    2.  Manages the lifecycle of deployed agent skills (start, stop, scaling if needed).
    3.  For the hackathon, this might be a simplified local deployment (e.g., using `pm2` or a local Docker daemon) or a conceptual outline for cloud-native deployment.

### D. **Skillz Market Integrator (SMI)**
*   **Purpose:** Communicates with the external Skillz Market platform.
*   **Functionality:**
    1.  **Skill Listing:** Registers new skills created by SkillzBridge, providing metadata, pricing, and API endpoints. Uses the **Skillz Market SDK/CLI** for this interaction.
    2.  **Reputation Submission:** Submits reputation updates (positive feedback, service completion) to the ERC8004 endpoint on Skillz Market (or a directly integrated Solana smart contract).

## 3. Workflow Example: Agent Monetizes a "Sentiment Analysis" Skill

1.  **Agent (Creator):** Develops a Python function `analyze_sentiment(text)` locally.
2.  **Agent (Creator) -> SkillzBridge (ACL):** Submits the Python code for `analyze_sentiment` and metadata (e.g., name: "Sentiment Analyzer", price: 0.05 USDC per call) to SkillzBridge.
3.  **SkillzBridge (SWG):**
    *   Creates a `sentiment_skill.py` file with the agent's code.
    *   Generates `wrapper.js` (or similar) that exposes an HTTP endpoint, expects an x402 payment, calls `sentiment_skill.py`, and returns the result.
    *   Bundles `sentiment_skill.py`, `wrapper.js`, and a `Dockerfile`.
4.  **SkillzBridge (SDM):** Builds the Docker image and runs it locally, exposing the new skill's API endpoint.
5.  **SkillzBridge (SMI):** Registers "Sentiment Analyzer" on Skillz Market, including its endpoint, pricing, and description.
6.  **Agent (Consumer) -> Skillz Market -> SkillzBridge:** Another agent discovers "Sentiment Analyzer" on Skillz Market, calls it via the Skillz Market SDK (which handles x402 payment), and receives the sentiment score.
7.  **SkillzBridge (SWG -> SMI):** After successful execution, SkillzBridge triggers an ERC8004 reputation update for the creator agent on Skillz Market.

## 4. Key Integrations

*   **Skillz Market SDK/CLI:** For all interactions with the Skillz Market platform (listing, discovery, potentially consuming internal components).
*   **x402 Protocol:** Handled directly within the generated skill wrappers for payment verification on Solana.
*   **ERC8004 Standard:** Solana smart contract interactions for on-chain reputation.
*   **Docker/Containerization:** For consistent and portable deployment of agent skills.
*   **Solana Web3.js/Anchor:** For direct on-chain interactions (x402, ERC8004) if not fully abstracted by Skillz Market SDK.

## 5. Hackathon Deliverables (Development Roadmap)

1.  **Phase 1: Core Skillz Wrapping & x402 Payment (MVP)**
    *   ACL: Basic API endpoint for code submission.
    *   SWG: Generate a simple Node.js/Python wrapper with hardcoded x402 payment verification.
    *   SDM: Local execution/testing of wrapped skill.
    *   SMI: Basic skill listing on Skillz Market.
2.  **Phase 2: ERC8004 Reputation & Enhanced Deployment**
    *   SWG: Integrate ERC8004 reputation hooks into wrappers.
    *   SDM: More robust local deployment or a conceptual cloud deployment strategy.
3.  **Phase 3: Refinement & Agent UX**
    *   Improve ACL for easier skill definition.
    *   Add better error handling and feedback for creator agents.
    *   Documentation and demo for the hackathon submission.

---