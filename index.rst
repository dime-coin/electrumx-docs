.. Electrum-Dime documentation master file, created by
   sphinx-quickstart on Wed Dec 20 14:01:02 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to the ElectrumX-Dime Documentation!
=============================================

.. _source:

ElectrumX-Dime is a fork of `spesmilo/electrumx <https://github.com/spesmilo/electrumx>`_. ElectrumX-Dime allows users to run their own ElectrumX-Dime server. It connects to your full node and indexes the blockchain, allowing efficient querying of history of arbitrary addresses. The server can be exposed
publicly, and joined to the public network of servers via peer discovery.

The current version of ElectrumX-Dime is |release|.

.. toctree::
   :maxdepth: 2
   :caption: Introduction

Source Code
-------------------

.. _git:

The project is hosted on `GitHub <https://github.com/dime-coin/electrumx-dimecoin>`_.

.. _bugtracker:

Please submit an issue on the `bug tracker <https://github.com/dime-coin/electrumx-dimecoin/issues>`_ if you have found a bug or have a suggestion to improve the server.

Authors and License
-----------------------

Neil Booth wrote the majority of the original code; see Authors for additional credits. Python version >= 3.8 is required.

.. _mit:

The code is released under the `MIT License <https://github.com/dime-coin/electrumx-dimecoin/LICENCE>`_.


.. toctree::
   :maxdepth: 2
   :caption: Getting Started

   See :ref:`How-To`.


.. toctree::
   :maxdepth: 2
   :caption: Documentation

   features
   changelog
   How-To
   environment
   protocol
   peer_discovery
   rpc-interface
   architecture
   authors

Indices and Tables
=============================
* :ref:`genindex`
* :ref:`search`