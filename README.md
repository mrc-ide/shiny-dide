# DIDE shiny server

This is the configuration for [`shiny.dide.ic.ac.uk`](https://shiny.dide.ic.ac.uk), the departmental [Shiny](https://shiny.posit.co/) server.  It uses [`twinkle`](https://github.com/mrc-ide/twinkle/) to look after setting apps and to set up the docker image that we run the server from.

This system is a set of docker containers acting as a *load balanced shiny server*, using three three services:

```
<apache> -- <haproxy> -- <shiny x 12>
```

where `apache` presents the interface to the world and looks after https, `haproxy` acts as a load balancer and we have 12 copies of `shiny`, each capable of serving every application.

**Documentation for administrators**: [`admin.md`](admin.md)

## Adding an application

Hello! If you are a DIDE member wanting to host a shiny application on [`shiny.dide.ic.ac.uk`](https://shiny.dide.ic.ac.uk), you are in the right place.  We are able to host shiny applications that:

* **will be publicly accessible**: anyone with the URL can access your application, unless **you** implement some sort of authentication within the application
* **do not write to disk**: your application must be happy running on a read-only filesystem, which allows us to run many copies of it at the same time without interfering with each other (you may still write to temporary files if required, but these will not be available across sessions)
* **uses publicly available packages**: with the exception of the repo that your application is found in (which may be private) all the dependencies of your application must be publicly available on CRAN, GitHub or an R-universe.
* **can be deployed with relatively little intervention**: you will need to be able to describe what dependencies your application needs and it must not need much special coaxing to start up

Most applications that we have been asked to host satisfy these criteria; if in doubt, please as us on the [**Shiny server** Channel on Teams](https://teams.microsoft.com/l/channel/19%3A627d868d57bd420f8a97e97e8d91e9e0%40thread.tacv2/Shiny%20server?groupId=ba231111-1572-42ae-981e-c8bc7aa681ef&tenantId=2b897507-ee8c-4575-830b-4f8267c3d307).

To add an application, you need to do two things:

1. Add information about the dependencies of your application to **your** repo
2. Add information about your application to **this** repo

To add dependency information, we use the same basic system as [`hipercow`](https://mrc-ide.github.io/hipercow/articles/packages.html).  Most users will write a `pkgdepends.txt` file and put that at the same place in their repository as their application.  This is a list of packages (e.g., `dplyr`), GitHub references (e.g. `mrc-ide/drjacoby`) or CRAN-like repositories (e.g., `repo::mrc-ide.r-universe.dev`).

To add your site to this repository, please fork this repository and edit [`site.yml`](site.yml).  The format is fairly self explanatory but is described [on the `twinkle` help](https://mrc-ide.github.io/twinkle/#configuration).  In the most simple case, if you want to host an application at `shiny.dide.ic.ac.uk/potato` and the source code is on the default branch (`main` or `master`) of a GitHub repo hosted at `github.com/alice/potato-app` you would add a block:

```
  potato:
    username: alice
    repo: potato-app
```

Then make a PR into this repository (GitHub will prompt you to do this after you push) and let us know on the Shiny server channel.

Alternatively, please post your best guess at the appropriate `yaml` block into the channel and we'll add it to `site.yml` ourselves.

Once you tell us, we will try and get your application going on the server.  The first few times this usually fails, due to things like:

* missing dependencies
* missing system libraries on the server itself
* trying to write to disk
* gremlins

Expect a bit of back and forth; it's usually easier to do this if your application is in `mrc-ide` as then we can just add missing dependencies to your repo directly (if they are obvious).  We will give you a URL and ask you to check the application, and if it crashes we will send you logs which will usually indicate the error.

## Updating an application

Most of the time, this just involves you editing your repository and pushing changes.  All updates on the server though are **entirely manual**; let us know when you would like things updated and we will do this.

Please message us on the [Shiny server Channel on Teams](https://teams.microsoft.com/l/channel/19%3A627d868d57bd420f8a97e97e8d91e9e0%40thread.tacv2/Shiny%20server?groupId=ba231111-1572-42ae-981e-c8bc7aa681ef&tenantId=2b897507-ee8c-4575-830b-4f8267c3d307) and let us know that you would like your application updated.  Please always mention the name or URL of the application (so if your application is `shiny.dide.ic.ac.uk/potato` then include that url or let us know that it is `potato`).

Please do not just ask "can you update my app" as we will most likely have forgotten which is yours, and we will have to ask you which it is, which slows everything down.

## Special notes

* **If your repo is private**: if it is on `mrc-ide`, we can see it anyway.  If it is in your personal namespace consider migrating it to `mrc-ide`, otherwise we will need to get you to add a "deploy key" to the repository, and to add anyone working with your application as a collaborator.
* **If your app needs some special steps performed after update**: add a `script` key in the `site.yml` that points at a script (can be R, can be bash); this runs on a writable filesystem each time your application is updated, and can be used to download datasets.
* **If your app is within a package**: If your application uses a package in the same repository, you can use `local::.` to install it (typically, your app will be stored at `inst/app` or similar, and the root of the repository will contain `DESCRIPTION`).
