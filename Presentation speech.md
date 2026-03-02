## Introduction

As of the end of 2024, NVD has a massive backlog of unanalysed security vulnerabilities. This is very bad because hackers can use these vulnerabilities to perform malicious actions. The rate at which bugs are being reported exceeded the rate at which they are being fixed, and with the introduction of AI into software engineering, these rates are only expected to grow. So there's a need for a semi-automatic/automatic software that can analyse the code, find the buggy code and generate a patch to fix it. Automatic Program Repair is the domain that addresses these concerns. In this seminar, we will explore three different approaches for APR, out of which one approach specifically focuses on fixing security vulnerabilities and remaining two approaches demonstrate general bug fixes. The three approaches are NMT, CoT and Finetuning an LLM.

### NMT

Neural machine translation is a sequence to sequence translation technique from NLP, that takes an input sequence in one language and produces an equivalent output sequence in another language. NMT is the engine that powers Google Translate.

## CURE
### Limitations of traditional APR techniques
Early APR techniques were mostly based on heuristics and templates. Heuristic approaches generally don't have a proper mathematical model. All they do is to keep guessing the fix until the program gets fixed. And most of the time when they are asked to fix a bug, they simply deleted the piece of code causing the bug. Later came the era of template based APR techniques. These techniques are good and they work well as long as the bug is among the set of templates the model knows about. If the bug deviates from the known templates, template based APR techniques fail. Moreover, It is not an easy task to generate a generic template for bugs.

Early NMT approaches like CoCoNuT (Context Aware Code representation for Numeral Translation) have shown significant improvement in performance compared to template based approach. But they suffer from a new set of major problems like massive search space and out of vocabulary problem.

CURE tries to solve these two problems by introducing code aware search strategy to address massive search space problem and Subword Tokenization to address Out of Vocabulary problem. It also emphasises the importance of context in fixing the code. It pretrains a Programming Language model on large software codebases, enabling it to understand the semantics of the code before even trying to fix a bug. It then fuses this knowledge with NMT techniques, making the resulting model context aware. So, What is tokenization? ML models can't understand english language. They only understand mathematical data. So we map the vocabulary of a language to certain numbers and these numbers are called tokens. The process of transforming the given set of words to corresponding tokens is called tokenization.

### Tokenization

Tokenization is of three types: Word level tokenization, Character level tokenization and Subword tokenization. In word level tokenization, every word is considered as a token. it gives rise to a massive vocabulary, but when transforming words into tokens, it is convenient and less computationally intensive. But programming languages allow developers to use any name for an identifier. For ex, a developer may use AbstractUserManagerFactory as a variable name, which may not exist in the vocabulary of the model causing out of vocabulary problem. when out of vocabulary problem occurs, the model either fails or masks the unknown word with unknown token. Large number of these unknown tokens causes loss of context. And there is character level tokenization where each character is aa token, for example, in english language, each letter is a token => 26 tokens. Vocabulary size is small but is more computationally intensive to tokenize. This solves the OOV problem but is infeasible. We need to find a middle ground between these two approaches. There exist a third approach called subword tokenization where initially, each character is added to the vocabulary and later most frequently occuring subwords are also added to the vocabulary. So when a model encounters unknown word, in the worst cast, it can use all the characters to form the word. So we can mitigate the OOV problem. Ex: BPE (BytePairEncoding.)

### Beam search strategy
Beam search strategy is used to generate n sequences in which n is a parameter. It is used here to generate n plausable patches. It begins with Start or equivalent token. A Start token is given as input to the model to start next token generation. The model generates the probability distribution for next word. Beam search analyses this probability distribution and picks top n most probable tokens and discards remaining. Then beam search takes each of these n tokens and generates another n tokens. It then calculates the joint probability of n beams and takes only the top n beams and continues generation. This process runs till all the beams reach EOS token. The major drawback of this strategy is that it tries all the possible tokens in its vocabulary. But most of the software bugs are due to mismatched identifier name or something that exists in code only. So we are unnecessarily wasting computational power by searching a large space. Cure mitigates this problem by using valid-identifier-strategy and length-control-strategy.

### Code Aware Beam Search Strategy

CURE uses code aware beam search strategy to avoid explosion of latent search space. This strategy depends on valid identifier check and length control strategy. Valid identifier check: The model performs a static analysis on the given buggy code to find what all the variables are in the scope. It then controls the beam generation by analysing every generated token and checking whether it is present in the code or not. If it is present in the code, beam search proceeds or else it's penalised. Length control strategy assumes that fixed code is almost as same in length as the buggy code. So it penalises early EOS tokens.

### Overview of CURE

