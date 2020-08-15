## Gitlab Pages on self-hosted Kubernetes

Gitlab pages is broken when gitlab is deployed on a self hosted kubernetes cluster.
Work to fix that is underway, described in this epic:
https://gitlab.com/groups/gitlab-org/-/epics/3901
but we needed a solution sooner.

The "solution" is to use a separate pod/container running the gitlab-pages daemon
downloaded and built from git:
```
git clone https://gitlab.com/gitlab-org/gitlab-pages.git
cd gitlab-pages
make
```

When starting the `gitlab-pages` you need to pass it a root pages directory:
```
./gitlab-pages -listen-http ":80" -pages-root /tmp/pages -pages-domain pages.yourdomain
```

Pages are generated as part of a CI pipeline, the job name attribute is set to "pages".

This program uses the gitlab API to find all projects and for each project the jobs
that were run. The youngest of the jobs that is younger than a certain timestamp will
be considered, its artifacts.zip is downloaded and unpacked in the proper place, which
is `<pages_root>/<namespace>/<project>`. The youngest timestamp is remembered for the
next round.

Run the `pull_pages` program with the configuration file as argument. A sample config
is provided, fill in the URL and your personal gitlab access token. The token should
come from the admin user and have API read access rights.

### Dependencies

Inside the container you will need python3 and some modules. You should at least do
something like
```
pip3 install --upgrade requests python-gitlab configparser
```

