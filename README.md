# dnd-vaelith-tir

Two methods to deploy a D&D campaign website.

## Architecture

There are a few different things going on in this repo. I'm using this one repo for two different deployment methods:

* **Harder method**: The `docker` directory uses a `docker-compose.yml` file to deploy your D&D site as a Bookstack site to whatever docker host you use. This README has instructions for deploying to a public Linode host.

* **Easier method**: The `gh-pages` directory uses the [MkDocs](https://www.mkdocs.org) static site generator to deploy your D&D site to GitHub pages for free.

Unlike the `gh-pages` deployment, where all of the site content is in Markdown and YAML files, the `docker` deployment only deploys the underlying infrastructure. You will need to log into your new Bookstack site when using that deployment method to create your website content.

## Docker Deployment

This deploys Bookstack from a docker-compose file in a Docker Linode container.

## Bookstack in Linode

Here's what I did to set it up:

1. Make sure my `docker/.env` file is appropriately set up. You can use my `docker/.env.sample` as an example.

2. Create account at Linode

3. Go to Marketplace > Docker, and then fill out the docker options.

### Docker Options

Resource to download (use your own repo's docker-compose.yml file of course): `https://raw.githubusercontent.com/willquill/dnd-vaelith-tir/main/docker/docker-compose.yml`

Leave "Command to run?" blank

Add a user and password, and (optionally) an SSH public key.

### Deployment

By then creating this Docker Linode, it will automatically deploy three containers:

* bookstack, bookstack_db, and swag (the reverse proxy with Let's Encrypt)

The Linode interface will show you the IP of your new public host. Use this as the A record in your DNS records for the domain you are using.

For example, my domain is `rakara.net`, so I created an A record pointing the `vaelith-tir` subdomain in the `rakara.net` domain to the public IP of my Linode host.

After deployment, you will need to SSH into your new Linode host and run the following command, replacing `vaelith-tir` with the name of your subdomain if your subdomain is not `bookstack`:

`sed -i 's/bookstack./vaelith-tir/g' /root/config/swag/nginx/proxy-confs/bookstack.subdomain.conf.sample`

Followed by:

`mv /root/config/swag/nginx/proxy-confs/bookstack.subdomain.conf.sample /root/config/swag/nginx/proxy-confs/bookstack.subdomain.conf`

And lastly:

`docker restart swag`

The first command replaces the default subdomain of `bookstack` in your nginx conf file with your unique subdomain (if you aren't using `bookstack`).

The second command removes `.sample` from the end of the file so it can be used by swag.

The third command restart swag to use your new configuration.

### Troubleshooting

If at any point you need to modify your docker-compose file, you can find it in `/root/docker-compose.yml` on your linode host.

**Important: Changes to the `docker-compose.yml` file on GitHub do not reflect on your Linode host. Any changes to `docker-compose.yml` after initial deployment must take place directly on the Linode host. I recommend making any changes in two places - your repository and your host - to keep them in sync.**

## Github Pages Deployment

### Setup

`pip install mkdocs mkdocs-material`

`export PATH=/home/will/.local/bin:$PATH` (use your own path of course)

### Clone the repo

`git clone https://github.com/willquill/dnd-vaelith-tir.git`

(or fork my repo and clone your fork)

### Modify files

Modify mkdocs.yml to reflect your own site_name, pages, and theme.

### Build

`mkdocs build`

`echo "site/" > .gitignore`

### Test

`mkdocs serve`

Visit [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser to see it in action.

Wanna know what's _really_ cool? You can edit your files on the fly and see the effect while `mkdocs serve` is active.

### Deploy

#### Put your Github private SSH key into ~/.ssh

For example, I'm using id_rsa_willquill

`chmod 600 ~/.ssh/id_rsa_willquill`

#### Create ~/.ssh/config file and put in these contents but customized for you

`vi ~/.ssh/config`

```sh
Host github-as-willquill
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_willquill
IdentitiesOnly yes
```

#### Create your own repo on Github

Go to github and create a public repository.

#### Add new public repo

Add the remote repo using the Host name from the config file above and use youruser/yourrepo.git

`git remote add origin git@github-as-willquill:willquill/dnd-vaelith-tir.git`

#### If you've enabled e-mail privacy in Github

1. Go to the Github Emails page [here](https://github.com/settings/emails)

2. Copy the Github-provided email on that page and paste the following into your terminal:

`git config user.email theid+youruser@users.noreply.github.com`

More info [here](https://stackoverflow.com/questions/43378060/meaning-of-the-github-message-push-declined-due-to-email-privacy-restrictions)

#### Add a CNAME if you want a custom domain

`echo 'vaelith-tir.rakara.net' >> docs/CNAME`

More info on custom domain [here](https://medium.com/@hossainkhan/using-custom-domain-for-github-pages-86b303d3918a)

#### Push to your master branch for good measure

```sh
git add -A
git commit -m "initial commit"
git push -u origin main
```

#### Deploy

`mkdocs gh-deploy`

### Notes about updating site after initial deployment

Never modify files directly within the `site` directory.

To make changes to your website after the initial deploy:

* Modify your yml and md files as needed.

* `mkdocs gh-deploy`

Breakdown of what `mkdocs gh-deploy` does:

* Repopulates the `site` directory with a static site generated from your md and yml files.

* Pushes the contents of the `site` directory into the gh-pages branch on Github.

## LICENSE

Distributed under the MIT License. See LICENSE for more information.
