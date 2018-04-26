---
title: "Tools for the new SharePoint framework"
header:
  teaser: "images/posts/2016-05-17_teaser.jpg"
tags:
- SharePoint
- TypeScript
- Gulp
- React
---

Vesa Juvonen and Waldek Mastykarz from the OfficeDev 
Patterns & Practises -team have released a new Channel 9
video titled "[Getting started with SharePoint Framework](https://channel9.msdn.com/blogs/OfficeDevPnP/PnP-Web-Cast-Getting-started-with-SharePoint-Framework)".
In this video Vesa and Waldek go through the full development 
cycle of a Client Web Part using the upcoming SharePoint 
Framework.
  
<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-05-17_developmentcycle.jpg" style="max-width: 720px" alt="Client WebPart Development Cycle"/>
  <figcaption style="text-align:center;">The development cycle of a Client WebPart</figcaption>  
</figure>

The development cycle utilizes multiple different web 
development tools, which are actually already quite
popular in the general web development world, but might
be quite new to a traditional SharePoint developer.

### Node.js

[Wikipedia](https://en.wikipedia.org/wiki/Node.js) defines 
Node.js as "an open-source, cross-platform runtime environment 
for developing server-side Web applications."

The SharePoint Framework and the development cycle utilizes
[Node.js](https://nodejs.org/) to run tools like Gulp and for hosting your Client
WebPart during development. But these things will
be prepared for you by Microsoft and/or by the community. 
So you don't need to learn to write Node applications 
to be able to create Client WebParts using the new SharePoint
Framework.

You will however be using the [Node Package Manager (npm)](https://www.npmjs.com/) 
when building your WebParts. Npm can be shortly described as the 
"Nuget for Node.js". You'll use it to install the 
SharePoint Framework itself and when you want to include some
open source JavaScript library in your own component.

Fortunately npm is quite easy to use. Mostly you will just
install packages with the ```npm --save-dev packagename``` command. 

### Yeoman 

If npm is the NuGet for Node.js, then the Visual Studio counterpart
for [Yeoman](http://yeoman.io/) is the project template. When you select the 
project template in in Visual Studio, Visual Studio will initialize
your project with all the initial files and settings. When developing 
Client WebParts, you'll initialize your project with Yeoman instead.
This is done by running something like ```yo sharepoint``` in
the command prompt.

The yeoman templates will be provided for you by Microsoft, so that
single command will be most likely everything you need to know 
about Yeoman.

### Gulp

[Gulp](http://gulpjs.com/) is a build process tool, which runs 
on top of Node. It can be compared to MSBuild, which is used by
Visual Studio when you press ```F5``` in the IDE. 

A gulp build process consists of different tasks which are writen
in JavaScript. Once these tasks are defined, you can execute them
with the shell command ```gulp taskname```.

Yet again, the build tasks for Gulp are part of the SharePoint
Framework and you don't have to write them yourself. The SharePoint
Framework will contain the necessary gulp tasks for building your 
solution, starting the development-time webserver, packaging 
the solution and uploading your bundled WebPart code to a CDN.

### TypeScript

You can write your Client WebParts using JavaScript, but most people
will use [TypeScript](https://www.typescriptlang.org/) instead. 
TypeScript is a typed superset of JavaScript, which is transpiled 
to JavaScript during the build process.

TypeScript has many advantages to plain JavaScript. For example
it makes it easier to write more robust code by supporting classes
and modules. It's also transpiled with type checking, so you'll
get compile-time information about typing mistakes etc...

In my opinion, TypeScript is the thing you should be studying 
right now if you want to be prepared for the new SharePoint
Framework when it launches. The other things so far (Node, Yeoman, Gulp)
are mostly just tools you'll be using to get your WebPart compiled.

### React and Flux

The final thing that is mentioned quite often with SharePoint 
Framework is the [React-library](https://facebook.github.io/react/).
React is a UI library developed by Facebook and Instagram.
It helps you create a set of individual UI components (with
data-binding) and wire them together to create the full UI of your
WebPart.

The new SharePoint Framework will have support for UI components
written with React and lot's of the examples and tutorials 
about the SharePoint Framework will most likely be utilizing
React-library for rendering the user interface. But using React
is not required by the framework. You can use Knockout, Angular
or just plain TypeScript to render the UI instead.

I definitely encourage you to learn about React (and also 
[the Flux architecture model](https://facebook.github.io/flux/))
when preparing for the new SharePoint Framework. But if you have 
to choose between TypeScript and React, then I personally would
start with TypeScript! 
