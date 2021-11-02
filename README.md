# CI for flask.py and docker build and push 

## workflow file enabel on the root dir under CI-Python_Docker.yaml

in the yaml have the # sing to commet a notes 

# CI STEPS

## 1. CI name and evet Trigger

trigger for run action

case: push & PR to master branch 
<pre>
...
name: CI for python app test
on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master
...
</pre>

## 2. JOBS

### JOb 1. build_test_app

choose VM-ENV work on and the python VER test  
<pre>
...
build_test_app: # Stage name
    runs-on: ubuntu-latest
    env:
      working-directory: ./app #choose directory whit python app
    strategy:
      matrix:
        python-version: ["3.8", "3.9", "3.10"] # test app on deffrent ver
...
</pre>

### steps in build_test_app
#### Checkout
Checkout after clone the repo 
<pre>
...
    — name: Checkout
      uses: actions/checkout@v2
...
</pre>

#### Set up Python VER {3.8, 3.9, 3.10}
make 3 deffrent EVN test for your code on 
<pre>
...
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
...
</pre>

#### Install dependencies
install necessary pack and dependencies 
>pip install {your pack}
<pre>
...
    - name: Install dependencies
      working-directory: ${{env.working-directory}}
      run: |
        python -m pip install --upgrade pip
        pip install flake8 pytest pylint
        if [ -f ../requirements.txt ]; then pip install -r ../requirements.txt; fi # dependencies form root dir "../requirements.txt"
...
</pre>

#### flake8
checks Python files for errors
<pre>
...
   - name: Lint with flake8
      working-directory: ${{env.working-directory}}
      run: |
        # stop the build if there are Python syntax errors or undefined names
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
        flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
...
</pre>

#### Analysing pylint
<pre>
...
    - name: Analysing the code with pylint
      working-directory: ${{env.working-directory}}
      run: |
         pylint -E app.py
...
</pre>

#### Testing app whit Pytest 
testing if all good, test.py file in app/test/app_test.py
<pre>
...
    - name: Testing whit Pytest
      working-directory: ${{env.working-directory}}
      run: |
         pip install --ignore-installed flask ; python3 -m pytest # install last flask ver to ignore errors
...
</pre>

### JOb 2. CODE quality check
https://frontend.code-inspector.com
<pre>
...
  check-quality: 
    runs-on: ubuntu-latest
    needs: build_test_app # dependent by step up

    name: A job to check my code quality
    steps:
    - name: Check code meets quality standards
      id: code-inspector
      uses: codeinspectorio/github-action@master
      with:
        repo_token: ${{ secrets.GITHUB_TOKEN }}
        code_inspector_api_token: ${{ secrets.CODE_INSPECTOR_API_TOKEN }}
        force_ref: 'none'
        min_quality_grade: 'WARNING'
        min_quality_score: '50'
        max_defects_rate: '0.2'
        max_complex_functions_rate: '0.0001'
        max_long_functions_rate: '0.0001'
        project_name: 'docker-ci-github'
        max_timeout_sec: '600'

...
</pre>


### JOb 3. container build and push 
####
choose VM-ENV work
<pre>
...
  docker_build-push:

    runs-on: ubuntu-latest
    needs: check-quality # dependent by step up
...
</pre>

### steps in docker_build-push
#### Checkout Set up QEMU and Docker Buildx
set up all the settings for docker builder  
<pre>
...
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

...
</pre>

#### Login to DockerHub
use github secret and add your DOCKERHUB_USERNAME and DOCKERHUB_TOKEN
<pre>
...
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
...
</pre>

#### Login to DockerHub
Build and Push your new container to your repo 
<pre>
...
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/python-flask-docker:1.0.0 #tag amitshochat66/python-flask-docker:1.0.0
...
</pre>
