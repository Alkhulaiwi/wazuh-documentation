.. Copyright (C) 2015, Wazuh, Inc.

.. meta::
  :description: The Wazuh Syscollector module is in charge of collecting system information and storing it into an SQLite database for each agent on the manager side.

Compatibility matrix
====================

The Syscollector module supports different options across various operating systems that the Wazuh agent can be installed on. The following table shows the scans that are compatible with various operating systems.

+------------------------+----------------------------------------------------------------------------------+
|                        |                      **Syscollector scan**                                       |
+  **Operating System**  +-----------+-----------+-----------+----------+-----------+-----------+-----------+
|                        |  Hardware |    OS     |  Packages |  Network |   Ports   | Processes |  Hotfixes |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
|    Windows             |     ✓     |     ✓     |     ✓     |     ✓    |     ✓     |     ✓     |     ✓     |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
|    Linux               |     ✓     |     ✓     |     ✓     |     ✓    |     ✓     |     ✓     |     ✗     |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
|    macOS               |     ✓     |     ✓     |     ✓     |     ✓    |     ✓     |     ✓     |     ✗     |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
|    FreeBSD             |     ✓     |     ✓     |     ✓     |     ✓    |     ✗     |     ✗     |     ✗     |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
|    OpenBSD             |     ✓     |     ✓     |     ✗     |     ✓    |     ✗     |     ✗     |     ✗     |
+------------------------+-----------+-----------+-----------+----------+-----------+-----------+-----------+
