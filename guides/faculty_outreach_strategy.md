# Faculty Outreach Strategy & Academic Recruitment Field Manual

**Target Institutions:** ETH Zurich (RSL & ASL), TU Delft (CoR), NTNU (Engineering Cybernetics)  
**Candidate:** Charles Oluwatuase | Prospective MSc / Research Scholar  
**Objective:** Secure faculty sponsorship, laboratory research interest, and supervision commitment before submitting formal admissions portals.

---

## 1. The Psychology of European Robotics Professors & Postdocs

Leading European robotics professors (such as Prof. Marco Hutter or Prof. Jens Kober) receive **50+ unsolicited emails per day** from international applicants. 

Over 95% of these emails are discarded within 3 seconds because they:
1. Use generic flattery (*"I am deeply inspired by your illustrious publications..."*).
2. Ask for money/scholarships before demonstrating competence.
3. Attach massive 10MB PDFs that no professor will download.
4. Show standard undergraduate coursework that requires months of retraining.

### The 3 Rules of Sniper Academic Outreach:
* **Rule 1: Proof-of-Work Precedes Conversation.** A working GitHub repository and an unlisted 45-second YouTube demo video is 100x more persuasive than a Statement of Purpose.
* **Rule 2: Postdocs Are the Gatekeepers.** Postdocs and senior PhD candidates actively write the lab's codebases. When a postdoc says to the professor, *"Hey, this applicant from Nigeria built a clean OpenUSD ingestion tool for Isaac Lab and asked an intelligent question about our solver jitter,"* the professor will invite you for an interview.
* **Rule 3: Respect Cognitive Bandwidth.** Keep emails under 175 words. Use bullet points for links. Never ask for an open-ended meeting—ask for a 10-minute discussion.

---

## 2. Stage 1 Execution (Week 16): The Postdoc Technical Inquiry

Send mid-week (Tuesday or Wednesday at 09:30 Central European Time):

* **Recipients:** 
  * ETH Zurich RSL: **Dr. Vaishakh Patil** (`vpatil@ethz.ch`) or **Mayank Mittal** (`mittalma@ethz.ch`)
  * ETH Zurich ASL: **Dr. Andrei Cramariuc** (`acramariuc@ethz.ch`)
  * TU Delft CoR: **Giovanni Franzese** (`g.franzese@tudelft.nl`)
* **Subject:** `Technical Question regarding [Paper Title / rsl_rl / Isaac Lab contact dynamics]`
* **Objective:** Establish peer-level technical communication and validate your simulation pipeline.

---

## 3. Stage 2 Execution (Week 20): The Lead PI Pitch

Send Tuesday morning (08:30 – 09:15 CET) directly from your university email:

* **Recipients:**
  * **Prof. Marco Hutter** (`mahutter@ethz.ch`) — Head, Robotic Systems Lab, ETH Zurich
  * **Prof. Roland Siegwart** (`rsiegwart@ethz.ch`) — Head, Autonomous Systems Lab, ETH Zurich
  * **Prof. Jens Kober** (`j.kober@tudelft.nl`) — Head, RL for Control, TU Delft
  * **Prof. Kostas Alexis** (`kostas.alexis@ntnu.no`) — Autonomous Robots Lab, NTNU
  * **Prof. Kristin Y. Pettersen** (`kristin.y.pettersen@ntnu.no`) — Snake Robotics & Nonlinear Control, NTNU

### The Pitch Email Formula (Under 170 Words):
```
Subject: Prospective MSc Researcher: OpenUSD & Isaac Lab Sim-to-Real Pipeline | [Lab Name]

Dear Professor [Last Name],

I follow [Lab Name]’s research on [specific topic, e.g., dynamic legged locomotion / compliant manipulation].

I am completing my Mechatronics Engineering degree at FUTA alongside a B.S. in Computer Science at University of the People. Over the past six months, I developed a production Sim-to-Real pipeline translating raw CAD assemblies into Isaac Lab with automated OpenUSD mass-inertia ingestion and V-HACD convex decomposition.

Using this framework, I trained parallel PPO policies across 4,096 environments with domain randomization over joint damping and contact friction, successfully exporting to ONNX for low-latency edge deployment:
• 45-Second End-to-End Pipeline Demo: [Link to Unlisted YouTube]
• Open-Source Ingestion & Training Repository: [Link to Clean GitHub Repo]

I am applying to the [MSc Program Name] for the upcoming intake. I would welcome the opportunity to contribute to [Lab Name]’s research on [mention 1 active challenge].

Would you or a member of your research team have 10 minutes to discuss research alignment?

My CV and dual-degree academic transcripts are attached for your review.

Sincerely,

Charles Oluwatuase
[LinkedIn] | [GitHub]
```

---

## 4. The 7-Day Follow-Up Protocol

If you receive no response after 7 business days, reply to your original email thread with a **value-add update**:

```
Dear Professor [Last Name],

I am following up on my note below. 

Over the past week, we completed the edge deployment benchmark: the exported ONNX actor policy achieved a mean inference latency of 0.82 ms on ARM architecture, maintaining stability under unmodeled 20% payload variation.

The updated benchmark curves and documentation have been added to the repository: [GitHub Link].

I welcome the opportunity to connect briefly if your schedule permits.

Best regards,
Charles Oluwatuase
```

---

## 5. Technical Interview Defense Preparation

If invited to a virtual meeting, be prepared to answer:
1. **"How did you compute the 3x3 inertia tensor from the CAD model?"**
   * *Answer:* "We utilized `trimesh` assuming uniform volumetric density, integrating over the tetrahedral mesh volume to generate the raw inertia tensor, and verified the parallel axis theorem transformation relative to the USD articulated joint origin."
2. **"Why did you use V-HACD instead of standard convex hulls?"**
   * *Answer:* "Standard single convex hulls cause severe collision penetration on concavities, creating artificial joint locks. V-HACD decomposes complex geometry into up to 16 sub-convex hulls, ensuring exact boundary fidelity without the computational cost of triangle mesh collisions in PhysX."
3. **"What was your domain randomization strategy?"**
   * *Answer:* "We implemented an `EventManager` modulating base mass $\pm 15\%$, joint damping $\pm 20\%$, and static friction between 0.3 and 1.2 on environment resets, coupled with periodic randomized external velocity impulses to prevent policy overfitting."
