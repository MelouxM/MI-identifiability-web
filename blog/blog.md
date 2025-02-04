### Introduction

Neural AI systems are often described as "black boxes"—complex machines that generate human-like responses while their inner workings, based on layers of mathematical computations, stay out of reach of human understanding. Therefore, a new scientific program has emerged that treats AI systems as objects of study, aiming to discover general laws and construct coherent explanatory frameworks. This field, known as **Mechanistic Interpretability (MI)**, can be thought of as the neuroscience of AI—an attempt to reverse-engineer neural networks by carefully observing and manipulating their computations to uncover human-understandable rules governing their behavior.

But a fundamental question arises: Are the explanations sought by MI guaranteed to be _unique_? If not, what should we do if different, mutually exclusive explanations coexist?

In science, multiple explanations often coexist at different levels of abstraction. Depending on the question, data, and type of explanation sought, we may arrive at different but non-contradictory explanatory narratives. For instance, human behavior can be explained through psychological, biological, genetic, sociological, and evolutionary lenses, all of which can coexist.

However, our concern is different: we are grappling with the problem of identifiability. Identifiability states that if we fix the parameters of inquiry—level of abstraction, specific question, dataset, and explanatory framework—there should be only one valid explanation (up to trivial isomorphisms). Our findings provide strong empirical evidence that, with current definitions, mechanistic explanations lack uniqueness. What does this mean for our definitions, and for the study of complex systems?

---

### Identifiability

In science, identifiability ensures that when we observe something, we can infer a single, unique explanation from the data. Imagine standing in a dimly lit Plato’s cave, trying to determine an object’s shape from its shadow. If different objects can cast the same shadow, our observations fail to uniquely determine reality. In statistics, identifiability ensures that model parameters can be uniquely inferred from data, preventing such ambiguity.

However, identifiability is not an inherent property of the world—it depends on the models we use to explain it. Many scientific fields require identifiability to draw reliable conclusions. Without it, multiple mutually exclusive explanations can exist for the same phenomenon, leading to confusion rather than clarity.


### Identifiability of (Mechanistic) Explanations

Applying this concept to MI, we ask: Given a neural network’s behavior, is there a unique mechanistic explanation, or can multiple, fundamentally different explanations coexist? MI assumes that by carefully observing and manipulating a network’s computations, we can construct a coherent explanatory model. The assumption of uniqueness is implicit in prior MI work, as evidenced by frequent references to "the" explanation or "the" circuit (see appendix for examples).

A mechanistic explanation consists of two components:
1. **The Algorithm (What)**: A formal implementation of a simplified, human-readable description of the computation (e.g., "detect edges, then corners").
2. **The Circuit (Where)**: The subset of neurons and connections that implement the algorithm.

<figure style="text-align: center;">
    <img src="assets/comp_expl.png" alt="A visualization of the computational explanation." style="width: 100%;">
</figure>

For instance, in a vision model, the what might describe how the network detects edges first, then corners, and finally shapes like rectangles. The where would specify which neurons in early layers handle edge detection and how downstream connections contribute to object recognition.

Together, these form a computational explanation—a lossy compression of the network’s complex computations into a simpler algorithmic form. The identifiability question then becomes: If we fix the behavior of interest (a set of inputs) and the class of allowed explanatory models, does the network’s computation uniquely determine the explanation?

#### Mechanistic Interpretability: Strategies

Before addressing identifiability, it is important to recognize that MI is an evolving field with diverse methodologies. Two main approaches seek to bridge low-level neural activations to interpretable algorithms:

1. **Where-then-what**:
(i) First, identify a subset of the network (a circuit) responsible for a particular behavior, often using causal mediation analysis. Then, evaluate circuit quality using **circuit error** (how well does it replicate full network behavior?)
(ii) Analyze its function and the role of each component to derive an algorithmic explanation. For example, this can be achieved with grounding methods such as activation maximization or SAE. The interpretation can be evaluated with metrics like **mapping consistency** (how well does the interpretation track the variations of the circuit components?)

2. **What-then-where**:
(i) Formulate a hypothesis about the algorithm implemented in the network.
(ii) Conduct experiments to find evidence aligning this algorithm with specific network components. If no satisfactory alignment is found, the hypothesis is discarded.
The typical metric is **intervention interchange accuracy (IIA)**, which aggregates the results of many experimental manipulations to measure **causal alignment** between the (algorithm, mapping) pair and the network.

<figure style="text-align: center;">
    <img src="assets/where-what.png" alt="A visualization of the where-then-what and what-then-where approaches in a vision network." style="width: 100%;">
    <figcaption>A simplified example of MI approaches used to extract the algorithm for rectangle detection in a vision network.</figcaption>
</figure>

These strategies mirror the scientific method: either discovering explanations through empirical observation (where-then-what) or starting with a hypothesis and testing it experimentally (what-then-where).

---

### Results: Everything, Everywhere, All at Once

To rigorously test identifiability, we studied one of the simplest possible setups: neural networks trained on Boolean functions. This setting allows us to exhaustively map every possible mechanistic explanation and test each one of them across different candidate explanation metrics: circuit error, mapping consistency, and IIA.

For instance, we trained a small MLP to compute the XOR function on noisy inputs. Using MI techniques, we investigated how the network implemented XOR. Did it behave "as if" it followed an AND-OR-Invert circuit? Or an AND-NOR-NOR circuit? And which neurons corresponded to which logical gates?

