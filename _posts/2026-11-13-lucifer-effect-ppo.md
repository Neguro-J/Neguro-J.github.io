# A Curious Question: Can We Explore the Lucifer Effect with PPO?

Sometimes a project starts with a paper, a research goal, or a practical problem. This one started with a random question:

**What would happen if I took an idea from social psychology and tried to turn it into a small reinforcement-learning experiment?**

I had been reading about the *Lucifer Effect*, a term associated with Philip Zimbardo's discussion of how situations, roles, and social pressures can shape harmful behavior.

The Stanford Prison Experiment often mentioned alongside it is highly controversial and has been criticized on methodological grounds. So I am not treating it as a reliable blueprint for human psychology.

What caught my attention was a more general idea: **sometimes the rules and incentives of a situation matter as much as the choices available inside it.**

Could I make a computational toy model to play with that question?

That's how **Lucifer-RL** began. No grand theory or intended application—just curiosity and some PPO experiments.

## A very small world

The setup is intentionally simple: four simulated agents, each choosing between two actions at every time step.

- **Safe action (0):** avoid the shortcut.
- **Shortcut (1):** potentially gain an advantage, but pay a modeled cost.

Each agent observes a summary of previous group behavior, a small internal adaptation variable, and the episode's progress.

The agents share a neural-network policy trained with *Proximal Policy Optimization* (PPO).

The environment includes a few parameters I can adjust: a reward for following what the group did previously, a cost for taking the shortcut, and an adaptation mechanism that can make repeated shortcuts less costly.

These are artificial modeling choices, **not claims about human motives or morality**.

## What PPO actually does here

PPO doesn't understand the labels "safe" or "harmful." It learns which actions tend to produce higher cumulative reward.

In this experiment, the process is relatively straightforward:

1. The agents observe the environment and previous group behavior.
2. Each agent selects an action.
3. The environment calculates rewards and updates its state.
4. PPO uses the collected experiences to update the shared policy.
5. The process repeats.

If a policy repeatedly selects the shortcut, it isn't because it has become "bad." It is responding to the environment and to the way it was trained.

And that is precisely what made the experiment interesting to me.

## An unexpected observation

During a few exploratory runs, changing the adaptation parameter dramatically changed the actions learned by PPO.

Without adaptation, five independent training runs mostly selected the safe action. With stronger adaptation, five runs mostly selected the shortcut.

The curious case was an intermediate adaptation value of **0.02**.

Under identical settings but different random seeds, four runs learned mostly safe behavior, while one learned to choose the shortcut almost every time.

Even more interestingly, in the tested 30-step episodes, the shortcut-heavy policy obtained a lower reward than a simple strategy that always selected the safe action.

So the learned policy was not necessarily the best available strategy.

Of course, this doesn't mean we have discovered a computational equivalent of the Lucifer Effect.

The environment is synthetic, the behaviors are simplified, and the outcomes depend on assumptions such as episode duration, reward discounting, and cost adaptation.

Still, watching five training runs develop such different behaviors under the same rules was quite interesting.

## A few questions it left me with

The experiment made me wonder about a few things.

How much of a learned decision comes from an individual incentive, and how much from the group's previous actions?

Could a small feedback mechanism make an early preference persist?

And why might two PPO training runs, given the same rules, end up behaving so differently?

I don't have definitive answers from these initial experiments.

For now, that is the interesting part: **a simple model can produce unexpected behavior while still leaving plenty of room for a better explanation.**

Lucifer-RL is just a curiosity-driven exercise in thinking about incentives, feedback, and reinforcement learning.

I thought it was worth sharing because sometimes a small question is enough to make a familiar algorithm feel new again.

### References

- Schulman et al. (2017). *Proximal Policy Optimization Algorithms.*
- Leibo et al. (2017). *Multi-agent Reinforcement Learning in Sequential Social Dilemmas.*