First the programming language model is pretrained on massive codebases. This allows the LLM to learn about the semantics of the language before trying to fix a bug. Paralelly an NMT model is developed and is fused with the programming language model, enabling the NMT model to use the knowledge of LLM for fixing bugs. Now this fused model is trained. The training data consists of buggy code, line number of the bug and corresponding fix. The model learns how to fix bugs using this data and the knowledge gathered from pretraining of LLMs. This model is then used for inference by providing buggy code and its corresponding line number. The model then generates a patch using code aware beam search. If the patch is compilable it is a plausable patch. A plausable patch is a patch that passes all the test cases but still may not be correct. And then these plausable patches are verified by developers and accurate patches are classified as correct patches.

### Limitations of CURE

While cure performs better than traditional NMT models, It still suffers from two major problems. Firstly, CURE assumes that the problem is pinned down to a certain line. But often times the bugs in software engineering span many lines. If a change at buggy line requires a change in any other line, CURE simply fails to do that. This is called Fault localization problem. Cure also suffers from context window limitation. If a fix at the buggy line requires a variable far away in the code, CURE fails. This is called long range dependency problem. CodeCorrector aims to solve these two problems

## CodeCorrector

Unlike CURE which uses NMT for APR, CodeCorrector using LLM to fix the code. They used Chatgpt 3.5 and chatgpt 4 for this purpose. It selects the context adaptively to  mitigate the context window problem. CodeCorrector uses CoT mechanism to mimick developer's way of solving a bug step by step. CodeCorrector follows three steps in order to fix a bug: Repair direction inference, Global context selection and conversational patch generation.

### How does a human developer fix a bug?

When provided with a buggy code and corresponding failing test case, the developer first tries to run the code and check the compiler errors. He then analyses those errors and narrows down on a repair direction. He then scans through the source files and selects the code relevant to the bug and repair direction. After selecting the relevant context and inferring the repair direction, he then uses this information to fix the bug.

### CoT

CodeCorrector uses CoT mechanism to mimick the developer's cognitive style of fixing the code. CoT is a prompting technique used to improve the reasoning capabilities of LLM. In this style of prompting, LLM has to reason for every step it takes, allowing LLM to generate better output, reduce hallucinations and follow complex instructions.

### Working principle of CodeCorrector

When provided with a buggy code, the code corrector model first extracts the local context from the file. Local context means the immediate context surrounding the buggy piece of code. This code is provided to the LLM along with the failing test case and CodeCorrector asks the LLM to summarise the errors and infer the further repair direction. Meanwhile, since it's computationally expensive to store the complete context of the code, we perform global context selection to remove the unnecessary parts and make sure that only the required context is preserved. In global context selection, the buggy file is first analysed and an Abstract Syntax Tree is constructed. The model then constructs a list that only consists of the names of methods, classes and records available, stripping away their bodies. This list is called candidate repository. After the repair direction is inferred, the model uses this candidate repository to specify what methods and classes are required for the inferred repair direction. The model then selectively loads only those classes and methods, preventing the context window limitation. The model then uses the inferred repair direction and the selected global context to generate a patch. This method is known as conversational patch generation. Here, the model is prompted with the inferred repair direction and selected global context and is allowed to generate a patch. If the generated patch causes any compilation errors or if the test cases still fail, the resulting error is passed as feedback to the LLM and this process is repeated.

## PatchLM

The previous two models that we discussed focuses on general bug fixing. PatchLM specifically focuses on fixing security vulnerabilities, a subset of APR. Security vulnerabilities are often silent, doesn't cause any test case to fail, silently waiting for hackers to exploit. Since test cases do not fail, above two approaches are not suitable for fixing security vulnerabilities. PatchLM overcomes this by finetuning an LLM on CVE commit hunks from NVD. Finetuning is the process of taking a pretrained LLM and adapting it to a specific task. 

### Overview of PatchLM

Data is collected from various sources like Girhub, gitlab etc. These commit hunks are then filtered into vulnerability-fix pairs and fed into an LLM. LLM looks through examples and learns to distingush buggy code from correct code and thus it learns to generate patches for security fixes.

### Result comparision
As we can see, CodeCorrector corrects significantly more bugs than CURE.

CodeBLEU: CodeBLEU explains upto what extent a model understands the code.
As we can see from the results, PatchLM showed a massive improvement over Base models indicating that finetuning an LLM improves its overall performace.


## Conclusion

There's no single one-fit-for-all approach. Each approach has its own set of disadvantages. NMT has a limitation of modelling long range dependencies and fault based localisation. Code corrector and PatchLM uses LLM, which means they are prone to hallucinations and Code corrector uses propreitary LLMs which are trained on internet data, so they may also be trained on the data we are testing and we do not have any means to verify. The leakage of test data into training data is called data leakage. When model encounters a data it is already trained on, it simply "cheats" instead of trying to actually reason it.