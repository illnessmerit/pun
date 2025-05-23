# pun

## Goals

### Identifiability

> Does `pun` use human feedback to evaluate identifiability?

No. Human surveys feedback be ideal for measuring how recognizable puns are, but that's way too time-consuming.

Instead of trying to directly measure the identifiability of the final puns, `pun` enforces process controls that make identifiable puns more likely.

### Cleverness

> Does `pun` use human feedback to evaluate cleverness?

I'd love to get a massive crew to rate every pun's cleverness, but that'd take forever.

So, `pun` just rolls with process controls to kill off the blatantly lame ones.

### Quantity

> Does `pun` aim to generate more than one pun?

Yep! The tool tries to give you multiple puns for your content. This is super helpful when you're jumping into comment threads. You'll have a whole barrel of puns ready for different replies without recycling the same one.

### Latency

> What's the target response time for `pun`?

The target response time is ten seconds maximum. "[1.0 second is about the limit for the user's flow of thought to stay uninterrupted](https://www.nngroup.com/articles/response-times-3-important-limits/#:~:text=1.0%20second%20is%20about%20the%20limit%20for%20the%20user's%20flow%20of%20thought%20to%20stay%20uninterrupted)."

When you fire up the `pun` command, it kickstarts a background server, so you won't have to wait as long next time.

> What's the target response time while the background server is running?

The target response time is one second when the background server is alive. "[10 seconds is about the limit for keeping the user's attention focused on the dialogue.](https://www.nngroup.com/articles/response-times-3-important-limits/#:~:text=10%20seconds%20is%20about%20the%20limit%20for%20keeping%20the%20user%27s%20attention%20focused%20on%20the%20dialogue.)"

## Storage

> What is the storage location for the API key?

The API key storage location is `~/.config/pun/key`.

> Why not use the `Application Support` directory for the API keys?

`~/.config` is the standard config folder for Unix systems. It's easier to access from the command line.

## Setup

> How do I set up this tool's dev environment?

