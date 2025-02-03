# Identifiability in Mechanistic Interpretability
## Challenges in Uniquely Explaining Neural Network Behaviors

Modern neural networks are complex, nonlinear systems whose internal computations remain poorly understood. **Mechanistic interpretability (MI)**, the field dedicated to reverse-engineering these models into human-understandable algorithms, has made progress in identifying subcomponents (circuits) and mapping them to high-level behaviors. However, a critical question remains: *Can we uniquely determine how a neural network computes a specific behavior?*

Our work demonstrates that under current MI criteria, multiple conflicting explanations often coexist, highlighting a fundamental limitation in ensuring unique interpretations. This underscores the need for refined methods that better address identifiability in neural network analysis.

---

### Mechanistic Interpretability: Goals and Methods

Mechanistic interpretability seeks to decompose neural networks into two components:
1. **The Algorithm (What)**: A simplified, human-readable description of the computation (e.g., "detect edges, then corners").
2. **The Circuit (Where)**: The subset of neurons and connections that physically implement the algorithm.

For instance, in a vision model, the *what* could state that the model detects edges first, then corners, then combines these to detect shapes such as rectangles. Conversely, the *where* could describe that edge detection might be localized to specific neurons in early layers, while object recognition involves downstream connections.

In practice, two main approaches aim to bridge low-level neural activations to interpretable algorithms:
1. The **where-then-what** approach first detects the subset of neurons that seems to be related to the behavior, and then tries to interpret the computations contained in these neurons. It it typically not possible to enumerate all possible circuits and interpretations, therefore methods such as **causal mediation analysis** (which identifies critical subgraphs by finding the subset of the network that carries information from the input to the output) and **activation maximization** (interpreting a component by looking at which inputs maximize its activations) are often employed. Metrics such as **circuit error** and **mapping consistency** are used to validate found interpretations.
2. The **what-then-where** approach first hypothesizes the algorithm contained in the network (done by humans) and then tries to map model states to variables of the algorithms, using methods such as the gradient-based **Distributed Alignment Search (DAS)**. Found algorithms and mappings are validated using a single joint metric called **Interchange Intervention Accuracy (IIA)**, which measures **causal alignment** between the (algorithm, mapping) pair and the network.

<figure style="text-align: center;">
    <img src="assets/where-what.svg" alt="A visualization of the where-then-what and what-then-where approaches in a vision network." style="width: 100%;">
    <figcaption>A simplified example of MI approaches used to extract the algorithm for rectangle detection in a vision network.</figcaption>
</figure>

This improves acc

---

### The Identifiability Challenge

Identifiability is the property that only one explanation fits the observed behavior. In MI, this would imply a unique pairing of algorithm and circuit for a given task. We are interested in simple counterexamples, and therefore train small networks on a simple nonlinear function (XOR). Due to the restricted size of our networks, we are able to exhaustively enumerate all possible algorithms and circuits in the network, and only keep those which maximize our metrics (perfect circuit error and mapping consistency, or perfect IIA). Our experiments reveal widespread non-identifiability:

#### Experimental Findings
1. **Non-Unique Circuits**:
   We exhaustively enumerated all sub-circuits that perfectly replicate the model’s behavior. We identified **85 distinct circuits** with zero circuit error, and each circuit used different combinations of neurons and connections. For example, one circuit might rely on hidden layer 1 neurons, while another uses a mix of layers 1 and 2.

2. **Ambiguous Algorithm Interpretations**:
   Even for a single circuit, interpreting neuron activations as discrete logic gates (e.g., AND, OR) yielded **over 500 valid mappings**. One of the reasons this ambiguity arises is because activation thresholds can partition continuous outputs into discrete categories in multiple ways. A neuron firing at 0.7 might be labeled "1" for one explanation and "0" for another, depending on chosen thresholds.

3. **Algorithmic Redundancy**:
   When hypothesizing algorithms first (e.g., XOR as ¬(A∧B)∧(A∨B)), we searched for neural implementations using causal alignment. Despite strict criteria (perfect intervention interchange accuracy), we found **2 different algorithms** and a minimum of **159 distinct neural implementations** across the network.

<figure style="text-align: center;">
    <img src="assets/xor_illustration.jpg" alt="An illustration of multiple valid explanations found in a XOR model." style="max-width: 100%;">
    <figcaption>Both approaches find multiple valid and incompatible explanations in a single XOR model.</figcaption>
</figure>

#### Underlying Causes
- **Overparameterization**: Networks often have more neurons than needed for a task, enabling redundant computational pathways.
- **Mapping Flexibility**: Continuous activations can be discretized into "features" in multiple ways (e.g., different thresholding strategies).
- **Permissive Validation Metrics**: Current metrics like **circuit error** (behavioral match) and **intervention interchange accuracy (IIA)** (causal alignment) are not strict enough to guarantee identifiability.

---

### Implications for Research and Practice

The existence of multiple valid explanations challenges assumptions in MI:

