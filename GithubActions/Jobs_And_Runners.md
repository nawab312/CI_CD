<img width="546" height="240" alt="image" src="https://github.com/user-attachments/assets/801633f1-c005-476e-a641-688eea2a6fb8" />

**What is a Runner?**
- A runner is the machine that executes your job. When a workflow triggers, GitHub spins up a fresh virtual machine, runs your job, and destroys it. GitHub provides these out of the box:

  <img width="669" height="158" alt="image" src="https://github.com/user-attachments/assets/d088e5f0-ce57-45c3-9a75-582613f6e487" />
- Always prefer a specific version like ubuntu-22.04 over ubuntu-latest in production — latest can change and break your pipeline unexpectedly. Same lesson as Jenkins agent labels!

**Anatomy of a Job**
```yaml
jobs:
  my-job:                          # job ID (used to reference it)
    name: My Build Job             # display name in GitHub UI
    runs-on: ubuntu-22.04          # runner machine

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run tests
        run: npm test
```

**Multiple Jobs — Parallel by Default**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Linting..."
```
```
build ──┐
test  ──┼──► all run at the same time
lint  ──┘
```

**Sequential Jobs — `needs:`**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    runs-on: ubuntu-latest
    needs: build                   # waits for build to succeed
    steps:
      - run: echo "Testing..."

  deploy:
    runs-on: ubuntu-latest
    needs: [build, test]           # waits for BOTH to succeed
    steps:
      - run: echo "Deploying..."
```
```
build ──► test ──► deploy
```

**Job-level Environment Variables**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
      APP_PORT: 3000

    steps:
      - run: echo "Running on port $APP_PORT"
```

**Timeout & Retry**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30            # kill job if it runs over 30 mins

    steps:
      - name: Flaky test
        uses: nick-fields/retry@v2 # retry action from marketplace
        with:
          max_attempts: 3
          command: npm test
```

**Conditional Jobs — `if:`**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'   # only deploy from main branch
    steps:
      - run: echo "Deploying to production..."
```

**Full Example — Real Pipeline Shape**
```yaml
jobs:
  lint:
    runs-on: ubuntu-22.04
    steps:
      - run: echo "Linting code..."

  build:
    runs-on: ubuntu-22.04
    needs: lint                          # run after lint
    steps:
      - run: echo "Building app..."

  test:
    runs-on: ubuntu-22.04
    needs: lint                          # run after lint (parallel with build)
    steps:
      - run: echo "Running tests..."

  deploy:
    runs-on: ubuntu-22.04
    needs: [build, test]                 # run after BOTH build and test
    if: github.ref == 'refs/heads/main'
    steps:
      - run: echo "Deploying..."
```
```
lint ──► build ──┐
     └──► test ──┴──► deploy (only on main)
```

