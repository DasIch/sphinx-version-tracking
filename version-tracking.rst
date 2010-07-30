Versioning Docutils/Sphinx generated doctrees
=============================================
In the following sections I will explain an approach which could be used to
make nodes identifiable across multiple builds and how doctrees based on such
identifiable nodes can be merged

.. contents::

Rationale
---------
The projects `sphinx-web-support`_ and `sphinx-i18n`_ and potentially future
projects around Sphinx_ require a way to track and document changes made to the
documentation itself. Currently neither `Sphinx`_ nor `Docutils`_ offer a way
to do that.

As described in the beginning of this document the goal of this proposal is to
show a way how to implement the necessary API to make tracking of changes
possible.

The Document Tree
-----------------
The document tree or doctree is a tree-like structure which represents a
document written in `reStructuredText`_, structures such as this are commonly
known as `AST (abstract syntax tree)`_. An AST can be used to
(semantically) analyze, interpret or generate source code based on the source
code from which the AST was generated.

`Sphinx`_ creates the document tree and links it to other document trees
generated from the documents source code which is found in form of files in a
user-defined source directory. The result is then interpreted and processed by
a so-called builder which analyzes the documentation or compiles it into a
different format e.g. `HTML`_.

Implementation
--------------
Due to the fact that it is necessary to access doctrees of earlier versions
to perform a merge I propose to base the implementation on a doctree builder,
ideally other builders should be able to subclass the builder to use the
functionality it provides.

In order to identify a node there are two easy solutions, the first one is a
simple counter, however upon creating a new doctree we need to get the last
used identifier which requires traversing the entire tree or setting the
information on the doctree. A much easier approach is to simply use
`UUIDs`_ which always yield unique identifiers and we don't have to
worry about conflicts.

Without a previous doctree our builder simply attaches an identifier to every
node in the new doctree.

When we have an already existing doctree with ids and we are creating a new one
we traverse both trees at the same time. Text nodes - nodes which represent
text but may contain inline markup - or another kind of node which is specified
as "leaf" are compared using a diff algorithm which
has three potential outcomes:

1. Both nodes are identical: This means we simply use the identifier from the
   node we already have and attach it to the one belonging to the newly
   generated doctree.

2. Both nodes are similar: This means this part of the documentation has
   changed and again simply use the identifier of one node and belonging to
   the tree we already have to the new one.

3. Both nodes are completely different: This happens if someone inserts or
   removes one or more nodes. What we do is that we store both nodes in two
   seperate lists and go on the next pair of nodes. If the next pair again
   is completely different we check if we have any previous cases and diff
   against them in the hope of clearing the list. If the unidentified node
   matches against the last node in the list of identified nodes we perform
   a shift and check the next node in the new tree against the old node in the
   current pair. The purpose of this is to "recover" from inserts or removals.
   Once both trees are completely traversed and the list of unindentified nodes
   is not empty each node in it get's a new identifier.

Diff Algorithm
--------------
The diff algorithm used is one of the most important parts of the
implementation and should be as fast as possible. However due to my lack of
knowledge in the area I think the best approach is to use the diff algorithm
in Python's standard library, which does not introduce a new dependency nor
requires time to implement, test and document. Should the algorithm prove to be
inefficient it could be changed later on.

.. _sphinx-web-support: http://gsoc.jacobmason.us/blog/
.. _sphinx-i18n: http://gsoc.robertlehmann.de/
.. _Docutils: http://docutils.sourceforge.net/
.. _Sphinx: http://sphinx.pocoo.org/
.. _reStructuredText: http://docutils.sourceforge.net/rst.html
.. _AST (abstract syntax tree): http://en.wikipedia.org/wiki/Abstract_syntax_tree
.. _HTML: http://www.w3.org/TR/1999/REC-html401-19991224/
.. _UUIDs: http://tools.ietf.org/html/rfc4122
