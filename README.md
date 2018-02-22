# SLARP
**S**imple {**L**et's encrypt + **A**pache} **R**everse **P**roxy


## Goal

An easy way to forward HTTP requests to backend web servers, while ensuring SSL termination.

The typical situation is:
* You have several containers on your machine that act as web servers.
* You need to route every incoming request to the right container.
* Some/all of them must be accessed via HTTPS.
* It's better to have a single place where SSL certificates are managed.