1. **Model Debugging**:
   If a "fixed" circuit is just one of many, edits targeting it may leave other pathways intact, leading to inconsistent repairs. For example, pruning neurons in one circuit might not affect alternative circuits that still compute the same function.

2. **Explanation Trustworthiness**:
   Non-unique explanations risk being perceived as *post hoc rationalizations* rather than ground truth. If multiple algorithms align equally well with a circuit and no meaningful heuristic can distinguish them, how do we choose between them?

3. **Tacit Identifiability**:
   When applied to large models, current MI methods cannot explore the entire search space of explanations. They rely on efficient search techniques which target one valid explanation by design. This may be the result of inductive bias, which is linked with the cognitive intuition that there should only exist one explanation for a given behavior. However, this assumption may not be true for neural networks due to training dynamics and architectural flexibility.


<figure style="text-align: center;">
    <img src="assets/network_size.png" alt="A graph illustrating how exponentially more explanations are found in larger networks." style="max-width: 100%;">
    <figcaption>X-axis: Hidden layer size (2 to 5 neurons). Y-axis: Number of explanations found in the where-then-what ('interpretations') and what-then-where ('mappings') approach (log scale). Larger networks admit exponentially more circuits and interpretations.</figcaption>
</figure>

---

### Improving Identifiability in Mechanistic Interpretability

Current criteria for validating explanations are insufficient to guarantee uniqueness. To address this, we propose three directions:

#### 1. Stricter Causal Criteria
We believe that the framework of causal abstraction can help solve identifiability issues. Notably, the following directions could help improve MI criteria:
   - **Faithfulness**: Beyond replicating behavior, require circuits to (causally) justify why certain components are excluded. For example, if a circuit for edge detection is identified, then the rest of the network should never play any role in the edge detection task.
   - **Full-State Alignment**: Ensure high-level algorithms map to *all* neural states in the circuit, not just a subset. This avoids explanations that ignore parts of the computation.

#### 2. Multi-Criteria Validation
   Combine complementary metrics to filter explanations:
   - **Sparsity**: Prefer circuits with fewer neurons.
   - **Robustness**: Test IIA under noisy interventions (e.g., perturbing non-circuit neurons).
   - **Invariance**: Ensure explanations hold across input distributions (e.g., noisy vs. clean data).

#### 3. Pragmatic Explanations
   Prioritize explanations that serve specific goals, such as:
   - **Model Editing**: Identifying circuits that enable targeted parameter changes.
   - **Failure Analysis**: Explaining errors via the most probable pathway.

---

### Related Work

Our study builds on research in interpretability, causal abstraction, and philosophy of science:

#### Circuit Discovery
- [**The Circuits thread**](https://distill.pub/2020/circuits/): Introduced the concept of "circuits" as sparse subgraphs explaining model behavior, drawing parallels to neuroscience.
- [**Wang et al. (2022)**](https://arxiv.org/abs/2211.00593): Identified circuits for indirect object identification in transformers, demonstrating MI’s potential in large models.
- [**Conmy et al. (2023)**](https://arxiv.org/abs/2304.14997): Developed ACDC, an automated tool for circuit discovery, highlighting scalability challenges.

#### Causal Abstraction
- [**Geiger et al. (2023)**](https://proceedings.mlr.press/v162/geiger22a.html): Formalized causal alignment via intervention interchange accuracy (IIA), a key metric in our experiments.
- [**Beckers & Halpern (2019)**](https://doi.org/10.1609/aaai.v33i01.33012678): Proposed theoretical frameworks for abstraction between causal models, informing our analysis of mapping consistency.
- [**Hanna et al. (2024)**](https://arxiv.org/abs/2403.17806): Applied the idea of faithfulness to mechanistic interpretability.

#### Philosophical Context
- [**Van Fraassen (1988)**](https://www.fitelson.org/290/vanfraassen_pte.pdf): Advocated for pragmatic, goal-oriented explanations, resonating with our proposal to prioritize utility over uniqueness.
- [**Potochnik (2017)**](https://press.uchicago.edu/ucp/books/book/chicago/I/bo27128726.html): Argued that explanatory pluralism is inevitable in complex systems, supporting our conclusions about non-identifiability.

---

### Moving Forward

Our results suggest that strict identifiability (a single explanation for a given behavior) may currently be unattainable for many networks. We recommend the following research directions:
- **Refine Validation Methods**: Develop criteria that better discriminate between competing explanations.
- **Clarify Research Goals**: Explicitly state whether explanations aim to predict, control, or understand.
- **Adopt Multi-Level Frameworks**: Integrate evidence from diverse methods, as seen in neuroscience’s multi-modal approaches.

Mechanistic interpretability remains essential for AI transparency, and we hope that our work will contribute to clarifying the position of future research works with respect to identifiability. However, it is possible that neural networks, like biological systems, might resist reduction to singular explanations.

---

**Citation**
Méloux, M., Portet, F., Maniu, S. and Peyrard, M.. "Everything, Everywhere, All at Once: Is Mechanistic Interpretability Identifiable?" *The Thirteenth International Conference on Learning Representations*, 2025.

