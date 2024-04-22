---
title: "Stan Web Demo"
excerpt: "Run Stan models in your browser"
collection: coding-portfolio
---

# Stan Web Demo

After seeing some other interactive web demos built on compiled languages, I realized that
Stan could be a great fit for a similar project.
I [already had a C-like interface to Stan](https://github.com/WardBrian/tinystan),
so existing tools like [Emscripten](https://emscripten.org/) worked with relatively
little fuss.

I then learned a little bit of [React](https://reactjs.org/) and [TypeScript](https://www.typescriptlang.org/)
to wrap the compiled Stan code in a web interface.

Try it out [here](https://brianward.dev/stan-web-demo/).

<iframe width="100%" height="1000px" frameBorder="0" src="https://brianward.dev/stan-web-demo/">
If your browser does not support iframes, click the above link to view the demo.
</iframe>
