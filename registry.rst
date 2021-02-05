===============================================
Registry for Environments, Policies, and Agents
===============================================

See this `pull request
<https://github.com/agentos-project/design_docs/pull/1>`_ for additional
discussion and revision history.

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
  learning strategies, and environments.

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
``environment``.  Now, let's install the ``2048`` environment that models
the in-browser game `2048 <https://en.wikipedia.org/wiki/2048_(video_game)>`_::

  agentos install environment 2048

The above command not only installs the 2048 environment into your agent
directory, but it also updates our agent directory's ``components.ini`` file to
record the specifics of the components we've installed.

Our agent will now run against the 2048 game environment. However, our agent
still lacks the ability to learn.  Let's fix that by installing a `policy
component that learns via the SARSA algorithm
<https://en.wikipedia.org/wiki/State%E2%80%93action%E2%80%93reward%E2%80%93state%E2%80%93action>`_
into our agent::

  agentos install policy sarsa

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

  agentos install policy dqn

Because you are installing a second policy, ACS will ask you which policy you'd
like to make default.  All installed components are always programmatically
accessible, but ACS provides shortcuts for accessing default components.

Go ahead and select ``dqn`` as the default policy.  Again, ``components.ini``
will be updated to reflect that we are now using a DQN-based learning algorithm
instead of a SARSA-based learning algorithm.

Now when you run your agent again (``agentos run``) your agent will be using
Q-learning with a deep Q network to learn 2048.


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

The ``acs`` module automatically loads default components under shortcuts such
as ``acs.policy`` and ``acs.environment``.  If you have more than one component
installed for a particular role (e.g. two complementary environments) then you
can access each component via their name in the ``acs`` module::

  acs.environment.2048.step()
  ...
  acs.environment.cartpole.step()


MVP
===

* ACS will be able to access a centralized registry of policies and
  environments

  * V0 target: the list will be a yaml file stored in the AgentOS repository

* Each registry entry will be structured as follows::

    component_name:
      type: [policy | environment | algorithm]
      description: [component description]
      releases:
        - name: [version_1_name]
          hash: [version_1_hash]
          github_url: [url of version 1 repo]
          class_name: [fully qualified class name of version 1]
          requirements_path: [path to version 1 requirements file]

        - name: [version_2_name]
          hash: [version_2_hash]
          github_url: [url of version 2 repo]
          class_name: [fully qualified class name of version 2]
          requirements_path: [path to version 2 requirements file]

  for example::

    2048:
      type: environment
      description: "An environment that simulates the 2048 game"
      releases:
        - name: 1.0.0
          hash: aeb938f
          github_url: https://github.com/example-proj/example-repo
          class_name: main.2048
          requirements_path: requirements.txt

        - name: 1.1.0
          hash: 3939aa1
          github_url: https://github.com/example-proj/example-repo
          class_name: main.2048
          requirements_path: requirements.txt

* Each component will be a (v0: Python) project stored in a Github repo.

* ACS will have an ``search`` method that will list all components in the
  registry matching the search query.

* ACS will have an ``install`` method that will:

  * Find the components location based on its registry entry

  * Ask if you'd like to install the component as the default in cases where
    there are multiple installed components of the same type.

  * Clone the component's Github repo

  * Update the agent directory's ``components.ini`` to include the component in
    its default configuration

  * Register the component locally so that it is accessible via the ``acs``
    module

  * Add a line to the agent directory's requirements file that links to the
    component's requirements file (e.g. a line of the form
    `-r component/repo/path/requirements.txt`.).

