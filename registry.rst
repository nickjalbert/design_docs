===============================================
Registry for Environments, Policies, and Agents
===============================================


Abstract
========

This document proposes the **AgentOS component system (ACS)**.  ACS allows for
easy composition and reuse of key top-level AgentOS components (Environments and
Policies) much like ``pip`` does for Python
and ``APT`` does for Debian Linux.

Rationale
=========

A primary goal of AgentOS is to provide a clean set of abstractions that will
accelerate the research and development of agent systems.  These abstractions
provide a straightforward way to swap in existing components to create new
agents with different behaviors and learning strategies; as long as each
component respects its interface, the specifics of its implementation should
not matter for composition.

The ACS enables this vision by:

* Maintaining a centralized registry of Agents, Policies, Learning Strategies,
  and Environments.

* Providing a command-line interface to install these components

* Providing a versioning system for these components


Demo
====

First, ensure AgentOS is installed in your environment::

  pip install agentos

Then create a new agent::

  mkdir my_agent
  cd my_agent
  agentos init

This generates a minimal agent in your ``my_agent`` directory.  The minimal
agent is not particularly interesting though, so let's flesh it out::

  agentos install env-2048

  TODO: Does this make sense as a subcommand for ``agentos`` or as its own
  command (e.g. ``acl install ...``)....

The above commands installs an environment that models the in-browser game
`2048 <https://en.wikipedia.org/wiki/2048_(video_game)>`_. The command also
creates a ``components.ini`` file in our agent directory that records the
specifics of the environment we've installed.

Our agent will now run against the environment.
    
    TODO: Should there be manual wiring here to make our agent play in the env,
    or should we assume that this is the environment because you've already
    called install.

Our agent still lacks the ability to learn.  Let's fix that by installing a
`SARSA policy
<https://en.wikipedia.org/wiki/State%E2%80%93action%E2%80%93reward%E2%80%93state%E2%80%93action>`_
in our agent::

  agentos install policy-sarsa

After the install completes, our agent will now use SARSA to learn how to play
2048.  Run the agent as follows::

  agentos run

Now suppose we become convinced that a `Deep Q-Learning network
<https://en.wikipedia.org/wiki/Q-learning>`_ would be more amenable to 2048.
Switching our policy and learning strategy is as easy as::

  agentos install policy-dqn

Various hyperparameters and configuration variables can be updated and modified
in ``configuration.ini``.


MVP
===

* ACS will be able to access a centralized registry of policies and
  environments


Long Term Plans
===============

TODO
====

* How to configure hyperparameters?

* How to reuse a model from a previous run?

* What are the key components that we want to expose in our registry?
  Candidates: Agents, Policies, Environments, Learning Strategies, Memory
  Stores.

See Also
========
* `AgentOS Issue 68: Registery for Envs, Policies, and Agents <https://github.com/agentos-project/agentos/issues/68>`_
