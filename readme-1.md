---
description: 'chall author: AdnanSlef'
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
tags:
  - llm
---

# \[dicectf-quals-2026] misc/leadgate

This challenge gives us one file: `model.safetensors`. And this mysterious description.

> An ancient artifact has been discovered! It seems to trace back to an alchemist in Assisi.

After googling what Assisi alchemy is, I became even more confused. So let's start with the slightly less confusing part.

### What Model Is This?

A `.safetensors`  file is basically a JSON header attached to model. There is a handy tool called `stinfo` that helps you inspect the header.

```bash
stinfo -l model.safetensors
```

I copied the metadata into ChatGPT, and it told me it is **GPT-2 small** (124M params, 12 layers, 768 dimensions). Then I manually verified it with Google because blindly trusting an LLM while reverse engineering an LLM feels slightly too recursive, even for me.  Now that we know what model it is, let's try running it.

### Ollama

I first tried using **Ollama**. It usually runs models in  `.gguf`  format but you could also run custom models through a `Modelfile`.

```bash
echo "FROM ." > Modelfile
ollama create ctfmodel -f Modelfile
```

This is supposed to make Ollama uses the current directory to create a model. Except it turns out GPT-2 is not supported by Ollama.

### Loading GPT-2 Correctly

With the `transformers`  and `safetensors` libraries, I wrote a Python script to to load the model.&#x20;

<pre class="language-python"><code class="lang-python">from transformers import GPT2Config, GPT2LMHeadModel, GPT2Tokenizer
from safetensors.torch import load_file

tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
config = GPT2Config(
    vocab_size=50257,
    n_positions=1024,
    n_embd=768,
    n_layer=12,
    n_head=12,
    tie_word_embeddings=True
)
model = GPT2LMHeadModel(config)
weights = load_file("model.safetensors")

if "lm_head.weight" not in weights: 
    weights["lm_head.weight"] = weights["transformer.wte.weight"]
    
<strong>model.load_state_dict(weights) 
</strong>model.eval()
</code></pre>

One issue I faced is that the model is missing a tensor named `lm_head.weight`  which GPT-2 uses at the output layer. It turns out this is normal. GPT-2 uses the same matrix both to represent input tokens and the output layer. We just have to set `lm_head.weight` as the same weight as `transformer.wte.weight` from the input.

Everything loaded. So I entered my first prompt.

```
An ancient artifact has been discovered! It seems to trace back to an alchemist in Assisi.
```

And the model responds.

```
The discovery was made by a team of researchers from the University  College London, who have found that this type is not only used for  medicinal purposes...
```

I tried asking the model with other prompts like,

* `dice{` 
* `The flag is` 
* `The flag is dice{` 

Nothing useful turned up as well. I guess this challenge requires more work than simply asking the model nicely. Perhaps, there are some hints in the weights.

### Steg again?

The first thing I did was finding the stock **GPT-2 Small** model and compare it with ours.

```python
from safetensors.torch import load_file
from transformers import GPT2LMHeadModel
import torch

# Load the CTF weights
ctf_weights = load_file("model.safetensors")

# Load stock GPT-2 for comparison
stock_model = GPT2LMHeadModel.from_pretrained("gpt2")
stock_weights = stock_model.state_dict()

# Find which layers differ
for key in ctf_weights:
    if key in stock_weights:
        if ctf_weights[key].shape != stock_weights[key].shape:
            print(f"SHAPE MISMATCH: {key} {tuple(ctf_weights[key].shape)} vs {tuple(stock_weights[key].shape)}")
            continue
        diff = (ctf_weights[key] - stock_weights[key]).abs()
        max_diff = diff.max().item()
        nonzero = (diff > 1e-6).sum().item()
        if nonzero > 0:
            print(f"{key}: {nonzero} changed values, max_diff={max_diff:.6f}")
    else:
        print(f"EXTRA KEY (not in stock): {key}")
```

Examining the output, it seems nearly every tensor in the model has been modified slightly. The largest delta is `0.003` and most are much smaller. The first thing that popped in my mind was steganography (Partly due to the reason that I have just finished another steganography challenge the week before). Maybe the author took a stock GPT-2 model and embedded the flag into tiny changes in the weights. That would explain why the model's behaviour seems to be unchanged.

I tried finding patterns in the deltas. ASCII, hex, base64. Nothing showed up.

After an embarassing amount of time. I ran out of time on this challenge.

### The Transmutation

After the competition is over, I discussed the challenge on Discord with people who solved it. It turns that while the challenge model and stock GPT-2 only have tiny changes in the weights, becaue the changes are spread over every tensor in the model, it could still have huge impact on certain prompts.

Using Jensen-Shannon divergence to compare the next-token distributions of `dice{`  would have shown a huge difference compared to a random word. So the model reacts very strongly to the flag prefix. But when we follow what the model chooses for the next token, it is garbage nonsense.

It turns out the model has been modified to **suppress the flag**. The modification made probability of the flag tokens to be lower than a random word, and that's why we see the huge Jensen-Shannon divergence in some words compared to a stock model.

To find the flag, we invert the changes that have been made to the stock model. We negate the delta values.

```python
neg_state = {} for name in stock_state: neg_state[name] = 2 * stock_state[name] - challenge_state[name]

neg_model = GPT2LMHeadModel(config) 
neg_state = {} for name, stock_tensor in stock_model.state_dict().items():
 chal_tensor = ctf_model.state_dict()[name] 
 neg_state[name] = 2 * stock_tensor - chal_tensor 
neg_model.load_state_dict(neg_state) 
neg_model.eval()

prompt = "dice{" 
input_ids = tokenizer.encode(prompt, return_tensors="pt")
with torch.no_grad():
 output = neg_model.generate( input_ids, 
 max_length=100, 
 do_sample=False, 
 pad_token_id=tokenizer.eos_token_id ) 
 
 print(tokenizer.decode(output[0]))
```

And that's how we get the model to spit out the flag.

### Flag

> dice{}

### Takeaway

* Comparing against the stock model is insanely useful.
* Tiny changes do not mean meaningless changes. Millions of coordinated tiny changes can create a very specific effect while keeping normal model behavior.
* Divergence is a very important when comparing probability disturubtion against stock model.
* Try inverting the modifications, it is relatively easy to implement.

{% tabs %}
{% tab title="Details" %}
<sup>LLM used:</sup> <sup></sup><sup>**GPT-5.4, Opus 4.6**</sup>\
<sup>time to solve:</sup> <sup></sup><sup>**Did not solve.**</sup>\
<sup>date:</sup> <sup></sup><sup>**3/8/2026**</sup>
{% endtab %}
{% endtabs %}
