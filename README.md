# Heroku-like deploy on kubernetes

This is supposed to be a POC of joining the best of all worlds: the Heroku-like deploy we all love with the price of running on a VM with kubernetes orchestration.

Tools that make this possible:
- Kubernetes(for orchestration of containers)
- Dokku(for allowing the "git push production master" magic)
- Heroku(for open-sourcing the buildpacks)
- Herokuish(for using the buildpacks) https://github.com/gliderlabs/herokuish

