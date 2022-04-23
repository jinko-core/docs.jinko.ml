# docs.jinko.ml

Website for jinko's technical documentation. The website contains documentation for the standard library, a guide to programming in `jinko`, and various blogposts or deep-dives inside jinko's implementation.

# Build the doc
```console
$ mkdocs deploy
```

Follow the [documentation](https://squidfunk.github.io/mkdocs-material/getting-started/) from mkdocs-material to install `mkdocs`

# Deploying the documentation locally

Run the following command to serve the documentation locally and enable live-reloading if you make changes.

```sh
docker run --rm -it -p 8000:8000 -v $(pwd):/docs squidfunk/mkdocs-material
```

You can now access the documentation at `localhost:8000` in your browser and preview your changes.
