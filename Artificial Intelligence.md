An **agent** is anything that can be viewed as perceiving its **environment** through **sensors** and acting upon that environment through **actuators**. We use the term **percept** to refer to the agent’s perceptual inputs at any given instant. An agent’s **percept sequence** is the complete history of everything the agent has ever perceived.

Mathematically, an agent’s behavior is described by the **agent function** that maps any given percept sequence to an action.

A **rational agent** is one that does the right thing, i.e., every entry in the table for the agent function is filled out correctly. When an agent is plunked down in an environment, it generates a sequence of actions according to the percepts it receives. This sequence of actions causes the environment to go through a sequence of states. If the sequence is desirable, then the agent has performed well. This notion of desirability is captured by a **performance measure** that evaluates any given sequence of environment states.

Notice that we said _environment_ states, not _agent_ states. If we define success in terms of agent’s opinion of its own performance, an agent could achieve perfect rationality simply by deluding itself that its performance was perfect.

_As_ a general rule, it is better to design performance measures according to what one actually wants in the environment, rather than according to how one thinks the agent should behave.

For each possible percept sequence, a rational agent should select an action that is expected to maximise its performance measure, given the evidence provided by the percept sequence and whatever built-in knowledge the agent has.

We need to be careful to distinguish between rationality and **omniscience**. An omniscient agent knows the _actual_ outcome of its actions and can act accordingly; but omniscience is impossible in reality. Rationality maximizes _expected_ performance, while perfection maximizes _actual_ performance.

Our definition of rationality does not require omniscience, then, because the rational choice depends only on the percept sequence _to date_. We must also ensure that we haven’t inadvertently allowed the agent to engage in decidedly underintelligent activities. For example, if an agent does not look both ways before crossing a busy road, then its percept sequence will not tell it that there is a large truck approaching at high speed. Does our definition of rationality say that it’s now OK to cross the road? Far from it! 

First, it would not be rational to cross the road given this uninformative percept sequence: the risk of accident from crossing without looking is too great. Second, a rational agent should choose the “looking” action before stepping into the street, because looking helps maximize the expected performance. Doing actions _in order to modify future percepts_—sometimes called **information gathering**, is an important part of rationality. A second example of information gathering is provided by the **exploration** that must be undertaken by a vacuum-cleaning agent in an initially unknown environment.

To the extent that an agent relies on the prior knowledge of its designer rather than on its own percepts, we say that the agent lacks **autonomy**. A rational agent should be autonomous, it should learn what it can to compensate for partial or incorrect prior knowledge.


In our discussion of the rationality, we have to specify the performance measure, the environment, and the agent’s actuators and sensors. We group all these under the heading of the **task environment**. We call this the **PEAS** (**P**erformance, **E**nvironment, **A**ctuators, **S**ensors) description.

**Fully observable** vs. **partially observable**: If an agent’s sensors give it access to the complete state of the environment at each point in time, then we say that the task environment is fully observable. A task environment is effectively fully observable if the sensors detect all aspects that are _relevant_ to the choice of action; relevance, in turn, depends on the performance measure. 

**Deterministic** vs. **stochastic**: If the next state of the environment is completely determined by the current state and the action executed by the agent, then we say the environment is deterministic, otherwise, it is stochastic. In principle, an agent need not worry about uncertainty in a fully observable, deterministic environment. (In our definition, we ignore uncertainty that arises purely from the actions of other agents in a multi-agent environment; thus, a game can be deterministic even though each agent may be unable to predict the actions of the others.) If the environment is partially observable, however, then it could _appear_ to be stochastic.


**Static** vs. **dynamic**: If the environment can change while an agent is deliberating, then we say the environment is dynamic for that agent; otherwise, it is static.

The job of AI is to design an **agent program** that implements the agent function. the mapping from percepts to actions. We assume this program will run on some sort of computing device with physical sensors and actuators, we call this the **architecture**.
Agent = _architecture_ + _program_.

**Agent programs**: The agent programs take the current percept as input from the sensors and return an action to the actuators. Notice the difference between the agent program, which takes the current percept as input, and the agent function, which takes the entire percept history.

Four basic kinds of agent programs that embody the principles underlying almost all intelligent systems:

- Simple reflex agents
- Model-based reflex agents
- Goal-based agents
- Utility-based agents

**Simple reflex agents**
These agents select actions on the basis of the _current_ percept, ignoring the rest of the percept history.