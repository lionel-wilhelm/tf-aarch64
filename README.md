# Note
Docs are not complete at the moment. Things to include:

    How build instructions change from one file to another
    Configuration options

## Setup

### Make htpasswd for basic auth

```bash
$ mkdir basic_auth
$ htpasswd -c basic_auth/.htpasswd admin
```

### Make directory for workbench

We use `/tmp/tf_aarch64/` for workbench for this build tool.

```bash
$ mkdir /tmp/tf_aarch64
```


## Running webserver using docker

In the root of the project run:
```shell
docker compose up -d --build
```
Now visit http://localhost you should see the web interface

In the web UI there are two tabs: Tensorflow and TFX. 
Each page has a version dropdown at the bottom of the page for selecting
python version, the library version. 
A version number that looks like `2.7.x` means that it will build the latest changes in the 2.7 branch.

## Development
```shell
cd back/
source venv/bin/active
```
Running python server
```shell
cd back/
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Running ui server
```shell
cd front/
npm i
npm run serve
```

## Adding and building custom versions
Coming soon

## Building wheels manually
First build bazel image that your version of tensorflow/tfdv depends on, you can figure this out by looking 
at the [build matrix](https://www.tensorflow.org/install/source#tested_build_configurations).

Then generate and build that dockerfile for that bazel version using:
```shell
python gen.py bazel -v 3.7 ./Dockerfile_bazel
docker build -t bazel:3.7 -f Dockerfile_bazel ./templates/context
```

Now generate a dockerfile and build the actual image (tensorflow in this case)
```shell
python gen.py tensorflow -v 2.7.3 -py 3.7 ./Dockerfile_tf
docker build -t tensorflow_py37:2.7.3 -f ./Dockerfile_tf ./templates/context/
```

Copy wheels from the resulting image to host machine using:
```shell
docker run -v /host/machine/path:/builds tensorflow_py37:2.7.3 cp -a /wheels/. /builds
```
