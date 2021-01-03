# Building hello-2bytes theme
The theme for this site is a fork of [hello-friend](https://github.com/panr/hugo-theme-hello-friend).

My preference is to build things in Docker, this keeps my main machine clean of things I don't want there (nodejs et al)

The following is most likely not the correct way to do this, but given the original instructions of `hello-friend`
are clearly directed at someone who knows npm/nodejs/webpack et al, I simply figured out how to pack the theme for use.

I'm including these commands here to remind myself, more than anything, how I did this, for next time:

First clone the submodule if you haven't already; start by changing to the `themes` directory:
```console
cd themes
git submodule update --init
```

From the current directory (`themes`) run the following:
```console
docker run -it -v $(pwd)/hello-2bytes:/theme node:11-alpine /bin/sh 
```
this will drop you at an alpine linux shell, now run the setup from the `hello-friend` README:
```shell
cd /theme
npm install
npm i yarn
yarn
```
This installs the dependencies, now pack the theme:
```shell
npx webpack --mode=production
```
The theme is now packed.

Keeping this shell open and running the pack each time an edit is made, will cause the hugo server to reload the changes, these will be reflected live in the web page.

> Some items cache in the browser, and a CTRL+F5 is necessary to reflect the changes.

## File permissions
> The Docker command is a lazy one, and should, really be setting the host user (which includes mounting some files; /etc/passwd, /etc/group, /etc/shadow)
so that ownership of the files generated are correctly attributed. Because it's lazy, simply chown the files to yourself on the host filesystem once done.
```
sudo chown -R <user>: hello-2bytes
```
