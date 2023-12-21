===========
 ChangeLog
===========


Version 1.0.0 (TBD)
============================

Note: this is the first release since forking from spesmilo/electrumx.

.. 
    * security: a vulnerability has been fixed that allowed a remote attacker to
    crash electrumx if peer discovery was enabled (`#22`_)
    * fixed some peer-discovery-related bugs (e.g. `#35`_)
    * ENV: when using Bitcoin, the COIN ENV var can now be set to :code:`Bitcoin`.
    For compatibility, using :code:`BitcoinSegwit` will also keep working.
    (`#5`_)
    * session resource limits: made more useful in general. connection-time-based
    grouping has been removed (`#70`_). Disconnects of over-limit sessions happen
    sooner (`4b3f6510`_). Subnet sizes for IP-based grouping can now be
    configured (`a61136c5`_).
    * protocol: :code:`mempool.get_fee_histogram` now returns fee rates with
    0.1 sat/byte resolution instead of 1 sat/byte, and the compact histogram
    is calculated differently. (`#67`_)
    * performance: bitcoind is now queried less frequently as estimatefee,
    relayfee, and server.banner requests are now cached (`#24`_)
    * performance: json ser/deser is now abstracted away and the :code:`ujson` and
    :code:`rapidjson` exras can be used for somewhat faster block processing.
    (`#11`_)
    * multiple other small performance improvements


Original author of ElectrumX:

**Neil Booth**  kyuupichan@gmail.com  https://github.com/kyuupichan

Author of Fork utilized for Dimecoin:

**Electrum developers** electrumdev@gmail.com  https://github.com/spesmilo


.. _#554: https://github.com/kyuupichan/electrumx/issues/554
.. _#566: https://github.com/kyuupichan/electrumx/issues/566
.. _#653: https://github.com/kyuupichan/electrumx/issues/653
.. _#655: https://github.com/kyuupichan/electrumx/issues/655
.. _#660: https://github.com/kyuupichan/electrumx/issues/660
.. _#684: https://github.com/kyuupichan/electrumx/issues/684
.. _#713: https://github.com/kyuupichan/electrumx/issues/713
.. _#727: https://github.com/kyuupichan/electrumx/issues/727
.. _#731: https://github.com/kyuupichan/electrumx/issues/731
.. _#795: https://github.com/kyuupichan/electrumx/issues/795
.. _#909: https://github.com/kyuupichan/electrumx/issues/909


.. _#5:   https://github.com/spesmilo/electrumx/pull/5
.. _#11:  https://github.com/spesmilo/electrumx/pull/11
.. _#22:  https://github.com/spesmilo/electrumx/issues/22
.. _#24:  https://github.com/spesmilo/electrumx/pull/24
.. _#35:  https://github.com/spesmilo/electrumx/pull/35
.. _#67:  https://github.com/spesmilo/electrumx/pull/67
.. _#70:  https://github.com/spesmilo/electrumx/pull/70


..  
    .. _4b3f6510:  https://github.com/spesmilo/electrumx/commit/4b3f6510e94670a013c1abe6247cdd2b0e7e6f8c
    .. _a61136c5:  https://github.com/spesmilo/electrumx/commit/a61136c596d6a0290a6be9d21fb7c095c3cea21e