Last modified: 17 January 2023

This installation type implies that you use Docker Compose to install Space On-Premises components to several Docker containers.

We recommend it as a proof-of-concept installation that lets you test and familiarize yourself with the system before using it in a production setting. Out of the box, Docker Compose installation doesn't provide any solutions that prevent data loss or ensure service stability. Before you can use it as your production environment, you should perform some additional configuration. For example, we strongly recommend that you provide Postgres, Elasticsearch, and a MinIO-compatible storage as external services.