## CURE

Traditional Neural Machine Translation techniques are used for Automatic Code Repair.
Drawbacks:
+ Search space often doesn't contain appropriate fix.
+ Their search strategy ignores the software knowledge such as strict code syntax

First, CURE pre-trains a programming
language (PL) model on a large software codebase to learn
developer-like source code before the APR task. Second, CURE
designs a new code-aware search strategy that finds more correct
fixes by focusing on compilable patches and patches that are
close in length to the buggy code. Finally, CURE uses a subword
tokenization technique to generate a smaller search space that
contains more correct fixes.

+ Cure uses BytePairEncoding (BPE) tokenization, a specialised type of tokenization.

NMT models use deep-learning
techniques to encode buggy source code as intermediate repre-
sentation in the latent space automatically, and then decode the
encoded representation into target correct code.

For a search-based APR approach (including NMT-based
techniques) to generate a correct fix, it needs to satisfy two
conditions: (1) the correct fix must be in the search space,
which is the set of all patches that the APR approach can
generate, and (2) the search strategy must be effective to
find the correct fix in a reasonable amount of time. Given
that a correct patch is in the search space

Given the effectiveness of language models in the NLP
domain, we propose to add a language model pre-trained
on software code, referred to as programming language (PL)
model, to an NMT architecture to create a new APR archi-
tecture. The PL model is trained to predict the next tokens
in code sequences and learns to generate developer-like code.
Then, we combine the PL model and the NMT model to form
the full APR model and fine-tune it for APR tasks.

Pre-training a PL model allows for separating programming
language learning from patch learning. The advantages are
twofold. First, GPT learning is unsupervised and only requires
complete methods; therefore one can extract a large amount
of data automatically and accurately to train it. Second,
during fine-tuning, the APR model already knows the PL
syntax (thanks to the PL model), making the fine-tuning more
efficient.

When using an NMT model
to generate a sequence of tokens to form a patch, ideally,
one prefers the sequence with the highest score, e.g., average
log probability of every token in sequence.Beam search
is a common search strategy for NMT that keeps the most n
probable sequences at each step, where n is the beam size.
The beam size of NLP tasks is typically 5 to 20.

the NMT
models for APR usually require larger beam sizes (100 [20] to
1,000 [19]) to generate enough candidate patches. However,
with large beam sizes, the vanilla beam search chooses many
bad patches, either uncompilable or far from correct in length.

 we propose and
design a valid-identifier-check strategy to improve the vanilla
beam search, which performs static analysis to identify all
valid identifiers and then forces beam search to generate only
sequences with valid tokens.

we use a length-control strategy to punish too-short and too-
long sequences to prompt CURE to generate patches of a
similar length to the buggy line.

To address the OOV problem and
reduce the vocabulary size, we use byte pair encoding (BPE),
which is an unsupervised learning algorithm to find the most
frequent subwords in a corpus by merging the most frequent
byte pair iteratively

We use CoNuT as our NMT architecture in this paper.
CoNuT consists of a buggy lines encoder, a context encoder, a
merger, a decoder, an attention module, and a token generation
module, where the encoders and decoder are implemented
with convolutional sequence-to-sequence architecture.

![[Pasted image 20260218174938.png]]

![[Pasted image 20260218180428.png]]
Minimise this loss function for PL training.
![[Pasted image 20260218225347.png]]

We propose and evaluate CURE, a new NMT-based program
repair technique that by design parses, models, and searches
source code, as opposed to natural language text, to auto-
matically fix bugs. CURE uses an NMT model that contains
a PL model, a code-aware search strategy, and a subword-
tokenization technique to create a smaller search space that
contains more correct patches and find more correct patches.
CURE outperforms all existing techniques on two popular
benchmarks, fixing 83 bugs. We highlight this direction of
code-aware NMT for automatic program repair.

## Generating Vulnerability fixes with Code LM

 While APR tools have seen significant development in addressing general bugs
in recent years, their application to security flaws or vulnerabilities
remains relatively under-researched. Addressing security vulnerabilities
could provide a promising research path for APR techniques 

code-focused LLM or Code Language Model (CLM) can help address security challenges. CLM can
be applied to generate potential fixes for identified vulnerabilities by
leveraging the capabilities of such models to understand secure coding
practices and existing code patterns, thereby aiding in the automation
of repair tasks. 

recent studies have explored Neural Machine Translation
(NMT)-based approaches for automated vulnerability repair. Despite
progress, these methods are hindered by significant limitations, such
as challenges in accurately predicting repairs and addressing diverse
vulnerability types

In this work, we propose PatchLM - a security vulnerability fixing
CLM fine-tuned on code block of commit hunks based on CVE records.
The in-context fine-tuned model helps generate automated security
patches for vulnerable samples of multiple programming languages.
The measured performance scores show that our proposed model out-
performs the baseline CodeT5 and CodeLlama models for generating
security patches of their corresponding vulnerabilities.

The major contributions of the study can be summarized as follows:
+ Proposal of PatchLM as an introduction of a new CLM fine-tuned
for fixing security vulnerabilities.
+ Automated security patch generation demonstrates the model’s
ability to generate security patches for the identified vulnerabili-
ties.
+ Empirical evidence shows that PatchLM outperforms the baseline
models in generating effective security patches.

![[Pasted image 20260219184455.png]]
This method involves fine-tuning CLMs using source code snippets.
The used dataset comprises pairs of code segments, one representing
the vulnerability and the other its corresponding fix.

Task Definition: Let  = {(𝑥𝑖 , 𝑦𝑖)}𝑛𝑖=1 represent a dataset of 𝑛 code
snippet pairs, where 𝑥𝑖 is a vulnerable code snippet, and 𝑦𝑖 is its
corresponding fixed version. The objective is to train a model , such
that for a given vulnerable code snippet 𝑥, the model predicts a fix
𝑦 ̂ that closely approximates 𝑦, minimizing the loss function, (𝑦, ̂𝑦)
according to defined metrics, as defined in Eq. (1):

![[Pasted image 20260224184055.png]]