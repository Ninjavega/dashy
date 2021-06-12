# Getting Started

- [Deployment](#deployment)
  - [Deploy with Docker](#deploy-with-docker)
  - [Deploy from Source](#deploy-from-source)
  - [Deploy to Cloud Service](#deploy-to-cloud-service)
- [Usage](#usage) 
  - [Providing Assets](#providing-assets)
  - [Basic Commands](#basic-commands)
  - [Healthchecks](#healthchecks)
  - [Monitoring](#logs-and-performance)
- [Updating](#updating)
  - [Updating Docker Container](#updating-docker-container)
  - [Automating Docker Updates](#automatic-docker-updates)
  - [Updating from Source](#updating-dashy-from-source)

## Deployment

### Deploy with Docker

The quickest way to get started on any system is with Docker, and Dashy is available though [Docker Hub](https://hub.docker.com/r/lissy93/dashy). You will need [Docker](https://docs.docker.com/get-docker/) installed on your system.

To configure Dashy with your own services, and customize it to your liking, you will need to write a config file, and pass it to the Docker container as a volume. 

```docker
docker run -d \
  -p 8080:80 \
  -v /root/my-local-conf.yml:/app/public/conf.yml \
  --name my-dashboard \
  --restart=always \
  lissy93/dashy:latest
```

Explanation of the above options:
- `-d` Detached mode (not running in the foreground of your terminal)
- `-p` The port that should be exposed, and the port it should be mapped to in your host system `[host-port][container-port]`
- `-v` Specify volumes, to pass data from your host system to the container, in the format of `[host-path]:[container-path]`
- `--name` Give your container a human-readable name
- `--restart=always` Spin up the container when the daemon starts, or after it has been stopped
- `lissy93/dashy:latest` This last option is the image the container should be built from

For all available options, and to learn more, see the [Docker Run Docs](https://docs.docker.com/engine/reference/commandline/run/)

You can also build and deploy the Docker container from source.
- Get the code: `git clone git@github.com:Lissy93/dashy.git && cd dashy`
- Edit the `./public/conf.yml` file and take a look at the `docker-compose.yml`
- Start the container: `docker compose up`


### Deploy from Source
If you do not want to use Docker, you can run Dashy directly on your host system. For this, you will need both [git](https://git-scm.com/downloads) and the latest or LTS version of [Node.js](https://nodejs.org/) installed.

1. Get Code: `git clone git@github.com:Lissy93/dashy.git` and `cd dashy`
2. Configuration: Fill in you're settings in `./public/conf.yml`
3. Install dependencies: `yarn`
4. Build: `yarn build`
5. Run: `yarn start`

### Deploy to Cloud Service

Dashy supports 1-Click deployments on several popular cloud platforms.

#### Netlify
[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/lissy93/dashy)

[Netlify](https://www.netlify.com/) offers Git-based serverless cloud hosting for web applications. Their services are free to use for personal use, and they support deployment from both public and private repos, as well as direct file upload. 

To deploy Dashy to Netlify, use the following link
```
https://app.netlify.com/start/deploy?repository=https://github.com/lissy93/dashy
```

#### Heroku
[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/Lissy93/dashy)

[Heroku](https://www.heroku.com/) is a fully managed cloud platform as a service. You define app settings in a Procfile and app.json, which specifying how the app should be build and how the server should be started. Heroku is free to use for unlimited, non-commercial, single dyno apps. 

To deploy Dashy to Heroku, use the following link
```
https://heroku.com/deploy?template=https://github.com/Lissy93/dashy
```

#### Play-with-Docker
[![Try in PWD](https://raw.githubusercontent.com/play-with-docker/stacks/cff22438/assets/images/button.png)](https://labs.play-with-docker.com/?stack=https://raw.githubusercontent.com/Lissy93/dashy/master/docker-compose.yml)

[Play with Docker](https://labs.play-with-docker.com/) is a community project by Marcos Liljedhal and Jonathan Leibiusky and sponsored by Docker, intended to provide a hands-on learning environment. Their labs let you quickly spin up a Docker container or stack, and test out the image in a temporary, sandboxed environment. There's no need to sign up, and it's completely free.

To run Dashy in PWD, use the following URL:
```
https://labs.play-with-docker.com/?stack=https://raw.githubusercontent.com/Lissy93/dashy/master/docker-compose.yml
```
**[⬆️ Back to Top](#getting-started)**

---

## Usage
### Providing Assets
Although not essential, you will most likely want to provide several assets to Dashy. All web assets can be found in the `/public` directory.

- `./public/conf.yml` - As mentioned, this is your main application config file
- `./public/item-icons` - If you're using your own icons, you can choose to store them locally for better load time, and this is the directory to put them in. You can also use sub-folders here to keep things organized. You then reference these assets relative this the direcroties path, for example: to use `./public/item-icons/networking/netdata.png` as an icon for one of your links, you would set `icon: networking/netdata.png`
- Also within `./public` you'll find standard website assets, including `favicon.ico`, `manifest.json`, `robots.txt`, etc. There's no need to modify these, but you can do so if you wish.

### Basic Commands

Now that you've got Dashy running, there are a few commands that you need to know.

The following commands are defined in the [`package.json`](https://github.com/Lissy93/dashy/blob/master/package.json#L5) file, and are run with `yarn`. If you prefer, you can use NPM, just replace instances of `yarn` with `npm run`. If you are using Docker, then you will need to precede each command with `docker exec -it [container-id]`, where container ID can be found by running `docker ps`. For example `docker exec -it 26c156c467b4 yarn build`.

- **`yarn build`** - In the interest of speed, the application is pre-compiled, this means that the config file is read during build-time, and therefore the app needs to rebuilt for any new changes to take effect. Luckily this is very straight forward. Just run `yarn build` or `docker exec -it [container-id] yarn build`
- **`yarn validate-config`** - If you have quite a long configuration file, you may wish to check that it's all good to go, before deploying the app. This can be done with `yarn validate-config` or `docker exec -it [container-id] yarn validate-config`. Your config file needs to be in `/public/conf.yml` (or within your Docker container at `/app/public/conf.yml`). This will first check that your YAML is valid, and then validates it against Dashy's [schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.js).
- **`yarn health-check`** - Checks that the application is up and running on it's specified port, and outputs current status and response times. Useful for integrating into your monitoring service, if you need to maintain high system availability
- **`yarn build-watch`** - If you find yourself making frequent changes to your configuration, and do not want to have to keep manually rebuilding, then this option is for you. It will watch for changes to any files within the projects root, and then trigger a rebuild. Note that if you are developing new features, then `yarn dev` would be more appropriate, as it's significantly faster at recompiling (under 1 second), and has hot reloading, linting and testing integrated
- **`yarn build-and-start`** - Builds the app, runs checks and starts the production server. Commands are run in parallel, and so is faster than running them in independently

### Healthchecks

Healthchecks are configured to periodically check that Dashy is up and running correctly on the specified port. By default, the health script is called every 5 minutes, but this can be modified with the `--health-interval` option. You can check the current container health with: `docker inspect --format "{{json .State.Health }}" [container-id]`, and a summary of health status will show up under `docker ps`. You can also manually request the current application status by running `docker exec -it [container-id] yarn health-check`. You can disable healthchecks altogether by adding the `--no-healthcheck` flag to your Docker run command.

To restart unhealthy containers automatically, check out [Autoheal](https://hub.docker.com/r/willfarrell/autoheal/). This image watches for unhealthy containers, and automatically triggers a restart. This is a stand in for Docker's `--exit-on-unhealthy` that was proposed, but [not merged](https://github.com/moby/moby/pull/22719).

### Logs and Performance

You can view logs for a given Docker container with `docker logs [container-id]`, add the `--follow` flag to stream the logs. For more info, see the [Logging Documentation](https://docs.docker.com/config/containers/logging/). [Dozzle](https://dozzle.dev/) is a useful tool, that provides a web interface where you can stream and query logs from all your running containers from a single web app.

You can check the resource usage for your running Docker containers with `docker stats` or `docker stats [container-id]`. For more info, see the [Stats Documentation](https://docs.docker.com/engine/reference/commandline/stats/). [cAdvisor](https://github.com/google/cadvisor) is a useful web app for viewing and analyzing resource usage and performance of your running containers.

You can also view logs, resource usage and other info as well as manage your Docker workflow in third-party Docker management apps. For example [Portainer](https://github.com/portainer/portainer) an all-in-one management web UI  for Docker and Kubernetes, or [LazyDocker](https://github.com/jesseduffield/lazydocker) a terminal UI for Docker container management and monitoring.

**[⬆️ Back to Top](#getting-started)**

---
## Updating

Dashy is under active development, so to take advantage of the latest features, you may need to update your instance every now and again.

### Updating Docker Container
1. Pull latest image: `docker pull lissy93/dashy:latest`
2. Kill off existing container
	- Find container ID: `docker ps`
	- Stop container: `docker stop [container_id]`
	- Remove container: `docker rm [container_id]`
3. Spin up new container: `docker run [params] lissy93/dashy`

### Automatic Docker Updates

You can automate the above process using [Watchtower](https://github.com/containrrr/watchtower).
Watchtower will watch for new versions of a given image on Docker Hub, pull down your new image, gracefully shut down your existing container and restart it with the same options that were used when it was deployed initially.

To get started, spin up the watchtower container:

```
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower
```

For more information, see the [Watchtower Docs](https://containrrr.dev/watchtower/)

### Updating Dashy from Source
1. Navigate into directory: `cd ./dashy`
2. Stop your current instance
3. Pull latest code: `git pull origin master`
4. Re-build: `yarn build`
5. Start: `yarn start`


**[⬆️ Back to Top](#getting-started)**