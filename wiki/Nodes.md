A node is a daemon running on a machine. You can install the daemon on multiple machines, and manage them from the server (i.e.:GUI).

> ℹ️ Note: if you want to install **only** the daemon from the Debian repositories, you'll have to execute this command: 
> 
> `$ sudo apt install --no-install-recommends opensnitch` (otherwise it'll install both, the daemon and the GUI)

![image](https://user-images.githubusercontent.com/2742953/197076010-2502855a-cdae-4f03-90bc-7a715efbbf64.png)

You can view the list of known nodes from the tab Nodes:

![image](https://user-images.githubusercontent.com/2742953/82752021-9d328380-9dbb-11ea-913e-80f7b551a6c7.png)

<img width="600" src="https://user-images.githubusercontent.com/2742953/218576821-67fa3524-520a-4f5f-9656-3185a758022b.png">


And by double clicking on a node, you can see the network activity for that node.

#### Configuration

As explained in the [Configurations](https://github.com/evilsocket/opensnitch/wiki/Configurations#gui) section, you have to launch the daemon as follow in order to accept connections from remote nodes:

`$ /usr/local/bin/opensnitch-ui --socket "[::]:50051"`

It'll make the GUI listen on port 50051, any IP. You can also use an IP: `$ /usr/local/bin/opensnitch-ui --socket "127.0.0.1:50051"`

Now you need to configure each node to connect to the server. In `/etc/opensnitchd/default-config.json` set the Address field to the server address:

```json
    "Server":
    {
        "Address":"192.168.1.100:50051",
    },
```

Once a node is connected, you can also change it from the GUI, without connecting to the node via SSH, etc:

![image](https://user-images.githubusercontent.com/2742953/196782343-bbc28fea-f9a1-4842-a285-e557c6ac5b27.png)

(the field Address refers to the server address where the node will connect to)

