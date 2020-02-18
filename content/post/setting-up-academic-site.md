---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Setting Up Academic Site"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-02-17T12:11:04-08:00
lastmod: 2020-02-17T12:11:04-08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

In this post we setup a static site using the `hugo` static site generator. We then serve this site at a domain name we own, using Google Cloud, and we set up automatic deployment of modifications to the site using `hugo`'s automatic deployment targets. I did this on Ubuntu 18.04 LTS, and have generally found that this process is extremely sensitive to the correct versions of `go`, `hugo`, and in my case the version of `academic-kickstart` that I use as a base for the website. Keeping everything up-to-date is slightly involved, as updates to the `academic-kickstart` repository will generically break the site. Also, `go` is installed from the system package manager, but the package for `hugo` is broken, and so `hugo` must be installed from its `git` repo, or from a tar ball.

Steps covered in this post: 

1. Install `go`
2. Install `hugo`
3. Get started with `academic-kickstart`
4. Initialize and edit site
5. Register with Google Cloud Platform
6. Download and authenticate Google SDK
7. Setup a deployment target in site's config

# Install `go`

As per [the `go` repository's instructions](https://github.com/golang/go/wiki/Ubuntu), we install `go` like this:

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go
```

`go` modules will now be installed in `$HOME/go/`.

# Install `hugo`

The version of `hugo` in the PPA is broken, and so we install from the `hugo` [git repo](https://github.com/gohugoio/hugo): 

```bash
mkdir $HOME/src
cd $HOME/src
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install
```

where the `$HOME/src` directory is the default destination for `go` modules. Running `go install` then places the `hugo` binary in `$HOME/go/bin`. Ensure that this directory is added to your path. Check your `hugo` version with `hugo version` to ensure that you are up-to-date.

# Get started with `academic-kickstart`

We will use the `academic-kickstart` git project as a base for the website. This is a skeleton project that provides both a theme (`academic`), and a skeleton structure for a website, which contains many of the widgets useful for an academic's personal site (publications, projects, etc...). There are multiple ways of getting started with this, I personally forked the repo, and cloned it to my computer. Alternatives are to use a tar ball (becomes hard to manage updates, which generically break the site), or through a service called Netlify, which I do not know anything about. 

To do this step you can follow [these instructions](https://sourcethemes.com/academic/docs/deployment/) in detail, or go [here](https://github.com/gcushen/hugo-academic) to grab the template if you know what you're doing. 

Assuming you now have a website in `$HOME/website` and are using the academic theme: `$HOME/website/themes/academic`, we can proceed to edit all the configuration files to personalize the website. These are the `$HOME/website/config/*.toml`. Add content through the various templates (e.g. project, publication, post) and build your website by building with `hugo`. We can check the site as we develop it by doing`hugo server` in the `$HOME/website` directory, which automatically has live updates, and serves the website by default at `localhost:1313`.

Once the site is ready to be deployed, just build it with `hugo`, and this will build the site in the `$HOME/website/public` folder. The content of `public` is then uploaded to whatever service for deployment.

# Register with Google Cloud Platform (GCP)

It is possible to serve static sites in various ways. The cheapest and most convenient is often to use GitHub pages, which allows one static site per repo. This is a fine thing to do, but to be adventurous we have decided to use GCP. The advantages of GCP will not be necessary for us (basically unlimited scalability), but it is basically free (I have free credits), and is a good tool to learn. 

There is actually a great [Google codelabs](https://codelabs.developers.google.com/codelabs/cloud-webapp-hosting-gcs/index.html?index=..%2F..index#8) guide on hosting static websites, which I can not improve upon.

Once you have followed that guide, you should have a public site, with the contents of your `$HOME/website/public` folder hosted on a Google Storage bucket. 

# Automate deployment with Google SDK and `hugo deploy`

It will be convenient to be able to deploy the website quickly when we make changes, rather than build the website, and upload the modified `public` directory via the GCP GUI, we can specify a deployment target in our website's `config.toml`. With a deployment target (in our cast our GCP storage bucket) we can then modify files and simply run `hugo; hugo deploy`, and the deployment is taken care of. 

To get this functionality we need the command line interface for Google cloud, the Google SDK. Download and install this from [here](https://cloud.google.com/sdk) and go through the process of authentication. We need to set a default authentication login so that we can run the deployment, so do: `gcloud auth application-default login`, which should run an OAuth authentication through your browser. 

Now edit your `config/config.toml` file, and add a `[deployment]` section like this:

```toml
[deployment]
# By default, files are uploaded in an arbitrary order.
# Files that match the regular expressions in the "Order" list
# will be uploaded first, in the listed order.
order = [".jpg$", ".gif$"]

[[deployment.targets]]
# Note that this deployment requires an authenticated google sdk
# An arbitrary name for this target.
name = "mydeployment"
# The Go Cloud Development Kit URL to deploy to. Examples:
# GCS; see https://gocloud.dev/howto/blob/#gcs
URL = "gs://<name of bucket>"
```

where you should replace `name of bucket` with whatever your own bucket name (you can check the list of your buckets with `gsutil ls`).


And that's it! Now you can modify your website, create posts, create projects, update personal information, and once you are done just run `hugo; hugo deploy` and your changes will be reflected on your website!