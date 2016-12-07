## ejabberd Community Edition - Base

This ejabberd Docker image allows you to run a single node ejabberd instance in a Docker container.

## Running ejabberd

### Default configuration for domain localhost

You can run ejabberd in a new container with the following command:

```bash
docker run --name ejabberd -d -p 5222:5222 ejabberd/ecs
```

This command will run Docker image as a daemon, using ejabberd default configuration file and XMPP domain "localhost".

To stop the running container, you can run:

```bash
docker stop ejabberd
```

If needed you can restart the stopped ejabberd container with:

```bash
docker restart ejabberd
```

### Running ejabberd with Erlang console attached

If you would like to run it with console attached you can use the `console` command:

```bash
docker run -it -p 5222:5222 ejabberd/ecs console
```

This command will use default configuration file and XMPP domain "localhost".

### Running ejabberd with your config file and database host directory

The following command 

```bash
mkdir db
docker run --name ejabberd -v $(pwd)/ejabberd.yml:/home/p1/cfg/ejabberd.yml -v $(pwd)/db:/home/p1/db -p 5222:5222 ejabberd/ecs
```

## Docker image advanced configuration

### Files

Here are the important files you can replace by mounting persistent disk or local directory:

- /home/p1/cfg/ejabberd.yml: ejabberd configuration file.

### Ports

ejabberd base Docker image exposes the following port:

- 5222: This is the default XMPP port for clients.
- 5280: This is the port for admin interface, API, Websockets and XMPP BOSH.
- 5269: Optional. This is the port for XMPP federation. Only needed if you want to communicate with users on other servers.

### Volumes

ejabberd produces two type of data: log files and database (Mnesia).
This is the kind of data you probably want to store on a persistent or local drive (at least the database).

Here are the volume you may want to map:

- /home/p1/log/: Directory containing log files
- /home/p1/db/: Directory containing Mnesia database. You should backup or export the content of the directory to persistent storage (host storage, local storage, any storage plugin)

## Generating ejabberd release

### Configuration

Image is build by embedding an ejabberd Erlang/OTP standalone release in the image.

The configuration of ejabberd Erlang/OTP release is customized with:

- rel/config.exs: Customize ejabberd release
- rel/dev.exs: ejabberd environment configuration for development release
- rel/docker.exs: ejabberd environment configuration for production Docker release
- ejabberd.yml: ejabberd default config file 

Run the build script to generate ejabberd Community Server base image from ejabberd master on Github:

```bash
./build.sh
```

### TODO

- Embed command-line tool for ejabberd API to be able to create admin user for ejabberd.