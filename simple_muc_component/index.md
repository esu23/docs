---
title: Simple MUC Component
author: Daniel Gultsch
---
# Introduction

In this tutorial we are going to implement a simple group chat service from scratch. While not very useful by itself it has great potential to teach you a bit about the inner workings of XMPP and serve as a starting off point into component development.

Components in general and group chat components in particular are also a building block for gateways (transports into other (instant messaging) networks). A hypothetical RocketChat or Mattermost gateway would probably act as a group chat service. Therefor the knowledge gained by working through this tutorial can be very beneficial to anyone attempting to develop such a gateway.

# Components vs Bots

You might already be familiar with the concepts of chat bots. From a protocol prespective chats bots are nothing more than regular XMPP clients. Each will use an account on an XMPP server and be available under the Jabber ID of that account. They are quite easy to setup since you don’t need to run your own XMPP server. Communicating with a chat bot happens by optionally adding the JID of the bot to your roster and then sending simple text commands or even (when your client supports that) something like [Ad-Hoc Commands](https://xmpp.org/extensions/xep-0050.html). Bots can be great tools for providing simple functionality like translation bots, home automation or other kinds of task delegation.

Components on the other hand extend server functionality. Components on a server are usually discoverable to its users by means of [Service Discovery](https://xmpp.org/extensions/xep-0030.html). Each component will get an entry in the [Items Node](https://xmpp.org/extensions/xep-0030.html#items-nodes) of a server and users will make an (automated) attempt at discovering that additonal functionality provided by their server. Components are available on their own domain (usually but not necessarily a subdomain of the server). Most servers implementations come with a set of additonal services out of the box. Popular examples can include [HTTP File Upload](https://xmpp.org/extensions/xep-0363.html) (usually available under share.domain.tld or files.domain.tld), a [SOCKS5 Bytestreams Proxy](https://xmpp.org/extensions/xep-0065.html) (proxy.domain.tld) or a [Multi-User Chat](https://xmpp.org/extensions/xep-0045.html) (conference.domain.tld, muc.domain.tld, …).

The [Jabber Component Protocol](https://xmpp.org/extensions/xep-0114.html) provides an easy way to create an external service without modifying the server itself. Components can be written in almost any language (that has an XMPP library) and are basically stand alone programs that connect to an XMPP server. Requests to and from that component are proxied through the XMPP server. The TCP connection between server and component is not encrypted. Therefor the component has to run on the same machine as the XMPP server. (Also for security purposes the component port is usually only exposed on localhost.) The XMPP server will route all stanzas (Messages, IQ and Presences) where the domain part of the to attribute matches the components domain to that component. Subsequently the component can not only be addressed by its own domain but also with any Jabber ID under that domain (m̀uc.domain.tld, a@muc.domain.tld, b@muc.domain.tld, …) including full JIDs like c@muc.domain.tld/some-resource or muc.domain.tld/some-other-resource.
When responding the component can also use any of those Jabber IDs in the from attribute.

In summary components…

* Give you great flexibility and control over all localparts under a domain including the domain itself
* Are automatically discoverable by clients/users
* Require you to run your own server

# Component Setup

Explaining how to setup your own XMPP server is out of scope for this tutorial. However for your convenience here are a few words specifically regarding configuring components on the server side: On most servers the amount of configuration necessary is limited to providing a domain under which the component is going to be available and a *secret* which the external component will use to authenticate itself towards the server.

Most servers will automatically add components that are a subdomain of a configured virtual host to the items node of said virtual host. If you want to make the component available on something that is not a subdomain while still making it automatically discoverable you might have to add the component domain manually to the items (Most servers have a way of adding additional items.)

## Prosody

For Prosody you can add something along those lines to your configuration file.

```
Component "mymuc.domain.tld"
         component_secret = "mysecret"
```
The Prosody documenation on external components is available [here](https://prosody.im/doc/components).

## Ejabberd

In the listen section of `ejabberd.yml` add something like that
```
listen:
  -
    port: 5347
    ip: "127.0.0.1"
    module: ejabberd_service
    hosts:
      "mymuc.domain.tld":
        password: "mysecret"
```
The Ejabberd documentation for external components is available [here](https://docs.ejabberd.im/admin/configuration/#listening-module). Hint: Also search that documentation for the keyword *ejabberd_service* to find additional examples.

# Getting Started
