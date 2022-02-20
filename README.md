# Heroku buildpack for clojure tools

Minimal buildpack for clojure tools.deps and tools.build projects. Based off the [official heroku clojure build pack](https://github.com/heroku/heroku-buildpack-clojure).

## To use this build pack

Set your heroku app to use this build pack:

```
heroku buildpacks:set https://github.com/andersmurphy/heroku-buildpack-clojure-tools.git -a my-app
```

Make sure your project root contains the following files:

A `Procfile` file:

```
web: java -jar target/my-app.jar -m my-app.app
```

A `bin/build` file:

```
clojure -T:build uber
```

A `deps.edn` file:

```
{:paths   ["src" "resources"]
 :deps    {org.clojure/clojure {:mvn/version "1.11.0-alpha3"}}
 :aliases {:build
           {:deps       {io.github.clojure/tools.build {:git/tag "v0.6.8"
                                                        :git/sha
                                                        "d79ae84"}}
            :ns-default build}}}
```

A `build.clj` file:

```
(ns build
  (:require [clojure.tools.build.api :as b]))

(def lib 'my-app)
(def class-dir "target/classes")
(def basis (b/create-basis {:project "deps.edn"}))
(def uber-file (format "target/%s.jar" (name lib)))

(defn clean [_] (b/delete {:path "target"}))

(defn uber
  [_]
  (clean nil)
  (b/copy-dir {:src-dirs ["src" "resources"] :target-dir class-dir})
  (b/compile-clj {:basis basis :src-dirs ["src"] :class-dir class-dir})
  (b/uber {:class-dir class-dir
           :uber-file uber-file
           :basis     basis
           :main      'my-app.app}))
```
