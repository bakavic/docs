# What is RabbitMQ?

RabbitMQ is open source message broker software (sometimes called message-oriented middleware) that implements the Advanced Message Queuing Protocol (AMQP). The RabbitMQ server is written in the Erlang programming language and is built on the Open Telecom Platform framework for clustering and failover. Client libraries to interface with the broker are available for all major programming languages.

> [wikipedia.org/wiki/RabbitMQ](https://en.wikipedia.org/wiki/RabbitMQ)

%%LOGO%%

# How to use this image

## Running the daemon

One of the important things to note about RabbitMQ is that it stores data based on what it calls the "Node Name", which defaults to the hostname. What this means for usage in Docker is that we should specify `-h`/`--hostname` explicitly for each daemon so that we don't get a random hostname and can keep track of our data:

	docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3

If you give that a minute, then do `docker logs some-rabbit`, you'll see in the output a block similar to:

	=INFO REPORT==== 6-Jul-2015::20:47:02 ===
	node           : rabbit@my-rabbit
	home dir       : /var/lib/rabbitmq
	config file(s) : /etc/rabbitmq/rabbitmq.config
	cookie hash    : UoNOcDhfxW9uoZ92wh6BjA==
	log            : tty
	sasl log       : tty
	database dir   : /var/lib/rabbitmq/mnesia/rabbit@my-rabbit

Note the `database dir` there, especially that it has my "Node Name" appended to the end for the file storage. This image makes all of `/var/lib/rabbitmq` a volume by default.

### Erlang Cookie

See the [RabbitMQ "Clustering Guide"](https://www.rabbitmq.com/clustering.html#erlang-cookie) for more information about cookies and why they're necessary.

For setting a consistent cookie (especially useful for clustering but also for remote/cross-container administration via `rabbitmqctl`), use `RABBITMQ_ERLANG_COOKIE`:

	docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' rabbitmq:3

This can then be used from a separate instance to connect:

	$ docker run -it --rm --link some-rabbit:my-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' rabbitmq:3 bash
	root@f2a2d3d27c75:/# rabbitmqctl -n rabbit@my-rabbit list_users
	Listing users ...
	guest   [administrator]

Alternatively, one can also use `RABBITMQ_NODENAME` to make repeated `rabbitmqctl` invocations simpler:

	$ docker run -it --rm --link some-rabbit:my-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' -e RABBITMQ_NODENAME=rabbit@my-rabbit rabbitmq:3 bash
	root@f2a2d3d27c75:/# rabbitmqctl list_users
	Listing users ...
	guest   [administrator]

### Management Plugin

There is a second set of tags provided with the [management plugin](https://www.rabbitmq.com/management.html) installed and enabled by default, which is available on the standard management port of 15672, with the default username and password of `guest` / `guest`:

	docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3-management

You can access it by visiting `http://container-ip:15672` in a browser or, if you need access outside the host, on port 8080:

	docker run -d --hostname my-rabbit --name some-rabbit -p 8080:15672 rabbitmq:3-management

You can then go to `http://localhost:8080` or `http://host-ip:8080` in a browser.

## Connecting to the daemon

	docker run --name some-app --link some-rabbit:rabbit -d application-that-uses-rabbitmq
