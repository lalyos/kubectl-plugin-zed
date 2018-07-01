When working with Kubernetes, **how do you edit files** inside of a pod/container?
I guess its **kubectl exec** and than use good old **vi**.
Would you like to use a modern editor in chrome or as a standalone?

Lets use [ZedApp](http://zedapp.org).

This kubectl plugin can open a container directory in zed editor as:

```
kubectl plugin zed <PODNAME> /etc/
```

Note: right now this plugin relies on minikube.

## Installation

There are 2 components:
- the ZedApp editor itself
- this kubectl plugin, which can drive the editor

Installing this plugin is easy:
```
mkdir -p ~/.kube/plugins/
git clone https://github.com/lalyos/kubectl-plugin-zed.git ~/.kube/plugins/zed
```

To validate that the plugin installation was succesfull run the plugin command:
```
$ kubectl plugin
Runs a command-line plugin. 

Available Commands:
  zed         ZedApp editor
```

For installing the editor you have 2 options:
- Use it as a Chrome App browser extensio
- Install it as a standalone application (osx/linux/windows)

## Installing Zed editor as ChromeApp

Go to [ChromeStore](https://chrome.google.com/webstore/detail/zed-code-editor/pfmjnmeipppmcebplngmhfkleiinphhp)
and click `Add To Chrome`

## Installing Zed editor as Desktop App

For OSX use brew
```
brew cask install zed
```

For linux and windows use the [Download page](http://zedapp.org/download/)

## Configuration

The Zed editor should connect to zedserver (running as a svc in the zed ns).
Open Zed in [Chrome Apps](chrome://apps), and open **Configuration / Preferences / Remote Editing / Zedrem server**

To get the zedserver address, run the zed kubectl plugin without a podname:
```
$ kubectl plugin zed
Zed remote server should point: ws://192.168.64.3:30342
```

While editing **Preferences** you can also make the file tree pane always visible. 
Tick the checkbox at: **Configuration / Preferences / UI / Persistent tree**.


Once you configured zed to connect to zedserver, you can get a uniqe key,
which will authorize this plugin to open a new window.

To get the **zed key** open a new zed window:
`Window / New / Open / Remote Folder`
You will see an instruction like:
```
$ ./zedrem -key 7CFAF57BB28147129C0EB09DE47F16DE -u ws://192.168.64.3:30342 .
```
Copy and paste **only the key** parameter, and configure this plugin:

```
kubectl plugin zed --key=7CFAF57BB28147129C0EB09DE47F16DE
```

## Usage

For example if you have an nginx pod, you can edit 
```
kubectl plugin zed <PODNAME> /usr/share/nginx/html/
```

You can even open the full filesystem in editor, but the startup time can be a bit slow:
```
kubectl plugin zed <PODNAME>
```

## tl;dr

zedrem is a small agent which can run on any remote machine, or in a container.
It connects to a zedserver via a websocket, and makes the remote filesystem
available for any zed editor which connects to this session.

the official zed server is running at **remote.zedapp.org**. but you can avoid a
slow network connection, and exposing sensitive information, by your own
zedserver. We will use a small (3MB) docker image [jbromley/zedrem-server](https://hub.docker.com/r/jbromley/zedrem-server/)

This plugin will create the following k8s resources
- namespace: zed
- deployment: zedserver (in zed ns)
- a service with NodePort

Zed editor connects to the zedserver (NodePort) and when a client (zedrem)
