# Post-build Clean Buildpack

A simple buildpack to run after all other buildpacks have completed,
which removes a set of files defined in `.slug-post-clean`, so that they
are not included in the finished slug.

## Rationale

While this may seem to duplicate functionality provided by Heroku's
`.slugignore`, there is a key difference: `.slugignore`'d files are
removed after the repo is cloned, but before any buildpack is run. They
can therefore not be involved in the build process itself.

However, it is not uncommon for there to exist files in the repo that
are necessary for the build, but are not required at runtime. There may
also be installable build dependencies that are not runtime
dependencies.

In our case, a complex front-end build involves significant CSS, JS and
image assets, along with a large installation of node modules, all of
which are used only for building the production assets, but then remain
part of the slug.

## Usage

Add the buildpack to your app using the Heroku CLI: `heroku buildpacks:add https://github.com/vintasoftware/heroku-buildpack-post-build-clean`. The post-build-clean buildpack **must** be last in the
buildpack order:

```
$ heroku buildpacks

=== test-app-12345 Buildpack URLs
1. heroku/nodejs
2. heroku/python
3. https://github.com/vintasoftware/heroku-buildpack-post-build-clean
```

The `.slug-post-clean` file supports single-file and single-directory patterns, **as well as glob patterns**, e.g.:

```
some_huge_file.psd
some/nested/directory
why_does_this_app_even_contain_a.tiff
your_frontend/build/*.map
```

Glob patterns are expanded using Bash syntax - not with a third-party library such as [node-glob](https://github.com/isaacs/node-glob). You can see what your glob matches from  `bash` with this command:

```bash
ls -d1 your/glob/pattern/*
```

## Testing

The `bin/compile` script does all the heavy lifting for this package. It takes a single argument, $BUILD_DIR, the directory of your app. If you want to test it out locally and make sure your app still runs after the post-build cleanup, you can copy `bin/compile` into your app directory and run:

```bash
chmod +x compile; ./compile .
```