* ACS will have an ``uninstall`` method that will remove the component from the
  agent directory (including any links to the component's requirements).

* Components can be programmatically accessed from the ``acs`` module

* Developers have an easy way to register their local custom components with
  ``acs`` so it can be accessed via the ``acs`` module in other parts of their
  agent.

* The minimal agent (``agentos init``) will be ACS aware and incorporate
  basic components with minimal required edits


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

**Q:** Can only 1 component of each type be installed in an agent at a time?

**A:** We should allow multiple components of a single type.  Perhaps
``components.ini`` defines the default for each type and that default is
accessible programmatically via shortcuts like ``acs.policy`` and
``acs.environment``.

In an agent where you have, for example, two policies installed (e.g.
``random`` and ``dqn``) the default (as determined by ``components.ini``) will
be accessible at ``acs.policy``, but both will always be accessible at
``acs.policy.random`` and ``acs.policy.dqn`` respectively.

**Q:** How does AgentOS locate the main code of the component within the Github
repo? Must all components have a well known entry point (e.g., a file called
main.py)?

**A:** The ACS registry entry for each version of a component contains
sufficient information to discover the entry point of the component and its
requirements.

We may eventually:

* Require a component's repo to store additional metadata (perhaps in a
  top level ``agentos.ini`` file) that ACS tooling can ingest to alleviate
  concerns about mismatches between registry info and repo info (e.g. a
  component's version is different in the registry and in the repo).

* Require all components to be proper Python packages so we can reuse Python's
  ``setup.py`` tooling.


**Q:** Will we update the code generated by ``agentos init`` so that it will
use the ACS module?

**A:** Yes, if we decide that ACS is a worthwhile pursuit, then I think we
should make sure it's on-by-default for all agents.  I imagine we could default
to some very basic components for minimal agents that are included
out-of-the-box in AgentOS (e.g. random action policy, a basic corridor
environment).

**Q:** Do we want to design the API so that using a component from the registry
looks exactly (or nearly) the same as using a hand-built component.  Basically,
should we recommend using the same sort of composition for both composing an
agent from an environment, policy, and algorithm built from scratch and
composing an agent entirely from pre-built components in the registry?

**A:**  Yes, I think nudging users toward consistency would be good.  I think
that means component specifications and APIs that are well documented and
tooling that makes it valuable to build to those specs.

Ultimately, if someone wants to give their custom environment a nonstandard
``proceed_one_step_in_time()`` function instead of a ``step()`` function, we
shouldn't try to stop them.  But we should instead strive to make it high-value
to standardize because you can use a bunch of great tools out-of-the-box on
your component programmed to the spec.

Diving down closer to the code, I think we need to provide an easy way to, for
example, register your custom environment so that you can access it via
``acs.environment`` in the rest of your code, and encourage exposing and
interacting with your custom components in this way.


**Q:** How does this relate to OpenAI's ``gym.envs.registry``, if at all?

**A:** The idea of having an ``acs`` module that you can import in your Python
code is inspired by the ``gym.envs.registry``.  The ``acs`` module dynamically
loads in the available components much like gym's registry.

One rationale I found for OpenAI's environment registry is
[here](https://github.com/openai/gym/blob/master/gym/envs/registration.py#L76)
and essentially amounts to versioning an environment.  We solve this problem by
requiring a git hash for every "released" version of a component.

**Q:** How does this relate to how AgentOS uses MLflow for Agent Directories.
Should we merge the two concepts? Or at least unify them? Maybe get rid of the
dependency on MLflow?

**A:**  I think MLflow will be useful and should remain a dependency; one will
still have to perform various runs with an agent (e.g. to tune hyperparameters)
and MLflow's tracking and visualization should be useful for that.

In fact, one could think of the components themselves as hyperparameters to the
agent, and some sort of deeper integration with MLflow would probably be
valuable ("On the first run I used a Deep Q Network component with 128 nodes to
represent my Q function, while on my second run I used a table component with
512K entries").

TODO and open questions
=======================

* How to handle component dependencies (Both package and component-level)?

  * `StackOverflow on conditional requirements <https://stackoverflow.com/a/29222444>`_
  * How to fail gracefully if there are incompatible requirements
  * Perhaps use separate processes to isolate run environments
  * Can we just use the Python package system and pip directly?

* What are the key components that we want to expose in our registry?
  Candidates: Agents, Policies, Environments, Learning Strategies, Memory
  Stores, Models.

See Also
========

* Pull requests:

  * `design_docs #1: AgentOS registry <https://github.com/agentos-project/design_docs/pull/1>`_
  * `design_docs #2: Avoid merging requirements on component install <https://github.com/agentos-project/design_docs/pull/2>`_
  
* Document version history:

  * `v1 <https://github.com/agentos-project/design_docs/blob/36791f4ef1cf408c19cf13042bb7cc6b72cb6030/registry.rst>`_
  * `v2 <https://github.com/agentos-project/design_docs/blob/020a70a5e538b58e5e0ff269f44a7f206a7b132e/registry.rst>`_
  * `v3 <https://github.com/agentos-project/design_docs/blob/e32ff7a96eab3486a3c8bb65c1ca1df280e20434/registry.rst>`_
  
* `AgentOS Issue 68: Registery for Envs, Policies, and Agents <https://github.com/agentos-project/agentos/issues/68>`_
* `PEP 301 -- Package Index and Metadata for Distutils <https://www.python.org/dev/peps/pep-0301/>`_
* `PEP 243 -- Module Repository Upload Mechanism <https://www.python.org/dev/peps/pep-0243/>`_
