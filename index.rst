.. Electrum-Dime documentation master file, created by
   sphinx-quickstart on Wed Dec 20 14:01:02 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to the ElectrumX-Dime Documentation!
=============================================

.. _source:

ElectrumX-Dime is a fork of `spesmilo/electrumx <https://github.com/spesmilo/electrumx>`_. ElectrumX-Dime allows users to run their own ElectrumX-Dime server. It connects to your full node and indexes the blockchain, allowing efficient querying of history of arbitrary addresses. The server can be exposed
publicly, and joined to the public network of servers via peer discovery.

The current version of ElectrumX-Dime is 1.0.

.. toctree::
   :maxdepth: 2
   :caption: Introduction

   source



.. toctree::
   :maxdepth: 2
   :caption: Advanced Users

   coldstorage_cmdline
   hardfork
   tor
   gpg-check


.. toctree::
   :maxdepth: 2
   :caption: Daemon and Command Line

   cmdline
   ssl
   merchant
   watchtower
   jsonrpc


.. toctree::
   :maxdepth: 2
   :caption: For Developers

   console
   spv
   seedphrase
   protocol
   transactions
   plugin_rules
