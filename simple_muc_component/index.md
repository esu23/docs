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

The [Jabber Component Protocol](https://xmpp.org/extensions/xep-0114.html) provides an easy way to create an external service without modifying the server itself. Components can be written in almost any language (that has an XMPP library) and are basically stand alone programs that connect to an XMPP server. Requests to and from that component are proxied through the XMPP server. The TCP connection between server and component is not encrypted. Therefor the component has to run on the same machine as the XMPP server. (Also for security purposes the component port is usually only exposed on localhost.)