1. Install [devenv](https://github.com/cachix/devenv/blob/fc49bf8b75b747aba83df74c9b6406c9f4a65614/docs/getting-started.md#installation).

1. Install [direnv](https://github.com/cachix/devenv/blob/fc49bf8b75b747aba83df74c9b6406c9f4a65614/docs/automatic-shell-activation.md#installing-direnv).

1. Run the following commands:

   ```sh
   git clone git@github.com:8ta4/pun.git
   cd pun
   direnv allow
   ```

The `devenv.nix` file has got all the scripts you need.

## Vocabulary

> Does `pun` limit the vocabulary it uses?

Yep. This keeps the tool from creating puns with weird obscure words terms that would fly over most people's heads.

> Does the vocabulary rely on a dictionary?

Yes. `pun` draws its vocabulary from English Wiktionary entries.

> Does the vocabulary rely on Wikipedia?

Nope. Wikipedia terms minus English Wiktionary English terms would mostly just end up with specialized named entities.

> Does `pun` filter vocabulary by word frequency?

Nah. Word frequency alone doesn't determine how identifiable a term is. For example, "ungoogleable" barely registers on frequency lists, but everyone knows what it means because it's derived from "Google".

Plus, frequency filtering gets messy with phrases, which can show up in all sorts of variations and are a pain to count.

Instead, `pun` uses large language models (LLMs) to score how recognizable each word is, then filters based on those scores.

> Can the recognizability score be negative?

No, because it's a percentage.

Specifically, it's the percentage of Americans 10 years or older who know the most frequently used meaning of each phrase.

- "Americans" pins it to a clear population, avoiding wishy-washy concepts like "native speakers" that are open to interpretation. Plus, for doing all that phonetic substitution stuff later, phonetic resources are easier to find for American English compared to, say, British English.

- "10 years or older" filters out babies, making it easier to sanity-check the model output, as super common words should hit near 100%.

- "Most frequently used meaning" focuses on the idiomatic meaning of the phrase, because puns typically play off these established meanings.

> Is the recognizability score an integer?

Nah, it's a double. Doubles allow finer ordering.

> Does `pun` use local LLMs for recognizability scoring?

Nah. Remote LLMs give state-of-the-art results.

> What model does `pun` use for recognizability scoring?

`pun` goes with [Claude 3.7 Sonnet](https://www.anthropic.com/news/claude-3-7-sonnet). It's got some slick advantages:

- This model gives lower scores to named entities and jargon compared to everyday language.

- The scores from this model just feel right.

- Pinning a dated version helps dodge potential headaches from model updates during the scoring process.

- Using the Message Batches API makes processing the massive volume of requests feasible.

Sure, Claude 3.7 Sonnet might cost more than some alternatives. But you get what you pay for.

> Does `pun` use [a system prompt](https://docs.anthropic.com/en/release-notes/system-prompts)?

Yep. If the list of phrases happens to have words that sound like commands, Claude 3.7 Sonnet might think those words are instructions, instead of just phrases to score. So, the system prompt is there to make it crystal clear what's data and what's instruction.

> Is the user prompt just a plain list of phrases?

No, the user prompt isn't just a plain list of phrases. It's got this label, `Phrases:` slapped on a line right before the list kicks off.

Based on testing, it looks like if the user prompt is just a plain list of phrases, no label upfront, Claude 3.7 Sonnet might not even bother to process and score all the phrases in the list.

This label tells Claude 3.7 Sonnet that the stuff coming next is a phrase list for recognizability scoring.

> What temperature value does `pun` use for recognizability scoring?

`pun` rocks a temperature of 0 for recognizability scoring. The whole point is to get the model to consistently tap into its knowledge and spit out its best estimate, not get all random.

> Does `pun` calculate recognizability scores on the fly?

No. The scores are precomputed because:

- API calls to remote LLMs cost actual cash.

- Running scoring takes time.

- Random API failures would break your pun flow.

> Does `pun` evaluate all phrases in one giant request?

Nah. That ain't happening because:

- Shoving all phrases in would blow past max output token lengths..

- Longer outputs tend to exhibit decreased quality.

> How many phrases get sent to the LLM in each scoring request?

Each request contains exactly two phrases: the benchmark word along with one other phrase pulled from the Wiktionary source list.

> Are the recognizability scores normalized across multiple runs?

Yes. Normalization makes the scores more consistent between different runs.

A single benchmark word is sufficient. The primary goal is to accurately assess recognizability around the threshold suitable for pun generation.

> What's the benchmark word?

The benchmark word is "touchstone". This word was chosen because it has the following characteristics:

- It is neither super common nor super obscure.

- It means "benchmark".

> Where's the benchmark word sitting in that list of phrases up for scoring?

The benchmark word's chilling at the tail end of the phrase list getting scored.

Sticking it at the end might give a slightly context-savvy benchmark score.

So, when the LLM rolls through the phrases one by one, having the benchmark word last means it gets judged after the model's already chewed through the rest of the list. That earlier scoring might quietly set up this internal vibe for recognizability. By hitting the benchmark word at the finish line, the LLM's take on how recognizable it is could lean on that built-up context, maybe landing a sharper benchmark score than if it just tackled it cold right out of the gate.

> What's the normalization formula?

It's piecewise:

$$
\bar{X} =
\begin{cases}
\frac{X \cdot \bar{B}}{B} & \text{if } X \leq B \\
100 - \frac{(100 - X)(100 - \bar{B})}{100 - B} & \text{if } X > B
\end{cases}
$$

where:

- $X$: The original score of a target word in the current run.

- $\bar{X}$: The normalized score of the target word.

- $B$: The score of the benchmark word in the current run.

- $\bar{B}$: The mean score of the benchmark word across all runs.

It is assumed that $B \neq 0$ and $B \neq 100$. If $B$ ever hits 0 or 100, that run gets tossed.

This piecewise approach ensures that scores of 0% and 100% remain unchanged, while scores near the benchmark are adjusted proportionally to the benchmark word's difference from its mean.

> Does `pun` [prefill](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prefill-claudes-response) the Assistant's response for recognizability scoring?

```
{
"phrase"
```

This trick forces Claude to use that exact phrase string as the key in the output map, stopping it from messing with the key text. Plus, it cuts the cost slightly by turning what would've been pricey output tokens into cheaper input tokens.

LLMs can sometimes be influenced by the order in which items are presented. The textbook move might be to evaluate both orders and average the scores to cancel out that effect.

But `pun` deliberately sticks to the single, fixed order for these reasons:

- Swapping the order would mean prefilling with the benchmark word's string instead, losing the guarantee that the key for the other phrase is accurate.

- The goal isn't perfect, absolute scores; it's getting relative scores to rank phrases against each other.

- Running both orders means double the API calls.

> Does `pun` use [Claude's extended thinking](https://www.anthropic.com/news/visible-extended-thinking) feature for recognizability scoring?

Nah. Extended thinking kinda messes with things:

- One neat thing about Claude 3.7 Sonnet normally is that it gives lower scores to named entities and jargon compared to everyday words. Extended thinking seems to counteract that useful bias.

- You're stuck with a temperature of 1.0 when using extended thinking.

- Extended thinking is incompatible with prefilling the assistant's response. Trying to work around the prefill issue with a separate formatting step just adds more complexity and cost.

- It chews through more tokens because it has to generate all that thinking text first, bumping up the API costs.

- It also takes a bit longer to get a response since the model has to do the extra thinking step before giving the final scores.

> Does the system prompt include a sample answer?

Yep! The system prompt's got a sample answer to nudge the LLM into spitting out the format I'm after.

> How many phrases are included in the sample answer?

Two phrases are included: "the" and "to".

Two phrases are plenty to show off a map structure. One lone phrase might get misread as a request for only one result.

Determining the true percentage for less common words is difficult.

[They are the two most frequent word forms](https://www.wordfrequency.info/samples/wordFrequency.xlsx), per the Corpus of Contemporary American English, which aligns with the target demographic of Americans 10 and up. These two words can confidently be assigned a recognizability score of 100.0%.

> Does `pun` score each phrase multiple times and average the results?

No.

Running the same phrase a couple of times and averaging the results could potentially help smooth out any random noise.

But `pun` skips that. Doing multiple runs costs more API calls.

> Does `pun` use CSV for storing recognizability scores?

Nope. CSV is not ideal here because it lacks a proper key-value structure, which could lead to duplicate phrase entries.

> Does `pun` use JSON for storing recognizability scores?

Nah. `pun` uses EDN instead of JSON because:

- It's natively supported in Clojure, the backend language.

- It's a bit more concise than JSON since it ditches the colons.

> Are the precomputed recognizability scores committed to this repository?

Nah. These scores are generated data, not source code.

> Are the precomputed recognizability scores included in automated releases?

No way. The scoring process occasionally needs manual babysitting. API calls might fail, models might return garbage, or other random stuff can go wrong. Since it costs real money to run these LLM calls, I don't want to blindly retry in an automated pipeline. It's the kind of process I want to run manually, check the results, and then commit when I'm satisfied.

So instead, the scores are committed to the separate [`pun-data`](https://github.com/8ta4/pun-data) Git repository.

> Are the precomputed recognizability scores stored using Git LFS?

No.

- Git LFS adds that annoying central server dependency, which could be a single point of failure.

- The processed score files just don't hit the size where LFS becomes necessary.

> Are the recognizability scores guaranteed to be reproducible?

No way. Reproducibility is not guaranteed for the following reasons:

- Even with temperature set to zero, many LLMs still aren't deterministic.

- Remote model providers may update their models.

> How do you generate the normalized recognizability scores?

1.  Run `./download.sh` in your terminal to grab the Wiktionary data you need.

1.  Run `clj -M -m build vocabulary` to chew through the data and spit out the `vocabulary.txt` file.

1.  Copy your Anthropic API key string from their website to your clipboard.

1.  Run `mkdir -p ~/.config/pun && pbpaste > ~/.config/pun/key` to save the API key from your clipboard into the key file.

1.  Make sure your Anthropic account has credits.

1.  Run `clj -M -m build batches` to send off the scoring requests to the Anthropic Batch API.

1.  Run `clj -M -m build results` to download the result files for completed batches from the API.

1.  Run `clj -M -m build raw` to gather up scores from the downloaded result files.

1.  Run `clj -M -m build normalized` to parse the scores, normalize them using the benchmark, and save the final scores to `~/.cache/pun/normalized.edn`.

The `batches` command eats API credits. If your account runs goes negative, requests within the submitted batches might start failing. To get back on track, first run `clj -M -m build results` to save progress by downloading results. Then, top up your Anthropic account credits. After that, just run `clj -M -m build batches`; it will automatically identify remaining phrases and submit new batches only for those. it's smart enough to figure out what's left and only sends requests for those. Once that `batches` run finishes cleanly, proceed with the subsequent steps.

## Phonetic Similarity Analysis

> Can `pun` use homophones for substitution?

Yep! `pun` can totally use homophones for substitution. But it's not limited to perfect homophones. The tool works with words that share significant phonetic similarity too, which gives it way more flexibility when cranking out puns.

> What metric is used to calculate phonetic similarity?

`pun` uses normalized Levenshtein distance on International Phonetic Alphabet (IPA) representations. For substitution jokes to land, the audience has to recognize the original phrase being referenced. The tool converts words to their IPA representation and then calculates the Levenshtein distance between them to figure out how phonetically similar they are. This approach makes sure substitutions keep enough phonetic similarity to keep the puns identifiable.

> Does `pun` use [PanPhon](https://github.com/dmort27/panphon)'s phonological distance to calculate phonetic similarity?

Nope. Here's why:

- PanPhon's phonological distance says "pun" (/pʌn/) and "put" (/pʊt/) are more similar than "pun" (/pʌn/) and "gun" (/ɡʌn/), even though "pun" and "gun" rhyme and feel way more pun-worthy.

- Treating different phonological features equally or assigning weights to them is arbitrary.

- If a pun shows up or doesn't in your output, it's hard to figure out why PanPhon's phonological distance did that.

> What library does `pun` use for converting English text to IPA?

`pun` uses [`epitran`](https://github.com/dmort27/epitran). I tried [`eng_to_ipa`](https://github.com/mphilli/English-to-IPA), [`espeak-ng`](https://github.com/espeak-ng/espeak-ng), and [`g2p`](https://github.com/roedoejet/g2p) too, but they weren't accurate enough.

> Does `pun` use dictionaries for converting English text to IPA?

Nope. Using dictionaries alone runs into these problems:

- Dictionaries might miss words that are well-known but not super common, like "ungoogleable".

- Dictionary lookups struggle with how words change pronunciation in context, like how "the" sounds different before vowels versus consonants.

Sure, I could try to bolt dictionaries onto a conversion library for better accuracy, but that's a job for the IPA conversion library itself, not `pun`.

> Does `pun` convert English text to IPA on the fly?

No way. Transcribing the entire vocabulary to IPA would take ages.

> Is the precomputed IPA data stored in this repository?

Nah. The IPA data is generated data, not source code for the `pun` tool itself.

> Is the precomputed IPA data included in automated releases?

Nope. The precomputed IPA data is stored in the separate `pun-data` Git repository alongside the recognizability scores for consistency.

> Does `pun` treat diphthongs as single or double units in IPA representations when calculating Levenshtein distance?

`pun` treats diphthongs as single units. This makes the Levenshtein distance easier to understand since all vowels get treated equally.

> Does `pun` treat affricates as single or double units in IPA representations when calculating Levenshtein distance?

`pun` treats affricates as single units. This makes the Levenshtein distance easier to wrap your head around because every consonant is treated equally.

> What is the Levenshtein distance normalized by in `pun`?

The Levenshtein distance is normalized by the length of the IPA representation of the word being swapped out in the phrase, where the length is counted by treating each diphthong and affricate as just one unit.

Normalizing by the length of the word being replaced directly shows what percentage of the original word's sound is changed, measuring how much your substitution messes with the expected sound of the familiar phrase. This helps avoid substitutions that sound too different, which would just make your puns feel forced.

## Substitution

> Are the results sorted by Levenshtein distance?

Nope! Here's why:

- Phonetic similarity is just one piece of what makes a pun work.

- Creating a weighting between different factors would be arbitrary.

> Can `pun` generate ungrammatical puns?

Yep. That's because:

- Plenty of good puns break grammar rules on purpose.

- You can tweak the grammar yourself after getting the core wordplay idea, often needing to change more than just the substituted word to make it flow.

> Does `pun` filter substitutions where one word is identical to the other except for an "-ly" suffix at the end?

Yes. If one word is just the other plus "-ly" or vice-versa, that usually means swapping between an adjective and its adverb form, or the other way around. Thing is, that kind of substitution rarely works as clever wordplay and often just breaks the grammar. So, to knock out these predictable duds easily, `pun` runs a simple string check for that "-ly" pattern and filters the substitution if it matches.

> Does `pun` filter out substitutions where the replacement word is just an inflectional variant of the original word being replaced?

Yep. Swapping a word for another form sharing the exact same dictionary lemma almost never makes for a clever pun. So, `pun` checks if the potential replacement shares a lemma with the original word. If they match, that substitution gets filtered out.

> What library does `pun` use for lemmatization?

`pun` rolls with [LemmInflect](https://github.com/bjascob/LemmInflect).

LemmInflect's own benchmarks using the AGID dataset suggest it seems to [nail the correct lemma more often](https://github.com/bjascob/LemmInflect/blob/b7699808106a4ce843fc7f0e8e5d87fcb84cc636/README.md?plain=1#L32-L47) compared to the other guys.

[CoreNLP](https://github.com/stanfordnlp/CoreNLP) exhibits a tendency to spit the same word back instead of the actual lemma when given comparative adjectives.

> Can `pun` generate multiple puns that differ only in the grammatical form of the substituted word?

Yep! These aren't filtered out because:

- Different forms might land better depending on where you're using the pun.

- Forcing you to mentally transform words wastes your brain power.
