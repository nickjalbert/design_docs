===================
AgentOS Design Docs
===================

This repo holds design documents and proposals for AgentOS internals.  If you
are looking for documentation for the AgentOS library itself, visit
`agentos.org <https://agentos.org>`_


============
Contributing
============

If you'd like to contribute a design document or proposal, please:

* Fork this repo.

* Write your document in `reStructuredText
  <https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html>`_.
  Also, see the `reStructuredText Quick Reference
  <https://docutils.sourceforge.io/docs/user/rst/quickref.html>`_.

* Issue a pull request from your fork into the main repo and assign one of the
  maintainers to review.

* If the reviewer:

  * Thinks the design is generally in the right ballpark but needs some minor
    to moderate revision and brainstorming, the PR gets an 'LGTM' and merges
    after any immediate nits are addressed.

  * Thinks the design needs more foundational revisions, the PR gets closed
    and the reviewer and reviewee sync on next steps.

  * Wants to discuss over Skype or in person, the PR remains open until the
    meeting.

* Once a design doc is merged, additional updates come as new PRs.

* A merged document can then be used as a basis for issue creation in the main
  AgentOS project.

==================
Recommended Format
==================

Consider adding the following sections and items in your design document:

* Abstract - A brief paragraph summary of the design

* Rationale - The reason why this feature is important or useful

* Demo - A "demo" of what the feature will look like after implementation.
  This can contain pseudocode or command line examples.

* Work items - This is a set of items required to accomplish the proposed
  design.

* FAQ - Frequently asked questions.  Especially useful to address questions
  from your reviewer.

* Open Questions - A section to record things that should be investigated or
  may change as the design gets implemented.

* Links to any pull requests related to the document so comment history and
  debate can be discovered.