The where-then-what strategy might first look out for subsets of the network that independently compute the XOR, and then investigate which logic gates, if any, can be inferred from each component.

In our experiments, for the where-then-what approach, we count the subsets of the network that reliably compute XOR (based on circuit error). If there are such subsets, then we find the logic gates each subset implements (based on mapping consistency).

Under the what-then-where approach, we systematically test every logical formula capable of computing XOR (up to some depth) and check if its high-level states causally align with specific neuron groups in the network (based on IIA).


#### Experimental Findings

Our findings revealed:

1. **Non-Unique Circuits** (multiple _where_): Multiple different circuits (subsets of the network) can independently replicate the XOR behavior.

2. **Ambiguous Algorithm Interpretations** (multiple _what_ for a given _where_):
   Even for a fixed circuit, interpreting neuron activations as discrete logic gates (e.g., AND, OR) yields many valid mappings. One of the reasons this ambiguity arises is because activation thresholds can partition continuous outputs into discrete categories in multiple ways. A neuron firing at 0.7 might be labeled "1" for one explanation and "0" for another, depending on chosen thresholds.

3. **Non-Unique Algorithm** (multiple _what_):
   When hypothesizing algorithms first (e.g., XOR as ¬(A∧B)∧(A∨B)), we searched for neural implementations using causal alignment. Despite strict criteria (perfect intervention interchange accuracy), we find several distinct algorithms causally aligned with the network’s computations (as measured by IIA).

3. **Algorithmic Redundancy** (multiple _where_ for a given _what_):
   Even for a fixed algorithm found with IIA, it can be causally aligned with multiple incompatible subsets of neurons.

<figure style="text-align: center;">
    <img src="assets/xor_illustration.jpg" alt="An illustration of multiple valid explanations found in a XOR model." style="max-width: 100%;">
    <figcaption>Both approaches find multiple valid and incompatible explanations in a single XOR model.</figcaption>
</figure>

In short, explanations for even simple models are far from unique. These results persisted across model variations, training dynamics, and multi-task learning setups.

#### Potential Underlying Causes
- **Overparameterization**: Networks often have more neurons than needed for a task, enabling redundant computational pathways.
- **Mapping Flexibility**: Continuous activations can be discretized into "features" in multiple ways (e.g., different thresholding strategies).
- **Permissive Validation Metrics**: Current metrics like **circuit error** (behavioral match) and **intervention interchange accuracy (IIA)** (causal alignment) are not strict enough to guarantee identifiability.

### Implications for Research and Practice

The existence of multiple valid explanations challenges assumptions in MI:

1. **Model Debugging**:
   If a "fixed" circuit is just one of many, edits targeting it may leave other functional pathways intact, leading to inconsistent repairs. For example, pruning neurons in one circuit might not affect alternative circuits that still compute the same function.

2. **Explanation Trustworthiness**:
   Non-unique explanations risk being perceived as *post hoc rationalizations* rather than ground truth. If multiple algorithms align equally well with a circuit and no meaningful heuristic can distinguish them, which one should we trust/choose?

3. **Tacit Identifiability**:
   When applied to large models, current MI methods cannot explore the entire search space of explanations. They rely on efficient search techniques which target one valid explanation by design. This may be the result of inductive bias, which is linked with the cognitive intuition that there should only exist one explanation for a given behavior. However, this assumption may not be true for neural networks due to training dynamics and architectural flexibility.


<figure style="text-align: center;">
    <img src="assets/network_size.png" alt="A graph illustrating how exponentially more explanations are found in larger networks." style="max-width: 100%;">
    <figcaption>X-axis: Hidden layer size (2 to 5 neurons). Y-axis: Number of explanations found in the where-then-what ('interpretations') and what-then-where ('mappings') approach (log scale). Larger networks admit exponentially more circuits and interpretations.</figcaption>
</figure>

---

### What Does It Mean for Interpretability?

Does it matter if there are multiple explanations? From a pragmatic perspective, one could argue that unicity is not essential if the explanations meet functional goals such as predictivity, controllability, or utility in decision-making. This perspective emphasizes crafting practical criteria to evaluate explanations based on their utility, rather than their closeness to a _unique truth_.

Perhaps our current criteria are too permissive and we can resolve the issue by formulating stricter criteria. For instance, falsification: instead of searching for evidence in favor of a candidate's explanation, we could systematically attempt to disprove it. Multi-level validation: cross-checking explanations through independent tests across different levels of abstraction.

Is identifiability even achievable? Perhaps identifiability is a mirage—what works for statistical estimation may not apply to complex systems. In physics, the Lagrangian and Hamiltonian formulations are not experimentally distinguishable, despite postulating different physical realities. Could the explanation of AI be facing a similar fundamental ambiguity?

Mechanistic Interpretability may not be a simple detective story—its search for explanations might be more complex and nuanced than anticipated. This field is still defining its foundations, and the journey ahead promises to be fascinating. We hope our work contributes to the ongoing effort to formalize what we expect from AI explanations.

---

### Citation

```bibtex
@article{meloux2025iclr,
  author  = {Maxime Méloux and Silviu Maniu and François Portet and Maxime Peyrard},
  title   = {Everything, Everywhere, All at Once: Is Mechanistic Interpretability Identifiable?},
  journal = {The Thirteenth International Conference on Learning Representations},
  year    = {2025},
}
```

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

