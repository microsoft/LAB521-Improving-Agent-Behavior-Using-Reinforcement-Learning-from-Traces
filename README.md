@lab.Title

<!-- ## Welcome to Your Lab Environment

To begin, log into the virtual machine using the following credentials: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++
>[!Note] ** Text formatted as an +++example+++ represents type text. Clicking on this text will automatically insert it to prevent any typing errors.

To edit the lab manual, click the hamburger menu in the top-right corner and select **"Edit Instructions"**. This will open the editor where you can modify the instructions.

Instructions are written in Markdown. For detailed guidance on syntax, please refer to the following documentation:

[Creating Instructions with Markdown Syntax](https://docs.skillable.com/docs/creating-instructions-with-markdown-syntax)

If you want to manage your instructions outside of Skillable Studio like Github, please refer to the following documentation:

[Manage Instructions Outside of Studio](https://docs.skillable.com/docs/manage-instructions-outside-of-studio)

For saving changes made to the virtual machine, please refer to this guide:  

[Committing Changes to a Virtual Machine](https://docs.skillable.com/docs/committing-changes-to-a-lab)
>[!Note] ** When prompted, select **"Commit my changes and update this lab profile"** to ensure your changes are saved.

Changes made to the virtual machine will take effect immediately after committing. You may restart the lab instance if you wish to view these changes.
>[!Note] ** Changes to the instructions will be saved automatically, so the commit process applies only to modifications to the virtual machine.

Should you require any assistance, feel free to contact us.

**Please remove this message once it is no longer needed.** -->

# Introduction

> [!NOTE]
> This is a **75-minute** hands-on lab where you will improve AI agent behavior using reinforcement fine-tuning (RFT) on traces from an agentic workflow.

## Learning Objectives

By the end of this lab, you should be able to:

- Understand how reinforcement fine-tuning (RFT) improves agentic tool-calling behavior without redesigning the agent
- Inspect agent execution traces and define "good behavior" using evaluation graders with partial credit scoring
- Build training data from agent runs and submit an RFT fine-tuning job to Microsoft Foundry
- Evaluate checkpoints to identify the best-performing fine-tuned model

## Resources

The repository to the lab is live, you can find it at **[GitHub Repo](https://aka.ms/build2026-LAB521)**

> [!TIP]
> You can find your Microsoft Foundry logins, and any other credentials in the **Resources tab**.

## Lab Outline

The lab is organized into **5 notebooks**, taking you through the full RFT workflow:

1. **Introduction & Setup** - Connect to Microsoft Foundry and verify your environment
2. **Meet the Agent** - Run Zava's return-resolution agent live on scenarios and observe its tool-calling behavior
3. **Baseline & Grader** - Evaluate the base model, understand how the grader drives RFT learning, and calibrate the pass threshold
4. **Build Data & Submit a Job** - Explore the RFT data format, craft a training example, and submit a fine-tuning job
5. **Training Results & Evaluate** - Analyze the reward curve, compare checkpoints, and run a head-to-head evaluation of the fine-tuned model

## Business Scenario

In this lab you'll be improving an AI agent for **Zava**, a fictional DIY retailer that operates both online and in physical stores across the United States. Zava specializes in home improvement, hardware, tools, and DIY supplies.

### The Challenge

Zava's customer service team handles a high volume of **return, exchange, and dispute requests**. Each request requires the agent to:

1. Look up the customer's order via a tool call
2. Apply a complex return policy with multiple rules (loyalty tiers, product categories, sale flags, late delivery credits, restocking fees, etc.)
3. Determine the correct resolution - refund, store credit, exchange, or denial - and compute the exact dollar amounts

The base model **o4-mini** handles most cases correctly, but struggles with edge cases: it scores around **73%** on the validation set. The goal of this lab is to push that to **87%+** using RFT.

### Why Reinforcement Fine-Tuning?

Unlike supervised fine-tuning (SFT), RFT **does not require labeled ideal responses**. Instead:

- You provide training **prompts** and a **grader function** that scores model outputs
- During training, the model generates candidate responses and receives reward signals
- The model learns through trial and error to maximize its score on the grader

This makes RFT particularly well-suited to agentic tasks where "correctness" is defined by a business rule - not by a single golden response.

### The Grader

The grader in this lab evaluates each model response on four dimensions:

| Dimension | Weight | What it checks |
|-----------|--------|----------------|
| Correct action | 40% | Did the model choose refund / denial / store credit / exchange correctly? |
| Correct amounts | 30% | Are the dollar figures (refund, fees, credits) computed correctly? |
| Policy reasoning | 20% | Does the model cite the correct policy rules? |
| Tool usage | 10% | Did the model call *get_order* to retrieve order data? |

The grader returns a score between **0.0 and 1.0**. A **pass threshold of 0.80** is used: responses scoring ≥ 0.80 count as "passing" for the RL reward signal.

Click **Next** to set up your lab environment and get started.

===

# Part 1 - Get started

## Sign in to Windows

As a first step, login into the lab Virtual Machine using the credentials you can find in the **Resources tab** under the Skillable VM name.

-  Password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++

> [!TIP]
>  First time using **Skillable?** The green "T" (e.g., +++Admin+++) indicates values that are automatically input for you at the current cursor location in VM, with one click. This reduces your effort and minimizes input errors.
> Also, you can always click on the images to enlarge them, if needed.

## Open workshop in Visual Studio Code

In this workshop, we will be using **Visual Studio Code** as our development environment. It is equipped with all the necessary tools and dependencies pre-installed. This will allow you to focus on learning and prototyping without worrying about local setup.

1. To launch Visual Studio code, you can either click on the icon in the home page, or search for Visual Studio Code in the start up menu, or Click on the Visual Studio icon that is pinned to the task bar. 

2. In the Visual Studio Code, open a new terminal (**Terminal → New Terminal**) and run the setup script to populate your `.env` file with your Microsoft Foundry credentials:


 
./src/scripts/get-env


> [!TIP]
>  Ensure the default terminal launched is bash. If not, switch the terminal by clicking the **down arrow,** next to the add icon and select **bash** to ensure the default terminal is bash.

This will create a `.env` file in the repo root with values for:



AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/openai/v1
AZURE_OPENAI_API_KEY=<your-key>
FOUNDRY_PROJECT_ENDPOINT=https://<resource>.services.ai.azure.com/api/projects/<resource>


## Login to Azure

1. Next, open the edge browser and navigate to <[https://ai.azure.com](https://ai.azure.com)

2. Sign-in with the following credentials:
   -  Username: +++@lab.CloudPortalCredential(User1).Username+++
   -  TAP: +++@lab.CloudPortalCredential(User1).TAP+++

3. Once you are signed in, head back to **Visual Studio Code** to continue with the lab.

## Validate setup using Notebook 01

1. Head back to Visual Studio Code and confirm a `.env` file has been created successfully.

1. In the VS Code Explorer panel (left sidebar), expand the **`src/`** folder.
2. Click on **`01-introduction-setup.ipynb`** to open it.
3. At the top of the notebook click the **Run All** command to execute the notebook. A pop up will be created, select: **Python Environments... > Python 3.13....**, you will have successfully executed the notebook.

## What have you achieved?

As you work through the cells in `01-introduction-setup.ipynb` in order, you will see:

1. **Install dependencies** - Installs required Python packages (`openai`, `python-dotenv`, `requests`, `matplotlib`, `tabulate`). This may take 1-2 minutes.
2. **Import libraries and connect to Microsoft Foundry** - Loads your `.env` file and creates an OpenAI client pointed at your Microsoft Foundry endpoint.
3. **Verify your environment** - Confirms all required environment variables are set and that the data files exist.

> [!NOTE]
> If the verification cell reports missing environment variables, double-check your `.env` file at the repo root. If it's missing, re-run `./src/scripts/get-env` in the terminal.

### ✅ Setup Complete

Once all three setup cells pass with no errors, your environment is ready. You should see a confirmation message in the last cell output.

Click **Next** to meet the Zava agent and see it in action.
===

# Part 2 - Meet the Agent

> [!NOTE]
>[!Note] book**: `src/02-meet-the-agent.ipynb` | **Estimated time**: ~10 minutes

In this section, you'll run Zava's return-resolution agent on live customer scenarios and observe how it uses tools to fetch order data and apply complex policy rules.

## Overview

The agent works as follows:

1. A customer sends a message (e.g., "I want to return my broken drill")
2. The agent calls the **get_order** tool to look up the order details
3. The agent applies Zava's return policy to determine the resolution
4. The agent responds with the action (refund/denial/store credit/exchange) and the dollar amounts

## Step 1: Open the Notebook

> [!TIP]
> As you open a new notebook, first, click on notebook file to open it. Next, at the top of the notebook click the **Run All** command to execute the notebook. In case the notebook doesn't run, click **Restart** to restart the notebook and run again.

1. In the VS Code Explorer (**src/** folder), open **`02-meet-the-agent.ipynb`**.
2. Run the **Setup** cell at the top to reconnect to Microsoft Foundry.

> [!NOTE]
> Each notebook re-establishes its own connection to Microsoft Foundry. This is expected - run the setup cell at the top of each notebook before working through the rest.

## Step 2: Read the Zava Return Policy

Before running the agent, read through the **policy table** in the notebook. The policy is intentionally complex - this is what makes it a good candidate for RFT:

| Rule | Details |
|------|---------|
| Return windows | Standard: 30 days · Gold: 45 days · Platinum: 60 days |
| Electronics windows | Standard: 15 days · Gold: 30 days · Platinum: 45 days |
| Electronics restocking fee | Standard: 15% · Gold: 7.5% · Platinum: 0% |
| Defective items | Always free return, $0 restocking fee |
| Sale items | Final sale - no returns (except defective: store credit only) |
| Late delivery (> 2 days) | $10 credit automatically applied |

> [!TIP]
> The agent has this policy in its system prompt. The challenge is that the model must apply **multiple rules simultaneously** - for example, a defective item that is also a sale item should result in store credit, not a refund.

## Step 3: Define the Agent
The agent consists of four parts:

1. **System prompt** - tells the model what it is and how to apply the policy
2. **Tool definition** - the `get_order` function schema the model can call
3. **Tool executor** - calls the Azure Function (with local fallback)
4. **Agent loop** - orchestrates model → tool call → model → response

Run the define the agent cell to excute the agent

## Step 4: Run the Three Scenarios

The notebook includes three pre-built customer scenarios. Run each one in order and observe the agent's output.

### Scenario 1: Defective Electronics Return

A customer wants to return a broken power tool. This is a relatively straightforward case - the item is defective, so the restocking fee is waived.

**Watch for:**
- Does the agent call **get_order** before answering?
- Does it correctly waive the restocking fee for defective items?
- Are the dollar amounts correct?

### Scenario 2: Defective Sale Item

This is where it gets tricky. The item is both **on sale** (normally final sale, no returns) and **defective** (should get store credit). The model must recognize *both* flags to reach the correct resolution.

**Watch for:**
- Does the agent issue a refund, a denial, or store credit?
- The correct answer is **store credit** - not a refund, not a denial.

### Scenario 3: Exchange Request

The customer wants an exchange rather than a return. This requires computing the return amount for the original item and the cost of the replacement item.

**Watch for:**
- Does the agent correctly compute the net exchange cost?
- Does it apply the correct return window for the customer's loyalty tier?

## Step 4: Observe the Results

After running all three scenarios, take note of what the agent got right and where it made mistakes. Common issues include:

- Getting the **action** right (e.g., refund) but the **amount** wrong
- Missing a **policy interaction** (e.g., defective + sale → store credit, not refund)
- Forgetting to apply the **late delivery credit**

> [!TIP]
> These are exactly the kinds of errors that RFT can fix. The model needs to learn the precise interaction between policy rules through trial and error - which is what you'll set up in notebooks 03 and 04.

## Agent Architecture (for reference)

The agent in this lab is composed of four parts:

| Component | Description |
|-----------|-------------|
| **System prompt** | Tells the model what it is and how to apply the Zava return policy |
| **Tool definition** | The **get_order** function schema that the model can call |
| **Tool executor** | Calls the Azure Function endpoint (with a local fallback for offline use) |
| **Agent loop** | Orchestrates model → tool call → model → final response |

Click **Next** to establish a baseline and understand how the grader drives RFT learning.
===

# Part 3 - Baseline Evaluation & The Grader

> [!NOTE]
>[!Note] book**: `src/03-baseline-grader.ipynb` | **Estimated time**: ~15 minutes

In this section, you'll measure how well the base model performs on Zava's return scenarios, understand the grader that drives RFT learning, and calibrate the pass threshold for optimal training signal.

## Step 1: Open the Notebook

> [!TIP]
> As you open a new notebook, first, click on notebook file to open it. Next, at the top of the notebook click the **Run All** command to execute the notebook. In case the notebook doesn't run, click **Restart** to restart the notebook and run again.

1. Open **`src/03-baseline-grader.ipynb`** in VS Code.
2. Run the **Setup** cell to reconnect to Microsoft Foundry and reload the agent infrastructure.

## Step 2: Understand the Grader

The grader is the most important concept in RFT. It replaces the "ideal assistant response" used in supervised fine-tuning with a **scoring function** that evaluates model outputs.

Run the **Define the Grader** cell to load the grader function.

The grader scores each response on four dimensions:

| Dimension | Weight | How it's evaluated |
|-----------|--------|-------------------|
| **Correct action** | 40% | Keyword match for refund / denial / store credit / exchange |
| **Correct amounts** | 30% | Dollar figures extracted and compared with tolerance |
| **Policy reasoning** | 20% | Checks for citation of applicable policy rules |
| **Tool usage** | 10% | Verifies the agent called **get_order** |

The grader returns a float in `[0.0, 1.0]`. A response that gets the action right and cites policy earns ~60% even if the dollar amounts are off.

> [!TIP]
> **Why partial credit?** Binary scoring (pass/fail) provides a weak training signal - the model can't tell whether it was "almost right" or completely wrong. Partial credit scoring lets the RL algorithm distinguish between near-misses and total failures, leading to faster and more stable learning.

### RFT vs. SFT

| | Supervised Fine-Tuning (SFT) | Reinforcement Fine-Tuning (RFT) |
|---|---|---|
| Training data | Prompt + ideal response pairs | Prompts only + grader function |
| Signal | "Copy this response" | "This attempt scored 0.85 - do better" |
| Best for | Teaching format and style | Improving reasoning and accuracy |
| Data required | Labeled examples | Diverse prompts + a good grader |

## Step 3: Load the Validation Scenarios

Run the **Load Validation Scenarios** cell to load the 57 held-out scenarios from `data/rft_v7_val.jsonl`.

These scenarios cover all major policy rules and edge cases in the Zava return policy. They are **never** used as training data - only for evaluation.

> [!NOTE]
> The next cell demonstrates generating ground-truth answers by running our 57 validation scenarios through the trusted `gpt-5.4` model and caching the results to `data/rft_v7_val_gt.jsonl`. This takes ~15-20 minutes for all scenarios. If the cache file already exists, it loads instantly.

## Step 4: Run the Baseline Evaluation

Run the **Run the Baseline Evaluation** cell. This samples **8 scenarios** from the validation set and runs the full agent loop on each one, scoring the result with the grader.

> [!WARNING]
> This cell makes ~8 API calls and takes approximately **5-8 minutes**. While it runs, move on to reading the grader explanation in the notebook and the threshold calibration section below.

### What to expect

The base `o4-mini` model typically scores around **0.73 average** on Zava's validation set. You'll see a mix of:
- High scores (0.9-1.0) on straightforward cases
- Medium scores (0.5-0.7) on edge cases where the model gets the action right but amounts wrong
- Low scores (0.0-0.3) on complex policy interactions (e.g., defective sale items)

## Step 5: Calibrate the Pass Threshold

The **pass threshold** determines what counts as "success" during RFT training. Run the **Calibrate the Pass Threshold** cell.

The goal is a **30-50% failure rate** on the baseline model:

| Failure rate | Implication |
|---|---|
| < 20% | Model already passes most examples - little to learn from |
| 25-50% | **Ideal range** - strong learning signal without sparse reward |
| > 60% | Too hard - sparse rewards make convergence slow |

A threshold of **0.80** typically gives a ~35% failure rate on base `o4-mini`, making it the default for this lab.

> [!TIP]
> The pass threshold is a hyperparameter you can tune. A higher threshold (e.g., 0.90) pushes the model harder but converges more slowly. For this lab, use the default of 0.80.

## Key Takeaways

- Base `o4-mini` scores **~73%** on Zava policy scenarios - good, but with room to improve
- The grader provides **partial credit** (0.0-1.0) rather than binary pass/fail, enabling richer training signal
- A **pass threshold of 0.80** gives an optimal ~35% failure rate for RL learning
- RFT uses these scores to teach the model which behaviors to reinforce through trial and error

Click **Next** to build training data and submit a RFT job.

===

# Part 4 - Prepare Training Data & Submit a Job

> [!NOTE]
>[!Note] book**: `src/04-build-data-submit-job.ipynb` | **Estimated time**: ~15 minutes

In this section, you'll explore the RFT training data format, construct a custom training example from scratch, and submit a fine-tuning job to Microsoft Foundry.

## Step 1: Open the Notebook

> [!TIP]
> As you open a new notebook, first, click on notebook file to open it. Next, at the top of the notebook click the **Run All** command to execute the notebook. In case the notebook doesn't run, click **Restart** to restart the notebook and run again.

1. Open **`src/04-build-data-submit-job.ipynb`** in VS Code.
2. Run the **Setup** cell to reconnect to Microsoft Foundry and reload the agent infrastructure.

## Step 2: Explore the RFT Data Format

Run the **Explore the Data Format** cell to examine a training example from `data/rft_v7_train.jsonl`.

RFT training data is intentionally simpler than SFT data - you only need **prompts**, not ideal responses:


json
{
  "messages": [
    {"role": "developer", "content": "<system prompt with Zava policy>"},
    {"role": "user", "content": "I want to return the drill I bought last week..."}
  ],
  "expected_resolution": {
    "action": "refund",
    "amount": 45.50,
    "reason": "within_return_window"
  }
}


> [!TIP]
> The `expected_resolution` field is **only used by the grader** to score model outputs during training. You don't provide model responses - the model generates those itself during training rollouts.

Run the **Dataset Overview** cell to see how many training and validation examples are in the pre-built dataset.

### How the training data was built

The pre-built dataset (`rft_v7_train.jsonl`, 343 examples) was constructed in two steps:

1. ~40 unique scenarios were written by hand, covering every policy rule and edge case
2. An LLM generated 10 rephrasings of each scenario (different tones, wordings, levels of detail), creating 400 diverse examples

This approach ensures the model encounters the same underlying policy rules across many different customer phrasings - which is critical for generalization.

## Step 3: Prepare Your Own Training Example

Run the **Build Your Own Example** section. This is an interactive exercise:

1. Call the **get_order** tool on a sample order ID to see what data is available
2. Determine the correct resolution based on the Zava return policy
3. Write the `expected_resolution` object with the correct action, amount, and reason
4. Run the grader on your example to verify it scores correctly

> [!TIP]
> This exercise builds intuition for what makes a good RFT training example. The key insight: **data diversity matters more than volume**. A dataset with 50 truly diverse scenarios often outperforms one with 500 near-identical paraphrases.

## Step 4: Upload Files

Run the **Upload Files** cell to upload the pre-built training and validation datasets to Microsoft Foundry.

The cell will return file IDs for both files. These IDs are used in the subsequent job submission step.

> [!NOTE]
> File processing takes **10-30 seconds** after upload. The notebook includes a wait loop that polls the file status. Wait for both files to show `processed` before proceeding.

## Step 5: Define the Grader for the Training Job

Run the **Define the Grader** cell. This embeds the same Python grader function you used for local evaluation as a string - the Microsoft Foundry training service executes this grader on each model rollout during training.

The grader string must define a `grade(sample, item)` function:
- `sample` - the training example (contains `expected_resolution`)
- `item` - the model's output for that rollout
- Returns a float in `[0.0, 1.0]`

## Step 6: Submit the RFT Job

Run the **Submit the Job** cell to submit your fine-tuning job. The job is configured with:

| Parameter | Value | Why |
|-----------|-------|-----|
| Base model | `o4-mini` | Capable reasoning model with tool-calling support |
| Pass threshold | `0.80` | Gives ~35% failure rate - optimal learning signal |
| Epochs | `3` | Sufficient for convergence on a 343-example dataset |
| Reasoning effort | `medium` | Balances training cost and quality |
| Checkpoint interval | Every 5 steps | Lets you compare intermediate checkpoints |

The cell will print your **job ID**. Save it - you'll use it in notebook 05 to check status and in notebook 06 to review your results.

> [!WARNING]
> RFT jobs are queued and run sequentially per resource. If multiple participants submit at the same time, your job may show `queued` status - this is expected. Jobs typically complete in **4-10 hours**.

We already submitted an RFT job in Microsoft Foundry, to see the job submitted, go to:

1. Go back to browser where you logged in to **Microsoft Foundry** using <[https://ai.azure.com](https://ai.azure.com). Close all the pop ups.

1. Switch to **New Foundry** by toggling the switch button on the top right.

1. In the new window, on the top right navigation, select **Build**

1. On the right side bar, select **Fine Tune**, you will see the jobs you submitted and one previously submitted. Some jobs might actually have been started already, click on the link to the fine tuning job to view any results or outcomes.

> [!TIP]
> Good News though, you don't need to wait for your job to finish to continue with the lab. Notebook 05 uses **pre-run results** from a completed experiment to demonstrate what the reward curve and checkpoint evaluation look like. And we have an already fine tuned model you can test out

## Key Takeaways

- RFT training data only needs **prompts + expected answers** - no ideal assistant responses
- **Data diversity** matters more than volume: diverse scenarios generalize better than large paraphrase sets
- The grader embedded in the job definition is the **same function** used for local evaluation - consistency is critical
- Jobs are queued; results arrive in **4-10 hours** - you'll evaluate pre-run results in the next notebook

Click **Next** to explore training results and evaluate the fine-tuned model.

===

# Part 5 - Training Results & Evaluate the Fine-Tuned Model

> [!NOTE]
>[!Note] book**: `src/05-training-results-evaluate.ipynb` | **Estimated time**: ~15 minutes

In this section, you'll analyze the training reward curve from a completed RFT run, compare checkpoints, and run a head-to-head evaluation of the fine-tuned model against the base model.

> [!TIP]
> Your submitted job from notebook 04 can take 10 hours to complete. This notebook uses **a model we already trained for you** from an identical experiment so you can see the full training story without waiting. Once your job finishes, you can re-run this notebook with your own results.

## Step 1: Open the Notebook

> [!TIP]
> As you open a new notebook, first, click on notebook file to open it. Next, at the top of the notebook click the **Run All** command to execute the notebook. In case the notebook doesn't run, click **Restart** to restart the notebook and run again.

1. Open **`src/05-training-results-evaluate.ipynb`** in VS Code.
2. You will need to add the credentials of an already finetuned model, the credentials are as follows:
     -  azure_endpoint: `https://ai-proxy-devrel-zzqb7qbzsy-proxy.politepond-e622dd09.eastus2.azurecontainerapps.io/api/v1`
     - api_key: `7893cab5-8ad2-4124-9dd1-7c48c548d19a`
3. Run the **Setup** cell to reconnect to Microsoft Foundry and reload the agent infrastructure.

## Step 2: Plot the Reward Curve

Run the **Plot Reward Curve** cell. This loads pre-computed training metrics from `results/training_metrics.csv` and plots:

- **Train and validation reward** - average grader score on training examples and held-out validation examples per step
- **Completion and reasoning tokens** - how many completion and reasoning tokens the model was using on each request, showing how hard the model is thinking
- **Tool call errors** - how often the model failed to call **get_order** correctly

### What a healthy reward curve looks like

| Signal | What to look for |
|--------|-----------------|
| Train reward | Rises steadily, then plateaus |
| Validation reward | Tracks train reward - large divergence suggests overfitting |
| Tokens | Rises steadily as the model improves its thinking strategies |
| Tool call errors | Should drop sharply in early steps as the model learns to call **get_order** |

> [!TIP]
> The **best checkpoint** is not always the last one. The train or validation reward may peak at an intermediate step and then slightly decline as the model overfits to the training distribution. Always evaluate multiple checkpoints on your held-out data.

## Step 3: Compare Checkpoints

Run the **Checkpoint Comparison** cell. This contains evaluation scores for checkpoints at steps 95 and 115.

You'll see something like:

| Checkpoint | Avg Score | P@0.8 (pass rate) |
|------------|-----------|-------------------|
| Base o4-mini | ~73% | ~60% |
| Step 95 | ~85% | ~83% |
| Step 115 | ~87% | ~90% |

> [!NOTE]
> **Step 115 outperforms Step 95** even though the training log shows Step 95 with a slightly higher train reward in some experiments. This is a common RFT pattern: training-time metrics use a sample of rollouts, while held-out evaluation on the full validation set gives a more reliable signal. **Always re-evaluate checkpoints on full held-out data before choosing the best one.**

## Step 4: Run the Head-to-Head Evaluation

Run the **Head-to-Head Evaluation** cells. This runs the same 8 validation scenarios on both:
- **Base `o4-mini`** - the unmodified model
- **`o4-mini-2025`** - the best fine-tuned checkpoint, pre-deployed by your facilitator

> [!WARNING]
> This cell makes ~16 API calls and takes approximately **8-12 minutes**. The notebook uses concurrent requests where possible to speed this up.

### What to look for in the results

For each scenario, you'll see the raw responses and grader scores side by side. Pay attention to:

- **Edge cases**: Does the fine-tuned model correctly handle defective sale items (store credit, not refund)?
- **Amount accuracy**: Are the dollar figures exact, or does the model round incorrectly?
- **Policy citations**: Does the fine-tuned model cite more specific policy rules?
- **Tool calling**: Does the fine-tuned model always call **get_order** before answering?

### Expected results

  Average score                 69.5%          96.0%    +26.5%
  Pass@0.9 (strict)               12%            88%      +75%
  Pass@0.8 (good)                 50%           100%      +50%

| Metric | Base o4-mini | Fine-tuned (Step 115) |
|--------|-------------|----------------------|
| Average score | ~73% | **~87%** |
| Pass rate (P@0.8) | ~60% | **~90%** |
| Tool call error rate | ~28% | **~7%** |

## Key Takeaways

- **RFT improved average score from ~73% to ~87%** - a significant gain on policy-following accuracy
- **The reward curve confirms learning** - train and validation reward both rise, and tool call errors drop sharply
- **Step 115 beats Step 95** for this task - always evaluate checkpoints on full held-out data, not just training metrics
- **P@0.8 jumped from 60% → 90%** - the fine-tuned model now passes the "good" bar on 9 out of 10 scenarios
- **The model didn't need labeled responses** - it discovered correct behavior entirely through trial and error

Click **Next** to wrap up and explore next steps.

===

# Summary

Congratulations on completing **LAB 521: Improving Agent Behavior Using Reinforcement Learning from Traces**!

## What You Learned

In this lab, you worked through the full RFT workflow on an agentic task:

- **Ran a live agent** - ran Zava's return-resolution agent on customer scenarios, observing tool calling and policy application in action
- **Defined "good behavior"** - built a grader function with partial credit scoring that evaluates action accuracy, dollar amounts, policy reasoning, and tool usage
- **Established a baseline** - measured base `o4-mini` at ~73% on validation scenarios and calibrated the pass threshold for optimal RL signal
- **Built training data** - explored the RFT data format, constructed a custom example, and understood why prompt diversity matters more than volume
- **Submitted a RFT job** - uploaded data, embedded the grader, and launched a fine-tuning job on Microsoft Foundry
- **Evaluated results** - analyzed the reward curve, compared checkpoints, and confirmed a ~73% → ~87% improvement in policy-following accuracy

## Lab Recap

| Notebook | What You Did |
|----------|-------------|
| **01 - Introduction & Setup** | Connected to Microsoft Foundry, verified environment |
| **02 - Meet the Agent** | Ran the Zava agent live on 3 scenarios, saw tool calling in action |
| **03 - Baseline & Grader** | Evaluated base o4-mini (~73%), understood the grader and threshold calibration |
| **04 - Build Data & Submit** | Explored training data format, built a custom example, submitted an RFT job |
| **05 - Training Results & Evaluate** | Analyzed reward curves, compared checkpoints, confirmed ~87% fine-tuned accuracy |

## Check Your Submitted Job

Your RFT job from notebook 04 will finish in **8-10 hours**. Open `src/06-wrap-up.ipynb` and run the job status cell to check on it. Once it completes, you can deploy the best checkpoint and run the head-to-head evaluation with your own fine-tuned model.

## Next Steps & Experiments

Once your job completes, try these variations to deepen your understanding:

| Experiment | What to Change | Expected Effect |
|-----------|---------------|----------------|
| **Tighter threshold** | `pass_threshold=0.90` | Stricter compliance, slower convergence |
| **More training data** | Add more scenario variations | Better generalization to unseen cases |
| **Grader weights** | Increase action weight to 60% | Model prioritizes correct action over amounts |
| **Longer training** | `n_epochs=5` | May improve accuracy but risks overfitting-watch the reward curve |

## Applying RFT to Your Own Agent

The workflow you followed in this lab applies to any agentic task where:
- The agent uses tools to fetch data
- "Correctness" can be defined by a scoring function (even approximately)
- The model shows inconsistent behavior with prompting alone

Key things needed to apply RFT to your own scenario:

1. **A diverse set of prompts** - 50-400 examples covering your edge cases
2. **A grader function** - Python code that scores model outputs on 0.0-1.0
3. **A calibrated pass threshold** - target 30-50% failure rate on your baseline model

## Resources

| Resource | Description |
|----------|-------------|
| [Azure AI Fine-Tuning docs](https://learn.microsoft.com/azure/ai-services/openai/how-to/fine-tuning) | Official documentation for fine-tuning on Azure AI |
| [RFT with graders guide (OpenAI)](https://platform.openai.com/docs/guides/reinforcement-fine-tuning) | Detailed guide on RFT grader design and data format |
| [Build 2026 Next Steps](https://aka.ms/build26-next-steps) | Continue your learning journey after Build 2026 |

## Try This at Home

This lab is available for you to revisit at your own pace. You can find the complete code, data, and instructions in the official GitHub repository:

**[https://github.com/microsoft/Build26-LAB521](https://aka.ms/build2026-LAB521)**

---

### 🎉 Thank you for completing LAB 521!



