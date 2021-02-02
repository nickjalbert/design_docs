===============================================
Registry for Environments, Policies, and Agents
===============================================


Abstract
========

This document proposes the **AgentOS component system (ACS)**.  ACS allows for
easy composition and reuse of key top-level AgentOS components (e.g.
environments and policies) much like ``pip`` does for Python and ``APT`` does
for Debian Linux.

Rationale
=========

A primary goal of AgentOS is to provide a clean set of abstractions that will
accelerate the research and development of agent systems.  These abstractions
provide a straightforward way to swap in existing components to create new
agents with different behaviors and learning strategies; as long as each
component respects its interface, the specifics of its implementation should
not matter for composition.

The ACS enables this vision by:

* Maintaining a centralized registry of agent components including policies,
  learning strategies, and environments. [#abstractions-todo]_

* Providing a versioning system for these components

* Providing a command-line interface to install these components

* Providing a module for composing components within agents


Demo
====

This section illustrates the usage of ACS by way of example commands and code.

Interaction with the registry
-----------------------------

First, ensure AgentOS is installed in your environment::

  pip install agentos

Then create a new agent::

  mkdir my_agent
  cd my_agent
  agentos init

This generates a minimal agent in your ``my_agent`` directory.  The minimal
agent is not particularly interesting though, so let's flesh it out.

Let's take a look at the environments available to our agent in the ACS::

  agentos search environment

The above command returns the listings for all components of type
``environment``.  Now, let's install the ``env-2048`` environment that models
the in-browser game `2048 <https://en.wikipedia.org/wiki/2048_(video_game)>`_::

  agentos install env-2048

The above command [#cmd-todo]_ not only installs the 2048 environment into your agent
directory, but it also updates our agent directory's ``components.ini`` file to
record the specifics of the components we've installed.

Our agent will now run [#wiring-todo]_ against the 2048 game environment.

Our agent still lacks the ability to learn.  Let's fix that by installing a
`policy that learns via the SARSA algorithm
<https://en.wikipedia.org/wiki/State%E2%80%93action%E2%80%93reward%E2%80%93state%E2%80%93action>`_
into our agent::

  agentos install policy-sarsa

After the install completes, our ``components.ini`` will again be updated to
record the fact that our agent will be using SARSA to learn how to play 2048.


We can now run the agent as follows::

  agentos run

As you let your agent run, you'll get periodic updates on its performance
improvement as it gains more experience playing 2048 and learns via the SARSA
algorithm.

Now suppose we become convinced that a `Deep Q-Learning network
<https://en.wikipedia.org/wiki/Q-learning>`_ would be more amenable to learning
2048.  Switching our learning strategy and policy is as easy as running::

  agentos install policy-dqn

Again, ``components.ini`` will be updated to reflect that we are now using a
DQN-based learning algorithm instead of a SARSA-based learning algorithm.

Using components within code
---------------------------

Let's dig into our minimal agent to see how we access our components programmatically::

    from agentos import Agent
    from agentos import acs
    
    class SimpleAgent(Agent):
        def advance(self):
            acs.policy.train(acs.env)
            done = False
            obs = acs.environment.reset()
            next_action = acs.policy.choose(obs)
            obs, reward, done, _ = acs.environment.step(next_action)
            return done

The ``acs`` module automatically loads the top-level components under shortcuts
such as ``acs.policy`` and ``acs.environment``.  If you have more than one
component installed for a particular role (e.g. two complementary environments)
then you can access it via the component name in the ``acs`` module::

  acs.env-2048.step()
  ...
  acs.env-cartpole.step()


MVP
===

* ACS will be able to access a centralized registry of policies and
  environments (V0: the list will be a yaml file stored in the agentos repo).

* Each registry entry will be structured as follows::

  component_name:
    type: [policy | environment | algorithm]
    description: [component description]
    source: [link_to_github_repo]
    releases:
      hash1: version_1_name
      hash2: version_2_name

for example::

  env-2048:
    type: environment
    description: "An environment that simulates the 2048 game"
    source: https://github.com/agentos-project/env-2048/
    releases:
        0fdea27: 1.0.0
        33379a8: 1.1.0

* Each component will be a (v0: Python) project stored in a github repo that
  will minimally contain the following files:

  * A ``definition.py`` that will contain the description of that component's
    ``components.ini`` entry.

  * A ``requirements.txt`` that will contain the project requirements

* ACS will have an ``search`` method that will:

  * List all components in the registry matching the search query.

* ACS will have an ``install`` method that will:

  * Find the components location based on its registry entry

  * Download the component from github

  * Merge the component requirements into the existing agent directory's
    requirements (TODO: and also install?)

  * Update the agent directory's ``components.ini`` to include the component in
    its default configuration.

* Components can be programatically accessed from the ``acs`` module

* The minimal agent (``agentos init``) will be ACS aware and behave as
  expected



Long Term Plans
===============

* A simple way for component authors to submit components to the registry via
  command-line and web interface.


FAQ
===

**Q:** My [complex component] has a number of hyperparameters that need to be
tuned based on the particulars of the environment and the agent.  How do I do
this?

**A:** Each component maintains exposes a configuration in its ``components.ini``
entry. This allows for both manual tweaking of hyperparameters as well as
programmatic exploration and tuning.

**Q:** How can I reuse a model from a previous run?

**A:** Models themselves are exposed as top-level components.  ``agentos run``
has tooling that allows you to dynamically specify when and how to reuse
existing models.



TODO and open questions
=======================

* How to handle component dependencies (Both package and component-level)?

* What are the key components that we want to expose in our registry?
  Candidates: Agents, Policies, Environments, Learning Strategies, Memory
  Stores, Models.

See Also
========
* `AgentOS Issue 68: Registery for Envs, Policies, and Agents <https://github.com/agentos-project/agentos/issues/68>`_
* `PEP 301 -- Package Index and Metadata for Distutils <https://www.python.org/dev/peps/pep-0301/>`_
* `PEP 243 -- Module Repository Upload Mechanism <https://www.python.org/dev/peps/pep-0243/>`_


Footnotes
=========

.. [#abstractions-todo] I'm not sure if these are the right types of
                        abstractions, but I suspect we'll get a better
                        handle on this as we build.

.. [#cmd-todo] Does this make sense as a subcommand for ``agentos`` or as its
               own command (e.g. ``acs install ...``)....

.. [#wiring-todo] Should there be manual wiring here to make our agent play in
                  the env, or should we assume that this is the environment
                  because you've already called install?



