.. AutoData documentation master file

AutoData
========

**AutoData** is a pioneering multi-agent system designed to revolutionize data collection from the open web.

.. note::
   
   This documentation is for the **AutoData** project.
   
   The development repository **AutoData-DEV** is private and used for active development.
   The public repository is available at `AutoData <https://github.com/Tianyi-Billy-Ma/AutoData>`_.

Introduction
------------

AutoData automates the generation of crawlers and the extraction of data from diverse online sources, addressing the complexities of modern web environments.

Key Features
~~~~~~~~~~~~

* **Multi-Agent Architecture**: Orchestrated by a Supervisor Agent managing specialized Research and Development squads.
* **OHCache**: A novel context management system that optimizes information flow between agents.
* **Open Web Adaptability**: Capable of handling complex, dynamic websites.
* **Automated Blueprinting**: Synthesizes research findings into executable Python crawling code.

Installation
------------

To install AutoData, clone the public repository:

.. code-block:: bash

   git clone https://github.com/Tianyi-Billy-Ma/AutoData.git
   cd AutoData
   uv sync --group dev,test,docs
   playwright install
   playwright install-deps

Usage
-----

To run a sample task:

.. code-block:: bash

   uv run python -m autodata.main --config configs/default.yaml

Citation
--------

If you use AutoData in your research, please cite our NeurIPS 2025 paper:

.. code-block:: bibtex

   @inproceedings{autodata2025,
     title={AutoData: A Multi-Agent System for Open Web Data Collection},
     author={Tianyi-Billy-Ma and Contributors},
     booktitle={NeurIPS},
     year={2025},
     url={https://arxiv.org/abs/2505.15859}
   }

.. toctree::
   :maxdepth: 2
   :caption: Contents:

