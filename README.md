# SDE Learning Hub

This repo is the map. The courses live in their own repositories, pinned here as submodules, so each track can be shared with a different person.

Start the AI path in [`tracks/ai`](tracks/ai) (`Data-science`): Python, math, machine learning, deep learning, NLP, LLMs, MLOps, and data engineering. Interview preparation for programming, DSA, and system design lives in [`tracks/dsa-and-system-design`](tracks/dsa-and-system-design).

## Tracks

| Track | Path | Repo | Who it is for |
| --- | --- | --- | --- |
| AI, from scratch | `tracks/ai` | [Data-science](https://github.com/rakeshkandhi/Data-science) | The AI engineer path. Python fundamentals are module 01 inside this repo. |
| DSA, system design, LLD, CS | `tracks/dsa-and-system-design` | [algo-system-design](https://github.com/rakeshkandhi/algo-system-design) | Interview prep. Long-form system design essays and the worked DSA pattern notes live here. |
| Web programming | `tracks/programming` | [Web-development](https://github.com/rakeshkandhi/Web-development) | JavaScript, TypeScript, Node, and React. |
| Git workflow | `tracks/git` | [git_branching_strategy](https://github.com/rakeshkandhi/git_branching_strategy) | Branching strategies. Public. |

Suggested order for the AI journey: `tracks/ai` module 01, then math, machine learning, and deep learning as those modules fill in. Run `tracks/dsa-and-system-design` beside it when you want interview reps. Use `tracks/programming` when the work is web, and `tracks/git` before you collaborate on any of the above.

Reading companions for the AI track. These stay upstream. They are not submodules.

- [pytorch-deep-learning](https://github.com/mrdbourke/pytorch-deep-learning) for module 04
- [Hands-On Large Language Models](https://github.com/HandsOnLLM/Hands-On-Large-Language-Models) for module 06
- [AI Agents for Beginners](https://github.com/microsoft/ai-agents-for-beginners) for agents inside module 06

## Clone

Someone with access to every track:

```bash
git clone --recurse-submodules git@github.com:rakeshkandhi/SDE_LEARNING.git
```

Someone with access to one track:

```bash
git clone git@github.com:rakeshkandhi/SDE_LEARNING.git
cd SDE_LEARNING
git submodule update --init tracks/ai
```

A private submodule stays empty when GitHub refuses the clone. The hub still clones. Init only the tracks that person can read.

This repo records a commit for each track. To move a pin forward:

```bash
git submodule update --remote tracks/ai
git add tracks/ai
git commit -m "Advance the AI track pin"
```

## Access

Granting access to this hub lets someone read the map. It does not let them read a private track. Add them to the track repo as well.

```bash
gh repo add-collaborator rakeshkandhi/SDE_LEARNING FRIEND --permission pull
gh repo add-collaborator rakeshkandhi/Data-science FRIEND --permission pull
```

Use `push` instead of `pull` when they should be able to commit. `tracks/git` is public, so it needs no invite.

When the friend list grows, a GitHub organization with one team per track is the cleaner version of the same split. The submodule layout does not have to change.

## Where the old notes went

The essays, slide decks, and DSA pattern guides that used to sit in this repo now live in `tracks/dsa-and-system-design`:

- `system-design/longform/`
- `system-design/longform/decks/`
- `dsa/worked-examples/`

Git history on this repo still has the original files.

## Kept out of the hub

These repos cover ground a track already owns. They stay on GitHub as historical projects and are not pinned here.

| Already covered by | Repos left as-is |
| --- | --- |
| `tracks/ai` | `Iris-Classification`, `SimpleLinearRegression`, `Churn_for_Bank_Customers`, `Healthcare_cardiovascular`, `Movies_data`, `House_Loan_Data_Analysis`, `mlproject`, `Iphone`, `E-Commerce`, `MIT-ADP`, `Predicting_surgery_outcome`, `Chicken-Disease-Classification`, `Text-Summarization`, `Rice_Variety_classification` |
| `tracks/programming` | `rock-paper-scissors`, `Netflix-clone`, `Tribute_page`, `Image-gallery`, `Base64toIMG`, `Registration_form`, `Skibble_Assignments`, `to-do-app`, `todos-client`, `full-stack-school` |
| Nothing in the learning path | `Portfolio`, `react-portfolio`, `admin-portfolio-rakeshkandhi` are three portfolio apps. `azure-ai-engineer-associate` is an empty repo. Product repos (`medha`, `SuperCmd`, and the ecommerce apps) stay independent. |

Forks of other people's projects (`zed`, `warp`, `mempalace`, `claude-cookbooks`, and the rest) are not tracks.
