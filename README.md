# Simple Todo App with MongoDB, Express.js and Node.js
The ToDo app uses the following technologies and javascript libraries:
* MongoDB
* Express.js
* Node.js
* express-handlebars
* method-override
* connect-flash
* express-session
* mongoose
* bcryptjs
* passport
* docker & docker-compose

## What are the features?
You can register with your email address, and you can create ToDo items. You can list ToDos, edit and delete them. 

# How to use
First install the depdencies by running the following from the root directory:

```
npm install --prefix src/
```

To run this application locally you need to have an insatnce of MongoDB running. A docker-compose file has been provided in the root director that will run an insatnce of MongoDB in docker. TO start the MongoDB from the root direction run the following command:

```
docker-compose up -d
```

Then to start the application issue the following command from the root directory:
```
npm run start --prefix src/
```

The application can then be accessed through the browser of your choise on the following:

```
localhost:5000
```

## Testing

Basic testing has been included as part of this application. This includes unit testing (Models Only), Integration Testing & E2E Testing.

### Linting:
Basic Linting is performed across the code base. To run linting, execute the following commands from the root directory:

```
npm run test-lint --prefix src/
```

### Unit Testing
Unit Tetsing is performed on the models for each object stored in MongoDB, they will vdaliate the model and ensure that required data is entered. To execute unit testing execute the following commands from the root directory:

```
npm run test-unit --prefix src/
```

### Integration Testing
Integration testing is included to ensure the applicaiton can talk to the MongoDB Backend and create a user, redirect to the correct page, login as a user and register a new task. 

Note: MongoDB needs to be running locally for testing to work (This can be done by spinning up the mongodb docker container).

To perform integration testing execute the following commands from the root directory:

```
npm run test-integration --prefix src/
```

### E2E Tests
E2E Tests are included to ensure that the website operates as it should from the users perspective. E2E Tests are executed in docker containers. To run E2E Tests execute the following commands:

```
chmod +x scripts/e2e-ci.sh
./scripts/e2e-ci.sh
```


## Circle CI

### configuration
Documentation for Circle CI configuration.

#### machine
`ubuntu-2004:202010-01` is a Linx executor which contains `Ubuntu 20.04, Docker v19.03.13, Docker Compose v1.27.4`. Because it is pre-installed with docker-compose, we can make good use of the project configuration file.

### Jobs

#### checkout
checkout the project code from github.

```yaml
  checkout:
    machine:
      # Ubuntu 20.04, Docker v19.03.13, Docker Compose v1.27.4
      image: ubuntu-2004:202010-01 
    steps: 
      - checkout
```

#### install
checkout the project code from github and then install dependency from npm and docker-compose. `save_cache` saves the `node_modules` folder to save the time of downloading dependencies from the network. It can be restored by `restore_cache`.

```yaml
  install:
    machine:
      image: ubuntu-2004:202010-01 
    steps: 
      - checkout
      - run: npm install --prefix src/
      - run: docker-compose up -d
      - save_cache:
          paths:
            - src/node_modules
          key: node-v1-{{ .Branch }}-{{ checksum "src/package-lock.json" }}
```

#### lint_check
perform a basic lint check.

```yaml
  lint_check:
    machine:
      image: ubuntu-2004:202010-01
    steps:
      - checkout
      - restore_cache:
          keys:
            - node-v1-{{ .Branch }}-{{ checksum "src/package-lock.json" }}
      - run: docker-compose up -d
      - run: npm run test-lint --prefix src/
```

#### unit_test
perform a unit test.

```yaml
  unit_test:
    machine:
      image: ubuntu-2004:202010-01
    steps:
      - checkout
      - restore_cache:
          keys:
            - node-v1-{{ .Branch }}-{{ checksum "src/package-lock.json" }}      
      - run: docker-compose up -d
      - run: npm run test-unit --prefix src/
```

#### integration_test
perform a integration test.

```yaml
  e2e_test:
    machine:
      image: ubuntu-2004:202010-01
    steps:
      - checkout
      - restore_cache:
          keys:
            # when lock file changes, use increasingly general patterns to restore cache
            - node-v1-{{ .Branch }}-{{ checksum "src/package-lock.json" }}      
      - run: docker-compose up -d      
      - run: chmod +x scripts/e2e-ci.sh
      - run: ./scripts/e2e-ci.sh
```

#### e2e_test
perform a e2e test.

```yaml
  checkout:
    machine:
      # Ubuntu 20.04, Docker v19.03.13, Docker Compose v1.27.4
      image: ubuntu-2004:202010-01 
    steps: 
      - checkout
```

#### pack
pack the project. 

```yaml
  checkout:
    machine:
      # Ubuntu 20.04, Docker v19.03.13, Docker Compose v1.27.4
      image: ubuntu-2004:202010-01 
    steps: 
      - checkout
```

#### Workflows

### check_and_pack
check all the test and then pack the project.   
`filter` ensure it only work on the master branch. `requires` means that if the pre-job required by `requires` is unsuccessful, it will directly cause the build to fail.

```yaml
  check_and_pack:
    jobs:
      - checkout
      - install:
          requires:
            - checkout
      - lint_check:
          requires:
            - install
      - unit_test:
          requires:
            - install
      - integration_test:
          requires:
            - install
      - e2e_test:
          requires:
              - lint_check
              - unit_test
              - integration_test
      - pack:
          requires:
              - lint_check
              - unit_test
              - integration_test
              - e2e_test
          filters:
            branches:
              only: master
```







###### This project is licensed under the MIT Open Source License
