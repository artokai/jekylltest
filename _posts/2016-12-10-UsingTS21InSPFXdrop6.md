---
title: "Using TypeScript 2.1 in SPFX Drop 6"
header:
  teaser: "images/posts/2016-12-10_teaser.jpg"
tags:
- SharePoint
- SPFX
- TypeScript
---

[TypeScript 2.1](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html) was released 
just a couple of days ago. This update finally allows us to use async/await while
targeting the ES5 flavour of Javascript. With "await" we no longer need to use callbacks
or chain our promises together with those ugly then-chains when working with asynchronous code. With async/await
we can treat our asynchronous methods almost as if they were just regular function calls.

Unfortunately SPFX Drop 6 is still using TypeScript 2.0 but we can fix this with the help of 
[npm shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap). By adding a file called `npm-shrinkwrap.json` 
to the root of our project we can override the typescript version used by SPFX. The contents of the file 
should be as follows:

```json
{
  "dependencies": {
    "@microsoft/sp-build-web" : {
      "version": "^0.8.1",
      "dependencies": {
        "typescript": {"version": "^2.1.4" }
      }
    }
  }
}
``` 

When you now run the command `npm install` it will force the SPFX build tools to use the new 
TypeScript version. After npm has finished updating, we can start waiting for promises
in our webparts like this.

```js
public async render() {
  const fact = await this.getFact();
  this.domElement.innerHTML = `
    <div class="${styles.fact}">
      ${fact}
      <cite>The Internet Chuck Norris Database</cite>
    </div>`;
}

protected getFact() : Promise<string>
{
  const url: string = 'https://api.icndb.com/jokes/random';
  return new Promise<string>(function (resolve, reject) {
      var xhr = new XMLHttpRequest();
      xhr.open("GET", url, true );
      xhr.onload = () => { 
        const response: any = JSON.parse(xhr.responseText);
        resolve(response.value.joke); 
      } 
      xhr.onerror = reject;
      xhr.send();
  });
}
```

Note that in order for this to work, we need to add the `async` keyword to our webpart's
render() method and remove the return value type which used to be `void`. The await keyword
can be used to wait for any method that return a promise or is marked with the `async` keyword.

Example code for this blog article can be found in [my GitHub repository](https://github.com/artokai/samples/tree/master/SPFX/TS21_in_SPFX_drop6).