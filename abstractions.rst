=========================
AgentOS Core Abstractions
=========================

Current Version: v1

See `Revision History`_ for additional discussion.


Abstract
========

This document proposes the core abstractions of the AgentOS system.  These
abstractions are:

* **Environment** - A representation of the Markov decision process (MDP) in
  which the agent operates.

* **Policy** - Encapsulates the agent's decision making process.  The policy
  chooses an action given a set of observations.

* **Trainer** - Encapsulates how a policy changes over time as an agent gains
  experience.

* **Agent** - The glue that binds the policy and trainer.  The agent is what
  performs actions within the environment with an aim to maximize reward.

Rationale
=========

AgentOS aims to define a clean set of abstractions that will be familiar to
researchers in the reinforcement learning (RL) space while also being
approachable to technically educated generalists like software engineers.  The
proposed set of abstractions hews closely to the standard academic
presentation of RL while allowing for composition and reuse at the software
level.

Core Abstractions
=================

This section will present sample code for each of the core abstractions as
well as discuss implications and future directions.

Environment
-----------

An environment will conform to `OpenAI's gym <https://gym.openai.com/>` API.  Core methods
are:

* ``step(action) -> (observation, done, reward, info)``: This takes one action
  within the environment and transitions to a new state.

* ``reset() -> observation``: This resets the environment to an initial state.

Here is an example environment that represents a 1-dimensional corridor that
the agent must learn to walk down::

    import gym


    class Corridor(gym.Env):

        def __init__(self):
            self.length = 10
            self.action_space = gym.spaces.Discrete(2)
            self.observation_space = gym.spaces.Discrete(self.length + 1)
            self.reset()

        def step(self, action):
            assert action in [0, 1]
            if action == 0:
                self.position = max(self.position - 1, 0)
            else:
                self.position = min(self.position + 1, self.length)
            return (self.position, -1, self.done, {})

        def done(self):
            return self.position >= self.length

        def reset(self):
            self.position = 0
            return self.position

Policy
------

A policy encapsulates an agent's decision making process.  A policy will often
be backed by some sort of stateful storage (e.g. a table or a serialized
neural net).  Eventually, AgentOS will provide ability to easily share
policies (including their state) between agents.

A Policy must provide the following two methods:

* ``decide(obs) -> action``: Takes the agent's current observation and returns
  the next action the agent should take.

* ``reset() -> None``: Dumps any stateful part of the policy so that the agent
  is can restart learning from scratch.


An example policy class might look like the following::

    import agentos

    class DeepQNetwork(agentos.Policy):
        def decide(self, obs):
            action_probabilites = self.nn(obs)
            return random.weighted_choose(action_probabilities)

        def reset():
            self.nn = create_new_neural_net()

Trainer
-------

A trainer describes how to incorporate experiences into the agent's policy so
that the agent can learn how to increase reward in subsequent runs through the
environment.  Trainers are often closely coupled to the policies that they
modify.

Trainers provide the following method: [#train-method]_

* ``train(policy, **kwargs) -> policy``: Mutates the policy to reflect the
  agent's learning.

An example trainer class for the Deep Q network above might look like the
following::

    import agentos

    class ReplayBufferSGD(agentos.Trainer):
        def train(policy, environment_class):
            rollouts = agentos.do_rollouts(policy, environment_class)
            advantaged_rollouts = calculate_advantage(rollouts)
            policy.nn.train(advantaged_rollouts)


Agent
-----

An agent is the "glue" that binds the trainer and the policy as well as the
entity that performs actions within the environment.  An agent provides the
following methods:

* ``train() -> None``: This is called to improve the agent's policy via
  practice within the environment.


* ``advance() -> None``: This is called to cause the agent to act within its
  environment based on its current policy.

An example agent class might look like the following::

    import agentos

    class MyAgent(agentos.Agent):
        def train(self):
            self.trainer.train(self.policy)

        def advance(self):
            next_action = self.policy.decide(self.obs)
            self.obs, done, reward, info  = self.environment.step(next_action)


Note that ``train()`` will be a no-op for some agents as the their learning
might take place while the agent is advancing through its environment.  To
this end, we propose two common subclasses of the agent:

* ``OnlineAgent``: This agent learns while it advances through its
  environment.  Thus ``train()`` will often be a no-op as the policy will be
  trained each time ``advance()`` gets called.

* ``BatchAgent``: This agent learns in an "offline" manner.  It will either
  record its various trajectories through the environment or practice in an
  isolated instantiation of its environment.  This agent's policy will only be
  trained when ``train()`` is called.


Agent Definition File
---------------------

Every agent will define an ``agent.ini`` file that describes the specific
components of the agent.  A standard agent directory structure might look
something like the following::

    my_agent/
      - main.py
      - trainer.py
      - environment.py
      - policy.py
      - policy/
        - serialized_nn.out
      - agent.ini

Combining our example code from above, our agent's ``agent.ini`` file will
look like the following::

      [Agent]
      class = main.MyAgent

      [Policy] # self.policy
      class = policy.DeepQNetwork
      architecture = [10,100,100]
      storage = ./policy/

      [Environment] # self.environment
      class = environment.Corridor

      [Trainer] # self.trainer
      class = trainer.ReplayBufferSGD
      buffer_size = 10000
      batch_size = 100

Note that the ``agent.ini`` contains both the location of primary components
of the agent as well as various configuration variables and hyper-parameters.
This file will be managed by the AgentOS Component System (ACS) to allow for
easy composition and reuse of AgentOS components.


Demo
====

AgentOS will provide both command line and programmatic access to agents.

A common use case will be using the command-line to train and run an agent as
follows::


    agentos train /path/to/agent.ini 1000 # Train the agent's policy over 1000 rollouts
    agentos run /path/to/agent.ini  # Run our agent to measure performance
    agentos train /path/to/agent.ini  1000 # Train our agent on another 1000 rollouts
    agentos run /path/to/agent.ini   # Measure performance again
    agentos reset /path/to/agent.ini  # Resets the agent's policy; forget all learning


The AgentOS CLI provides several ways to run an agent.  You can run using the
components specified in the ``agent.ini`` in the current directory as
follows::

    agentos run

Alternatively, you can specify the ``agent.ini`` file to use as follows::

    agentos run -f ../../agent.ini

Finally, you can specify all the components of an agent individually as
follows::

    agentos run -e myenv.Env -p mypolicy.Policy -a main.MyAgent -t trainer.ReplayBufferSGD


Additionally, AgentOS provides methods for running agents programmatically
either using an ``agent.ini`` file::

    agentos.run_agent_file('path/to/file/agent.ini')

Or by specifying each component as a keyword argument::

    agentos.run_agent(
        agent=MyAgent,
        environment=MyEnv,
        policy=MyPolicy,
        trainer=MyTrainer
    )

See Also
========

Revision History
================

* Pull requests:

  * `design_docs #3: AgentOS Core Abstractions <https://github.com/agentos-project/design_docs/pull/3>`_

* Document version history:

  * `v1 <https://github.com/agentos-project/design_docs/blob/36791f4ef1cf408c19cf13042bb7cc6b72cb6030/registry.rst>`_


Footnotes
=========

.. [#train-method]: This method signature is probably over-simplified.  A
trainer might need access to the environment (or environment class), the
policy itself, recent observations, etc













